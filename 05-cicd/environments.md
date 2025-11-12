# CICD-001: Multi-Environment Architecture and Promotion Flow

## Purpose

This guide documents the four-tier environment strategy (Dev → Test → Stage → Prod) for the Rosenwald project, including the promotion workflows, environment-specific configurations, and the critical database clone strategy. The multi-environment approach ensures code quality through progressive validation while protecting production stability. Each environment serves a specific purpose in the development lifecycle, with strict controls on data movement and deployment permissions.

## Applies To
- All Omeka-S deployments following the RWCF framework
- Teams implementing GitHub Actions with AWS SSM
- Projects requiring strict environment separation for compliance
- HBCU institutions replicating this infrastructure

## Maintainer
RWCF Engineers

## Last Updated
Nov 6 2025

---

## 1. Environment Overview

### 1.1 Four-Tier Architecture

The Rosenwald project uses four distinct environments, each with its own EC2 instance, RDS database, and S3 bucket. This separation ensures that development experiments never affect production, while providing multiple validation stages before public release. The environments are completely isolated at the network level, residing in different subnets with separate security groups.

**Environment Purposes:**

| Environment | Purpose | Users | Data Type |
|-------------|---------|-------|-----------|
| **Dev** | Active development and experimentation | Developers only | Synthetic test data |
| **Test** | Automated testing and QA validation | QA team + Developers | Test datasets |
| **Stage** | Production mirror for final validation | Stakeholders + Team | Production-like data |
| **Prod** | Live public site | End users | Real archival data |

### 1.2 Infrastructure Isolation

Each environment maintains complete infrastructure separation to prevent cross-contamination and ensure security. Resources are tagged with their environment designation, and IAM policies enforce that resources from one environment cannot access another. This isolation extends to databases, file storage, and application servers.

**Resource Naming Convention:**
```
Format: rwcf-{resource}-{environment}
Examples:
- rwcf-ec2-dev
- rwcf-rds-stage
- rwcf-s3-prod
```

---

## 2. Code Promotion Strategy

### 2.1 Git Tag-Based Deployment

Code promotion uses Git tags to trigger deployments through GitHub Actions. Tags follow a strict format that includes both version number and target environment. This approach creates an audit trail of what was deployed when and prevents accidental deployments to the wrong environment.

**Tag Format:**
```
v{major}.{minor}.{patch}-{environment}

Examples:
v1.0.0-dev     # Deploy version 1.0.0 to Dev
v1.0.0-test    # Promote same version to Test
v1.0.0-stage   # Promote to Stage after testing
v1.0.0-prod    # Final production deployment
```

The version number remains constant as code promotes through environments, making it easy to track which version is running where. Only the environment suffix changes during promotion.

### 2.2 Promotion Flow Rules

Code must progress through environments sequentially. Skipping environments is prohibited to ensure proper validation at each stage. This gated approach catches issues before they reach production.

**Required Promotion Path:**
```
main branch → Dev → Test → Stage → Prod
     ↓         ↓      ↓       ↓       ↓
   (merge)  (v1-dev)(v1-test)(v1-stage)(v1-prod)
```

**Promotion Requirements:**

**Dev → Test:**
- All unit tests passing
- Code review completed
- No critical security vulnerabilities
- Module/theme loads without errors

**Test → Stage:**
- All automated tests passing
- QA team sign-off
- Performance benchmarks met
- No regression issues identified

**Stage → Prod:**
- Stakeholder approval obtained
- Full backup completed
- Deployment window scheduled
- Rollback plan documented

### 2.3 Environment-Specific Configurations

Each environment requires specific configurations that are NOT promoted with code. These settings remain stable per environment and are managed outside the CI/CD pipeline to prevent override of environment-specific values.

**Configuration Differences:**

```php
// Dev Environment
'logger' => ['log' => true, 'priority' => Logger::DEBUG],
'display_errors' => true,
'cache' => false,

// Stage Environment  
'logger' => ['log' => true, 'priority' => Logger::WARN],
'display_errors' => false,
'cache' => true,

// Prod Environment
'logger' => ['log' => true, 'priority' => Logger::ERROR],
'display_errors' => false,
'cache' => true,
'https' => 'required'
```

---

## 3. Database Clone Strategy

### 3.1 Clone Restrictions

Database cloning is intentionally restricted to prevent data contamination and maintain environment integrity. Production data must never flow backward into lower environments, and test data must never pollute production. These restrictions are enforced at the script level with hard blocks.

**Allowed Clone Operations:**
```
Stage → Prod: ✅ ALLOWED (via manual script)
Dev → Test: ❌ BLOCKED
Test → Stage: ❌ BLOCKED  
Prod → Stage: ❌ BLOCKED
Any other combination: ❌ BLOCKED
```

The single allowed path (Stage → Prod) ensures that only validated, stakeholder-approved data reaches production. This happens only during major releases or content updates, not routine deployments.

### 3.2 Clone Process

The database clone from Stage to Prod is a manual process requiring SSH access to the staging server. This deliberate friction ensures cloning is intentional and authorized. The process cannot be triggered accidentally through Git tags or CI/CD workflows.

**Clone Execution Steps:**

1. **SSH to Staging Server:**
```bash
aws ssm start-session --target i-staging-instance-id
```

2. **Navigate to Scripts Directory:**
```bash
cd /opt/scripts/
```

3. **Execute Clone Script:**
```bash
sudo ./clone-db.sh
```

4. **Confirm Operation:**
The script will prompt for confirmation and display what will happen:
```
WARNING: This will replace ALL production data with staging data
Source: rwcf-rds-stage
Target: rwcf-rds-prod
Backup will be created at: s3://rwcf-backups-prod/pre-clone/
Continue? (type 'yes-clone-to-prod' to proceed):
```

### 3.3 Clone Script Operations

The clone script performs several critical operations to ensure data integrity and provide rollback capability. Each step is logged for audit purposes.

**Script Workflow:**

1. **Environment Validation**: Confirms Stage → Prod is the operation
2. **Credentials Retrieval**: Fetches database passwords from AWS Secrets Manager
3. **Production Backup**: Creates full backup before any changes
4. **Data Export**: Dumps entire staging database
5. **Data Import**: Loads staging data into production
6. **Verification**: Runs basic checks to confirm import success
7. **Cleanup**: Removes temporary files, logs operation

**Backup Locations:**
```
Local: /backup/pre-clone-YYYYMMDD-HHMMSS.sql
S3: s3://rwcf-backups-prod/pre-clone/YYYYMMDD-HHMMSS.sql
```

---

## 4. AWS SSM Deployment Architecture

### 4.1 GitHub Runner + SSM Flow

The deployment architecture uses a self-hosted GitHub runner on a private EC2 instance combined with AWS Systems Manager for secure command execution. This eliminates the need for SSH keys in CI/CD while maintaining security.

**Deployment Sequence:**

1. Developer creates Git tag (e.g., `v1.0.0-dev`)
2. GitHub Actions workflow triggers on tag
3. Self-hosted runner (private EC2) picks up job
4. Runner uploads artifact to S3
5. Runner sends SSM command to target EC2
6. Target EC2 downloads from S3 and deploys
7. Validation checks run
8. Status reported back to GitHub

### 4.2 IAM Role Configuration

Each component has minimal IAM permissions following the principle of least privilege. Roles are scoped to specific environments to prevent cross-environment access.

**GitHub Runner EC2 Role:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:SendCommand",
        "ssm:GetCommandInvocation"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Environment": ["dev", "test", "stage"]
        }
      }
    },
    {
      "Effect": "Allow", 
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Resource": "arn:aws:s3:::rwcf-artifacts-*/*"
    }
  ]
}
```

**Target EC2 Role:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::rwcf-artifacts-{environment}/*"
    },
    {
      "Effect": "Allow",
      "Action": ["ssm:UpdateInstanceInformation"],
      "Resource": "*"
    }
  ]
}
```

---

## 5. Environment-Specific Settings

### 5.1 One-Time Manual Configurations

Certain configurations are set once per environment and explicitly excluded from CI/CD to prevent override. These settings are documented here but managed manually through Omeka admin or direct server access.

**Per-Environment Manual Tasks:**

| Task | Dev | Test | Stage | Prod |
|------|-----|------|-------|------|
| **Application Environment** | development | testing | staging | production |
| **Debug Mode** | Enabled | Enabled | Disabled | Disabled |
| **Error Display** | On | On | Off | Off |
| **Cache** | Off | Off | On | On |
| **SSL Required** | No | No | Yes | Yes |
| **Backup Frequency** | Never | Daily | Daily | Hourly |

### 5.2 Environment Variables

Each environment uses specific environment variables set in `.htaccess` or system configuration. These control application behavior without code changes.

**Example .htaccess per Environment:**

```apache
# Development
SetEnv APPLICATION_ENV development
SetEnv DB_HOST rwcf-rds-dev.region.rds.amazonaws.com
SetEnv S3_BUCKET rwcf-media-dev

# Production
SetEnv APPLICATION_ENV production
SetEnv DB_HOST rwcf-rds-prod.region.rds.amazonaws.com
SetEnv S3_BUCKET rwcf-media-prod
SetEnv FORCE_SSL true
```

---

## 6. Monitoring and Validation

### 6.1 Health Checks

Each environment has automated health checks to verify deployment success and ongoing operation. These run after each deployment and continuously via CloudWatch.

**Validation Points:**
- HTTP response code (200 expected)
- Database connectivity
- S3 bucket accessibility  
- PHP error log status
- Module activation status
- Theme rendering

### 6.2 Rollback Triggers

Automatic rollback occurs if post-deployment validations fail. The system restores the previous version from backup without manual intervention.

**Rollback Conditions:**
- HTTP 500 errors detected
- Database connection failure
- Critical PHP errors in logs
- Missing required modules
- S3 access denied

---

## 7. Security Boundaries

### 7.1 Network Segmentation

Each environment operates in isolated network segments with no cross-communication allowed. This prevents lateral movement in case of compromise.

**Network Architecture:**
```
VPC: 10.0.0.0/xx
├── Dev Subnet: 10.0.1.0/24
├── Test Subnet: 10.0.2.0/24  
├── Stage Subnet: 10.0.3.0/24
├── Prod Subnet: 10.0.4.0/24
└── Bastion Subnet: 10.0.99.0/24
```

### 7.2 Access Controls

Access to each environment is role-based with increasing restrictions as you move toward production.

| Role | Dev | Test | Stage | Prod |
|------|-----|------|-------|------|
| **Developers** | Full | Full | Read | None |
| **QA Team** | Read | Full | Full | None |
| **DevOps** | Full | Full | Full | Limited |
| **Stakeholders** | None | None | Read | None |
| **Production Support** | None | None | Read | Full |

---

## Related Documents

- `deployment-checklist.md` - Pre-deployment validation steps
- `backup-restore.md` - Backup and recovery procedures
- `architecture.md` - System architecture diagrams

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Nov 6 2025 | Sai Kiran Boppana | Initial environment documentation |

---

*This document is part of the Rosenwald Fund Collections documentation suite.*
