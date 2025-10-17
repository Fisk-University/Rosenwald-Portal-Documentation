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

## 1. Elastic Cloud Compute (EC2) - Virtual Server Provisioning

### Conceptual Overview

A compute instance is a virtual server that runs your application code. In this architecture, we deploy four separate compute instances, one for each environment (Development, Testing, Staging, Production). These instances run on a LAMP stack (Linux, Apache, MySQL client, PHP) with the application pre-installed.

The key principle is **isolation**: each environment has its own dedicated compute resources to prevent interference between development work and production operations. Private environments (Dev/Test) are isolated in private network segments accessible only through a secure jump server, while public environments (Stage/Prod) are accessible from the internet with appropriate security controls.

### AWS Implementation: EC2 Instance Setup

#### Step 1: Open AWS Console and Search for EC2

Navigate to the AWS Console and search for "EC2" in the service search bar. Select **EC2 – Virtual Servers in the Cloud** from the list.

![AWS Console Search Bar Showing EC2 Service](./images/ec2-01-console-search.png)

#### Step 2: Launch Instance Configuration

Navigate to the EC2 Dashboard in AWS Console. Click **"Launch Instance"** to begin the provisioning process.

![EC2 Dashboard showing the Launch Instance button](./images/ec2-02-launch-instance.png)

#### Step 3: Name and Tags Configuration

For each environment, use a consistent naming convention:

| Environment | Instance Name |
|------------|---------------|
| Development | `omeka-dev-instance` |
| Testing | `omeka-test-instance` |
| Staging | `omeka-stage-instance` |
| Production | `omeka-prod-instance` |

![EC2 Launch Instance showing the Name and Tags Section](./images/ec2-03-name-tags.png)

To add tags, click on the **"Add additional tags"** option next to the name field.

**Required Tags for Resource Management:**

| Key | Value |
|-----|-------|
| Environment | Dev (or Test/Stage/Prod) |
| Project | Omeka-Digital-Collections |
| Department | [Your Department Name] |
| CostCenter | [Your Cost Center Code] |

![EC2 Launch Instance showing the Name and Tags filled out](./images/ec2-04-name-tags-filled.png)

#### Step 4: AMI (Amazon Machine Image) Selection

You have two options for the base image:

**Option A - Custom AMI:**
If you have created a custom AMI with the application pre-installed:
- Go to **"My AMIs"** tab
- Select your custom image (e.g., "Application Installation - No MySQL")
- This AMI should include: Ubuntu 22.04 LTS, Apache 2.4, PHP 8.1, application files

**Option B - Base Ubuntu AMI (Manual Installation):**
- Select **"Ubuntu Server 24.04 LTS (HVM), SSD Volume Type"**
- Architecture: **64-bit (x86)**
- This option requires manual installation of all components

![EC2 Launch Instance showing the Custom AMI with Ubuntu as OS](./images/ec2-05-ami-selection.png)

#### Step 5: Instance Type Selection

Select instance types based on environment requirements and expected load:

| Environment | Instance Type | Specifications | Use Case |
|------------|--------------|---------------|----------|
| **Development** | t3.medium | 2 vCPUs, 4 GiB Memory | Light development work, single developer access |
| **Testing** | t3.medium | 2 vCPUs, 4 GiB Memory | Automated testing, QA processes |
| **Staging** | t3.large | 2 vCPUs, 8 GiB Memory | Stakeholder reviews, pre-production validation |
| **Production** | m5.large | 2 vCPUs, 8 GiB Memory | Live public access, concurrent users |

> **Note:** m5 instances provide better network performance and dedicated hardware compared to t3

![EC2 Launch Instance showing Various Instance Types with Selection as t3.medium](./images/ec2-06-instance-types.png)

#### Step 6: Key Pair Configuration

Select an existing key pair or create a new one for SSH access:

**Creating a New Key Pair:**
1. Click **"Create new key pair"**
2. Key pair name: `omeka-environments-keypair`
3. Key pair type: **RSA**
4. Private key file format: **.pem** (for Mac/Linux) or **.ppk** (for Windows using PuTTY)
5. Click **"Create key pair"** and save the downloaded file securely

> **CRITICAL: Key Security**
> - **Linux/Mac:** `chmod 400 omeka-environments-keypair.pem`
> - **Windows:** Use file properties to restrict access to your user account only
> - Backup this key in a secure password manager or key management system
> - Never commit this key to version control or share via email

![EC2 Launch Instance showing Create New Key-Pair](./images/ec2-07-keypair-create.png)

#### Step 7: Network Settings

Network configuration differs between private (Dev/Test) and public (Stage/Prod) environments:

**For Development and Testing (Private Environments):**

| Setting | Configuration |
|---------|--------------|
| **VPC** | Select your existing VPC (e.g., `Omeka-Project-VPC`) or default VPC |
| **Subnet** | Select private subnet |
| - Development | 10.0.3.0/24 (or your designated private subnet) |
| - Testing | 10.0.4.0/24 (or your designated private subnet) |
| **Auto-assign public IP** | **Disable** (these instances should not have direct internet access) |

![Network settings showing VPC and private subnet selection with auto-assign public IP disabled](./images/ec2-08-network-private.png)

**For Staging and Production (Public Environments):**

| Setting | Configuration |
|---------|--------------|
| **VPC** | Select your existing VPC (e.g., `Omeka-Project-VPC`) |
| **Subnet** | Select public subnet |
| - Staging | 10.0.1.0/24 (or your designated public subnet) |
| - Production | 10.0.2.0/24 (or your designated public subnet) |
| **Auto-assign public IP** | **Enable** (these instances need internet accessibility) |

![Network settings showing VPC and public subnet selection with auto-assign public IP enabled](./images/ec2-09-network-public.png)

#### Step 8: Security Group Configuration

Security groups act as virtual firewalls controlling inbound and outbound traffic. Create separate security groups for each environment type:

**Bastion Host Security Group (Already Exists):**
- Name: `bastion-sg`
- Inbound Rules:
  - SSH (Port 22) from your institution's IP range
- Outbound Rules:
  - All traffic allowed

**Development/Testing Security Group:**
- Create new security group: `omeka-dev-test-sg`

| Rule Type | Protocol | Port | Source | Purpose |
|-----------|----------|------|--------|---------|
| **Inbound Rules** |
| SSH | TCP | 22 | Bastion security group ID | Administrative access |
| HTTP | TCP | 80 | Bastion security group ID | Testing via tunnel |
| Custom TCP | TCP | 3306 | Same security group | RDS access |
| **Outbound Rules** |
| All traffic | All | All | 0.0.0.0/0 | Package updates and API calls |

**Staging/Production Security Group:**
- Create new security group: `omeka-stage-prod-sg`

| Rule Type | Protocol | Port | Source | Purpose |
|-----------|----------|------|--------|---------|
| **Inbound Rules** |
| SSH | TCP | 22 | Bastion security group ID | Administrative access |
| HTTP | TCP | 80 | 0.0.0.0/0 | Public access |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Secure public access |
| Custom TCP | TCP | 3306 | Same security group | RDS access |
| **Outbound Rules** |
| All traffic | All | All | 0.0.0.0/0 | All outbound connections |

#### Step 9: Storage Configuration

Configure the root EBS (Elastic Block Store) volume for the operating system and application files:

**Root Volume Configuration:**

| Setting | Dev/Test | Stage/Prod |
|---------|----------|------------|
| **Volume Type** | GP3 (General Purpose SSD) | GP3 (General Purpose SSD) |
| **Size** | 20 GiB | 50 GiB |
| **IOPS** | 3000 (default) | 3000 (default) |
| **Throughput** | 125 MiB/s (default) | 125 MiB/s (default) |
| **Delete on termination** | Yes | No (Production only) |
| **Encryption** | AWS managed keys | AWS managed keys |

![Storage configuration showing GP3 volume with size settings](./images/ec2-10-storage-config.png)

**Additional Storage Considerations:**
- Media files will be stored in S3, not on the instance
- Database will use RDS, not local storage
- Consider creating an AMI snapshot schedule for backup purposes

#### Step 10: Advanced Details - IAM Instance Profile

Attach an IAM role to allow the EC2 instance to access other AWS services:

**IAM Instance Profile Selection:**
- Select existing role or create new: `omeka-ec2-role`

This role should have the following policies attached:
- S3 access to environment-specific buckets
- CloudWatch logs write access
- Systems Manager (SSM) access for maintenance

> **Note:** For detailed IAM configuration, see Section 4: IAM (Identity and Access Management) Configuration

#### Step 11: Launch Instance and Verify

Review all settings in the Summary panel and click **"Launch Instance"**

![Summary page showing all configured settings before launch](./images/ec2-11-summary.png)

**Post-Launch Verification:**
1. Go to EC2 Dashboard → Instances
2. Wait for Instance State to show **"Running"**
3. Wait for Status Checks to show **"2/2 checks passed"**

![Instance Summary Showing Status as “Running”](./images/ec2-12-instance-status.png)

#### Step 12: Elastic IP Configuration (Production Environment Only)

Production requires a static IP address that persists even if the instance is stopped/started.

**Allocating an Elastic IP:**

1. Navigate to EC2 Dashboard → Network & Security → **Elastic IPs**
2. Click **"Allocate Elastic IP address"**
3. Configure settings:
   - Network Border Group: Select your region
   - Public IPv4 address pool: Amazon's pool of IPv4 addresses
4. Add Tags:
   - Key: `Name` | Value: `omeka-prod-eip`
   - Key: `Environment` | Value: `Production`
5. Click **"Allocate"**

**Associating Elastic IP with Production Instance:**

1. Select the newly allocated Elastic IP
2. Click **"Actions"** → **"Associate Elastic IP address"**
3. Configure:
   - Resource type: **Instance**
   - Instance: Select your production instance
   - Private IP address: Leave as default
   - Allow reassociation: **No** (unchecked)
4. Click **"Associate"**

> **Important Notes on Elastic IPs:**
> - Elastic IPs are free when associated with a running instance
> - Unassociated Elastic IPs incur charges
> - Each region has a default limit of 5 Elastic IPs (can be increased via support request)
> - Document the Elastic IP address for DNS configuration

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Oct 17 2025 | Sai Kiran Boppana | Initial runbook creation |
| 1.1 | Oct 17 2025 | Sai Kiran Boppana | Add EC2 Virtual Server Provisioning |

---

**Next Section:** 2. RDS (Database) Configuration

