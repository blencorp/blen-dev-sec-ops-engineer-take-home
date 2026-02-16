# DevSecOps Engineer Take-Home: Secure Three-Tier Architecture on AWS

## Overview

This assessment evaluates your ability to design and deploy a **secure, scalable three-tier architecture** on AWS using Terraform. You will build the infrastructure from the ground up, starting with the networking foundation and progressing through the data, application, and presentation tiers. A pre-built Next.js application is provided so you can focus entirely on infrastructure and DevOps practices.

### Architecture Diagram

```
                        ┌─────────────────────────────────────────────────────┐
                        │                       VPC                           │
                        │                                                     │
                        │  ┌─────────────────┐     ┌─────────────────┐       │
        Internet ──────►│  │  Public Subnet   │     │  Public Subnet   │      │
                        │  │     (AZ-1)       │     │     (AZ-2)       │      │
                        │  │  ┌───────────┐   │     │  ┌───────────┐   │      │
                        │  │  │    ALB    │   │     │  │    ALB    │   │      │
                        │  │  └───────────┘   │     │  └───────────┘   │      │
                        │  │  ┌───────────┐   │     │  ┌───────────┐   │      │
                        │  │  │  NAT GW   │   │     │  │  NAT GW   │   │      │
                        │  │  └─────┬─────┘   │     │  └─────┬─────┘   │      │
                        │  └────────┼─────────┘     └────────┼─────────┘      │
                        │           │                        │                │
                        │  ┌────────▼─────────┐     ┌────────▼─────────┐      │
                        │  │  ┌────────────┐   │     │  ┌────────────┐   │     │
                        │  │  │ ECS Fargate│   │     │  │ ECS Fargate│   │     │
                        │  │  └─────┬──────┘   │     │  └─────┬──────┘   │     │
                        │  └────────┼──────────┘     └────────┼──────────┘     │
                        │           │                         │               │
                        │  ┌────────▼─────────┐     ┌────────▼─────────┐      │
                        │  │ Isolated Subnet   │     │ Isolated Subnet   │     │
                        │  │     (AZ-1)        │     │     (AZ-2)        │     │
                        │  │  ┌────────────┐   │     │  ┌────────────┐   │     │
                        │  │  │  RDS (Pri) │◄──┼─────┼──│ RDS (Stdby)│   │     │
                        │  │  └────────────┘   │     │  └────────────┘   │     │
                        │  └───────────────────┘     └───────────────────┘     │
                        └─────────────────────────────────────────────────────┘
```

## What You Will Build

| Tier              | Components                            | Subnet Type |
|-------------------|---------------------------------------|-------------|
| **Presentation**  | Application Load Balancer             | Public      |
| **Application**   | ECS Fargate (Next.js containers)      | Private     |
| **Data**          | RDS PostgreSQL                        | Isolated    |

---

## Step 1: Networking Foundation

> This is the most critical step. A well-architected network is the backbone of a secure cloud deployment.

Using Terraform, create the following:

### VPC
- A VPC with a `/16` CIDR block (e.g., `10.0.0.0/16`)

### Subnets (across 2 Availability Zones)
| Subnet Type       | Purpose                              | Internet Access         |
|--------------------|--------------------------------------|-------------------------|
| **Public** (x2)    | ALB, NAT Gateways                   | Direct via IGW          |
| **Private** (x2)   | ECS Fargate tasks (application)      | Outbound via NAT GW    |
| **Isolated** (x2)  | RDS instances (database)             | None                    |

### Gateways
- **Internet Gateway (IGW)**: Attached to the VPC, enabling internet access for public subnets
- **NAT Gateway(s)**: Deployed in each public subnet, enabling outbound internet access for private subnets (e.g., pulling container images)

### Route Tables
- **Public Route Table**: Routes `0.0.0.0/0` to the Internet Gateway
- **Private Route Table(s)**: Routes `0.0.0.0/0` to the NAT Gateway
- **Isolated Route Table**: No route to `0.0.0.0/0` (completely isolated from the internet)

### Validation Criteria
- Public subnets can reach the internet
- Private subnets can reach the internet only via NAT Gateway (outbound only)
- Isolated subnets have **no** internet access
- All route table associations are correct

---

## Step 2: Data Tier - RDS PostgreSQL

Using Terraform, provision the database layer:

### RDS Instance
- **Engine**: PostgreSQL
- **Deployment**: Place in the **isolated subnets** using a DB subnet group
- **High Availability**: Enable **Multi-AZ deployment** so the database has a primary and standby in different Availability Zones
- **Public Access**: Disabled (`publicly_accessible = false`)
- **Storage**: Use a reasonable instance size (e.g., `db.t3.micro` for this exercise)

### Security
- Create a **security group** for RDS that allows inbound PostgreSQL traffic (port `5432`) **only** from the application tier's security group
- No other inbound traffic should be allowed
- Store database credentials in **AWS Secrets Manager**

### Bonus: Bastion Host
- Deploy an EC2 bastion host in a **public subnet** for secure database administration
- The bastion should be the only additional resource allowed to connect to RDS
- Configure SSH key-based access to the bastion

---

## Step 3: Application Tier - ECS Fargate

Deploy the provided Next.js application using ECS Fargate.

### Container Registry
- Create an **ECR repository** for the application image
- Build the provided Docker image and push it to ECR

### ECS Cluster & Service
- Create an **ECS cluster** with Fargate launch type
- Create a **task definition** that:
  - Uses the image from ECR
  - Retrieves database credentials from **Secrets Manager** and injects them as environment variables (`DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_PORT`)
  - Runs in the **private subnets**
- Create an **ECS service** with desired count of at least 1

### Security
- Create a **security group** for ECS tasks:
  - Allow inbound traffic on port `3000` from the ALB security group only
  - Allow outbound traffic to the RDS security group on port `5432`
  - Allow outbound HTTPS (port `443`) to required AWS service endpoints (e.g., ECR, ECS, Secrets Manager, CloudWatch, STS), either via internet egress or VPC interface endpoints, for pulling images, accessing Secrets Manager, and ECS control-plane operations
  - Ensure DNS resolution is possible (allow egress on port `53` to the VPC DNS resolver or configure appropriate VPC endpoints for DNS resolution)

---

## Step 4: Presentation Tier - Application Load Balancer

### ALB Configuration
- Deploy an **Application Load Balancer** in the **public subnets**
- Create a **target group** pointing to the ECS service on port `3000`
- Configure a **listener** on port `80` forwarding to the target group
- Set up **health checks** against the application

### Security
- Create a **security group** for the ALB:
  - Allow inbound HTTP (port `80`) from `0.0.0.0/0`
  - Allow outbound traffic to the ECS security group on port `3000`

### Validation
- The application should be accessible via the ALB's DNS name
- The application's database connectivity indicator should show a successful connection

---

## Step 5: CI/CD Pipeline

Set up a **GitHub Actions** workflow:

### Pipeline Stages

```
Push to main ──► Build Docker Image ──► Push to ECR ──► Deploy to ECS
```

1. **Trigger**: On push to the `main` branch
2. **Build & Push**:
   - Authenticate with ECR
   - Build the Docker image
   - Tag and push the image to ECR
3. **Deploy**:
   - Update the ECS task definition with the new image
   - Trigger an ECS service update to deploy the new version

---

## Step 6: Security Review

Ensure the following security practices are implemented throughout your infrastructure:

- [ ] **IAM**: All roles follow the principle of least privilege
- [ ] **Network Isolation**: RDS is in isolated subnets with no internet access
- [ ] **Security Groups**: Traffic flows only between intended tiers (ALB -> ECS -> RDS)
- [ ] **Secrets Management**: Database credentials are stored in Secrets Manager, never hardcoded
- [ ] **Encryption**: RDS storage encryption is enabled
- [ ] **No Public DB Access**: RDS `publicly_accessible` is set to `false`

---

## Provided Application

The `app/` directory contains a pre-built Next.js application with:
- A **Dockerfile** for containerization
- A home page that displays database connectivity status
- An API endpoint (`/api/db-check`) that tests the PostgreSQL connection
- Sample `.env.local` showing the required environment variables

### Required Environment Variables
| Variable      | Description            |
|---------------|------------------------|
| `DB_HOST`     | RDS endpoint           |
| `DB_USER`     | Database username       |
| `DB_PASSWORD` | Database password       |
| `DB_NAME`     | Database name           |
| `DB_PORT`     | PostgreSQL port (5432)  |

> You do **not** need to modify the application code. Focus on the infrastructure.

---

## Important Notes

- Use **Terraform** for all infrastructure provisioning
- Use **Terraform modules** to keep your code modular and maintainable
- Implement **Terraform state management** using an S3 backend (include setup instructions)
- Provide clear comments in your code and comprehensive documentation in your README
- In your documentation, explain:
  - Your approach to networking and why you segmented subnets the way you did
  - How you securely manage database credentials
  - How traffic flows through each tier
  - Any additional security measures you implemented

---

## Bonus Points (Optional)

If time allows, consider implementing:
- **Bastion Host** for secure database access (see Step 2)
- **AWS WAF** on the Application Load Balancer
- **CloudWatch** alarms and dashboards for monitoring
- **Container image scanning** in the CI/CD pipeline
- **HTTPS** with ACM certificate on the ALB

---

## Out of Scope

To help you focus on the core requirements:
- Complex application logic (the app is provided and ready to use)
- Custom domain and DNS configuration
- Comprehensive monitoring and alerting systems
- Advanced database features (read replicas, Performance Insights)
- Service mesh or advanced networking features

---

## Submission Instructions

1. Fork this repository and implement your solution
2. Ensure all infrastructure-as-code and CI/CD configurations are committed
3. Update the README.md with:
   - Clear instructions on how to deploy your solution
   - Architecture decisions and trade-offs
   - Security considerations
4. Submit the link to your GitHub repository

---

