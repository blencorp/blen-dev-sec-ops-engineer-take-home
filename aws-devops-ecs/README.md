# DevSecOps Engineer Assessment: Containerized Next.js Application with RDS

## Overview
This assessment evaluates your DevSecOps skills, focusing on infrastructure as code, containerization, database integration, security implementation, and CI/CD pipeline setup. You'll deploy a provided Next.js application in a secure, scalable containerized architecture using Terraform and AWS services, including RDS PostgreSQL integration.

## Challenge Requirements
1. Use Terraform for all infrastructure provisioning
2. Deploy the provided Next.js application on AWS ECS (Elastic Container Service) with Fargate launch type
3. Use AWS ECR (Elastic Container Registry) for storing the application's Docker image
4. Set up an RDS instance running PostgreSQL and integrate it with the application
5. Implement AWS Secrets Manager for secure credential management
6. Set up a CI/CD pipeline using GitHub Actions (This does NOT need to be in Terraform)

## Provided Resources
- A GitHub repository containing a simple Next.js application with:
  * A pre-configured Dockerfile
  * Sample environment variable configuration for database connection
  * Basic database query functionality to demonstrate connectivity

## Functionality to Implement

### 1. Networking (using Terraform)
- Create a VPC with two public and two private subnets across two availability zones
- Set up an Internet Gateway and NAT Gateways for internet connectivity
- Configure appropriate route tables

### 2. Compute and Containers (using Terraform)
- Create an ECS cluster using Fargate launch type
- Set up an ECR repository for the application image

### 3. Database Setup (using Terraform)
- Provision an RDS instance running PostgreSQL in the private subnets
- Configure appropriate security groups to allow traffic only from the application's security group
- Use AWS Secrets Manager to store the database credentials

### 4. Application Deployment and Database Integration
- Push the provided application's Docker image to ECR
- Create an ECS task definition and service to run the application
- Configure the ECS task to use environment variables for database connection, retrieving credentials from AWS Secrets Manager
- Ensure the application can successfully connect to the RDS instance
- Make the application accessible through an Application Load Balancer on port 80

### 5. Security Implementation
- Implement IAM roles following the principle of least privilege
- Secure all network traffic using appropriate security groups
- Ensure secure transmission of database credentials from Secrets Manager to the application

### 6. CI/CD Pipeline
Set up a GitHub Actions workflow with the following stages:
a. Trigger: On push to the main branch
b. Build and Push: 
   - Build the Docker image
   - Push the image to ECR
c. Deploy: 
   - Update the ECS task definition
   - Trigger an ECS service update

## Project Structure
Organize your repository as follows:
- `/terraform`: Terraform configuration files
- `/app`: The provided Next.js application code (including Dockerfile)
- `/.github/workflows`: GitHub Actions workflow files
- `README.md`: Project documentation

## Important Notes
- Prioritize security in your implementation
- Provide clear comments in your code
- In your documentation, explain:
  * Your approach to securely managing database credentials
  * How you've ensured the principle of least privilege in your IAM configurations
  * Any additional security measures you've implemented
  * How you would approach monitoring and logging in a production environment

## Submission Instructions
1. Fork the provided GitHub repository and implement your solution
2. Ensure all your infrastructure-as-code and CI/CD configurations are committed
3. Update the README.md with:
   - Clear instructions on how to deploy your solution
   - Any assumptions or decisions you made during implementation
   - Explanations requested in the "Important Notes" section
4. Submit the link to your GitHub repository

## Evaluation Criteria
Your submission will be evaluated based on:
1. Correct implementation of the required infrastructure and services
2. Effective use of Terraform for infrastructure as code
3. Proper implementation of containerization and ECS deployment
4. Successful integration of the application with RDS
5. Security considerations and implementation
6. Completeness and efficiency of the CI/CD pipeline
7. Code quality, organization, and documentation
8. Ability to explain your design decisions and approach

Good luck! We look forward to seeing your solution.