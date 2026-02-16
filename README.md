# BLEN Engineering Take-Home Challenge

Welcome to the BLEN Engineering Take-Home Challenge repository. This challenge is designed to assess candidates' skills in cloud infrastructure, networking, security, containerization, and DevOps practices.

## Challenge

**DevSecOps Engineer: Secure Three-Tier Architecture on AWS**
- [Challenge Instructions](aws-three-tier-architecture.md)
- Focus: Networking design, infrastructure as code, containerization, database security, and CI/CD pipeline setup

### What Candidates Will Build

A production-style three-tier architecture on AWS:

1. **Networking Foundation** - VPC with public, private, and isolated subnets, IGW, NAT Gateways, and proper route tables
2. **Data Tier** - RDS PostgreSQL in isolated subnets with no public access
3. **Application Tier** - ECS Fargate running a containerized Next.js app in private subnets
4. **Presentation Tier** - Application Load Balancer in public subnets
5. **CI/CD Pipeline** - GitHub Actions for automated build and deployment
6. **Security** - IAM least privilege, security groups, Secrets Manager, network isolation

## Provided Resources

The `app/` directory contains a pre-built Next.js application with:
- A Dockerfile for containerization
- Database connectivity check (displays connection status on the home page)
- No application code modifications required - candidates focus purely on infrastructure

## How to Use This Repository

**Candidates:**
1. Read the [challenge instructions](aws-three-tier-architecture.md) thoroughly
2. Fork this repository
3. Implement your solution
4. Submit the link to your forked repository

**BLEN Team Members:**
- Use your internal evaluation rubric when assessing submissions
- Place significant emphasis on networking and security when evaluating solutions

## General Guidelines

- All infrastructure must be provisioned with Terraform
- Code quality, documentation, and the ability to explain your solution are important
- Prioritize security and proper network segmentation
- Time management is crucial; focus on core requirements before bonus features
