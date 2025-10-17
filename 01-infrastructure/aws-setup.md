# CLOUD INFRASTRUCTURE RUNBOOK FOR DIGITAL COLLECTIONS

**Document ID:** aws-setup.md  
**Version:** 1.0   
**Maintainer:** Sai Kiran Boppana & LaTaevia Berry   
**Last Updated:** Oct 17 2025  
**Review Cycle:** Quarterly 

## Document Overview

This runbook outlines the end-to-end deployment process for a cloud-hosted Omeka S infrastructure tailored for higher education digital collections. It combines platform-agnostic principles with detailed AWS implementation guidance, allowing institutions to adapt to the same architecture on AWS, Azure, Google Cloud, or other platforms according to their technical capacity and budget.

The design follows a four-tier environment model—Development, Testing, Staging, and Production—to ensure strict isolation between internal development systems and publicly accessible services. This structure promotes secure, reliable workflows while providing high-availability access for users exploring digital archives online.

## Scope and Applicability

**Applies To:**
- All Omeka S deployments in HBCU/ higher education environments
- AWS regions: us-east-1, us-west-2 (adjust per institution)
- Environment tiers: Development, Testing, Staging, Production

## Target Audience

This documentation is written for:
- **Primary:** Full-stack engineers and DevOps professionals responsible for initial deployment
- **Secondary:** System administrators managing ongoing maintenance and scaling
- **Tertiary:** Technical project managers overseeing infrastructure decisions

Readers should have basic familiarity with cloud computing concepts, networking fundamentals, and Linux system administration.

## Service Equivalency Matrix

For multi-cloud deployments, refer to this service comparison:

| Service Category | AWS | Azure | Google Cloud |
|-----------------|-----|-------|--------------|
| Compute | EC2 | Virtual Machines | Compute Engine |
| Database | RDS MySQL | Azure Database for MySQL | Cloud SQL |
| Storage | S3 | Blob Storage | Cloud Storage |
| IAM | IAM | Azure AD + RBAC | Cloud IAM |
| DNS | Route 53 | Azure DNS | Cloud DNS |
| SSL/TLS | ACM | App Service Certificates | Certificate Manager |

## Prerequisites

Before starting the deployment, ensure all technical, organizational, and environmental requirements are met. This section outlines the accounts, permissions, skills, tools, and information you'll need to complete the setup successfully.

### 2.1 AWS Account and Permissions

Before beginning this deployment, verify that you have an active AWS account with the necessary IAM permissions. Your IAM user or role should have full administrative access to the following services:

**Core Services Required:**
- **Compute & Networking:** EC2, VPC
- **Storage & Databases:** S3, RDS
- **Security & Monitoring:** IAM, CloudWatch, Systems Manager
- **DNS & CDN:** Route 53, CloudFront (optional)

**Specific Permission Requirements:**

| Action | Service | Permission Level |
|--------|---------|-----------------|
| Create/configure EC2 instances | EC2 | Full Access |
| Manage security groups and key pairs | EC2 | Full Access |
| Modify VPC components | VPC | Full Access |
| Deploy RDS instances | RDS | Full Access |
| Create S3 buckets and policies | S3 | Full Access |
| Configure IAM roles and policies | IAM | Full Access |
| Manage CloudWatch logs | CloudWatch | Read/Write |
| Access Systems Manager sessions | SSM | Full Access |

> **Important:** If operating within a shared institutional AWS account, confirm with your AWS administrator that you have these permissions, or request temporary elevated privileges for the initial deployment.

**Required Information to Document:**
- [ ] AWS Account ID: 
- [ ] Primary Region: 
- [ ] VPC Name/ID: 
- [ ] Availability Zones: 

### 2.2 Technical Knowledge Requirements

This runbook assumes familiarity with key concepts and tools across infrastructure and web application management.

**Required Knowledge:**
- **Networking fundamentals:** subnets, CIDR notation, public vs. private IPs, security groups
- **Linux system administration:** SSH access, file permissions, package management, log review
- **Web server management:** Apache configuration, PHP runtime settings, service restarts
- **Database connectivity:** basic SQL commands, MySQL/MariaDB structure, RDS endpoint configuration
- **DNS and domain management:** for production deployments

**Beneficial (Optional) Skills:**
- Infrastructure as Code (IaC) concepts (Terraform, AWS CDK, or CloudFormation)
- JSON/YAML syntax for AWS configurations and template files
- Git version control for configuration management

### 2.3 Required Tools and Software

Ensure the following tools are installed and properly configured on your local machine:

| Tool | Version | Purpose | Installation Notes |
|------|---------|---------|-------------------|
| AWS CLI | v2.x or later | Command-line AWS access | Configure using `aws configure` |
| SSH Client | Latest | Secure shell access | Built-in (macOS/Linux), PuTTY (Windows) |
| Text Editor | Any | Configuration editing | VS Code, Sublime Text, or Atom |
| Web Browser | Modern | AWS Console access | Chrome, Firefox, or Edge |
| Git | 2.x+ | Version control | Optional but recommended |

### 2.4 Environmental and Organizational Requirements

Coordinate with your institution's IT and finance teams to gather:

**Network Configuration:**
- [ ] Approved IP ranges for SSH access
- [ ] Approved IP ranges for RDS access 
- [ ] Firewall exceptions needed

**Organizational Standards:**
- [ ] Cost center codes
- [ ] Project name for tagging 
- [ ] Department identifier 
- [ ] Resource naming convention 

**Compliance Requirements:**
- [ ] HIPAA compliance needed: Yes/No
- [ ] FERPA compliance needed: Yes/No
- [ ] Data residency requirements 
- [ ] Encryption standards 

**Stakeholder Matrix:**

| Environment | Primary Users | Access Level | Authentication Method |
|------------|---------------|--------------|----------------------|
| Development | Developers | Full | SSH Key |
| Testing | QA Team | Limited | SSH Key |
| Staging | Stakeholders | Read-only | HTTPS |
| Production | Public | Read-only | HTTPS |

### 2.5 Pre-Deployment Checklist

Complete these tasks before launching any AWS resources:

- [ ] Create local workspace folder structure:
```
  omeka-deployment/
  ├── configs/
  ├── keys/
  ├── scripts/
  └── logs/
```
- [ ] Verify AWS service quotas:
  - [ ] EC2 instances (minimum 4 required)
  - [ ] Elastic IPs (minimum 1 for production)
  - [ ] RDS instances (minimum 1 required)
- [ ] Obtain Omeka S files:
  - [ ] Download latest stable version OR
  - [ ] Access organizational AMI ID 
- [ ] Set up credential management
  - [ ] Configure AWS Secrets Manager OR
  - [ ] Set up secure password manager
- [ ] Review policies:
  - [ ] Backup policy documented
  - [ ] Disaster recovery RTO/RPO defined
  - [ ] Change management process confirmed
- [ ] Production setup:
  - [ ] Domain/subdomain confirmed 
  - [ ] SSL certificate method chosen
  - [ ] Expected traffic documented 
- [ ] Integration requirements:
  - [ ] LDAP endpoint
  - [ ] SSO configuration 
  - [ ] Storage integration 

## Outcome

Once these prerequisites are met, you will have:
- A verified AWS account with required permissions
- The tools and credentials ready for secure deployment
- Institutional compliance and budget approvals documented
- A clear, consistent workspace for the infrastructure build

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Oct 17 2025 | Sai Kiran Boppana | Initial runbook creation |

---

**Next Section:** [1. Compute Instance (EC2) Provisioning →](./01-compute-instance.md)
