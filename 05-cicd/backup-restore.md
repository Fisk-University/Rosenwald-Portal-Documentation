# CICD-002: Backup, Restore, and Disaster Recovery Procedures

## Purpose

This guide documents the comprehensive backup strategy for the Rosenwald project, including automated S3 file rotation, RDS snapshot retention policies, and detailed recovery procedures. The multi-layered backup approach ensures data can be recovered from various failure scenarios, from accidental file deletion to complete infrastructure loss. Each environment has tailored backup frequencies and retention periods based on criticality and compliance requirements.

## Applies To
- All RWCF Omeka-S deployments across Dev, Test, Stage, and Prod
- Database administrators managing RDS instances
- DevOps teams responsible for disaster recovery
- Compliance officers requiring audit trails

## Maintainer
RWCF Engineers

## Last Updated
Nov 6 2025

---

## 1. Backup Architecture Overview

### 1.1 Three-Layer Backup Strategy

The Rosenwald project implements three distinct backup layers, each protecting different aspects of the system. This redundancy ensures that multiple failure types can be recovered from without data loss. The layers work independently but complement each other for comprehensive protection.

**Backup Layers:**

| Layer | What's Protected | Backup Method | Recovery Time |
|-------|-----------------|---------------|---------------|
| **Application Files** | Media uploads, themes, modules | S3 versioning + lifecycle | Minutes |
| **Database** | All Omeka metadata, users, config | RDS automated snapshots | 15-30 minutes |
| **Full System** | Complete EC2 state | EBS snapshots | 1-2 hours |

### 1.2 Environment-Specific Backup Policies

Different environments require different backup strategies based on their criticality and rate of change. Production gets the most aggressive backup schedule, while development environments have minimal backups to reduce costs.

**Backup Frequency by Environment:**

| Component | Dev | Test | Stage | Prod |
|-----------|-----|------|-------|------|
| **S3 Media** | No backup | Daily sync | Versioning enabled | Versioning + replication |
| **RDS Database** | No automated | Daily, 1 retained | Daily, 7 retained | Hourly, 35 days retained |
| **EC2/EBS** | Never | Weekly | Daily | Daily + pre-deployment |

---

## 2. S3 Backup and Rotation Policies

### 2.1 S3 Versioning Configuration

S3 versioning maintains multiple versions of each file, protecting against accidental deletion or overwrite. For the Rosenwald project, production buckets maintain all versions for 90 days before transitioning to cheaper storage classes.

**Versioning Setup per Environment:**

```bash
# Production bucket with versioning
aws s3api put-bucket-versioning \
  --bucket rwcf-media-prod \
  --versioning-configuration Status=Enabled

# Stage bucket with versioning  
aws s3api put-bucket-versioning \
  --bucket rwcf-media-stage \
  --versioning-configuration Status=Enabled
```

### 2.2 S3 Lifecycle Policies

Lifecycle policies automatically move older versions through storage tiers to optimize costs while maintaining recoverability. Production data follows a gradual degradation path from instant-access to archive storage.

**Production S3 Lifecycle Rules:**

```json
{
  "Rules": [
    {
      "Id": "ArchiveOldVersions",
      "Status": "Enabled",
      "NoncurrentVersionTransitions": [
        {
          "NoncurrentDays": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "NoncurrentDays": 90,
          "StorageClass": "GLACIER"
        }
      ],
      "NoncurrentVersionExpiration": {
        "NoncurrentDays": 365
      }
    },
    {
      "Id": "DeleteIncompleteMultipartUploads",
      "Status": "Enabled",
      "AbortIncompleteMultipartUpload": {
        "DaysAfterInitiation": 7
      }
    }
  ]
}
```

**Storage Class Progression:**
- **Days 0-30**: STANDARD (immediate access)
- **Days 31-90**: STANDARD_IA (infrequent access, lower cost)
- **Days 91-365**: GLACIER (archive, 3-5 hour retrieval)
- **After 365 days**: Deleted permanently

### 2.3 S3 Cross-Region Replication

Production media files are replicated to a different AWS region for disaster recovery. This protects against regional outages and provides geographic redundancy. Replication happens asynchronously within minutes of file upload.

**Replication Configuration:**

```json
{
  "Role": "arn:aws:iam::ACCOUNT:role/s3-replication-role",
  "Rules": [
    {
      "ID": "ReplicateProdMedia",
      "Priority": 1,
      "Status": "Enabled",
      "Filter": {},
      "Destination": {
        "Bucket": "arn:aws:s3:::rwcf-media-prod-dr",
        "ReplicationTime": {
          "Status": "Enabled",
          "Time": {
            "Minutes": 15
          }
        },
        "StorageClass": "STANDARD_IA"
      }
    }
  ]
}
```

---

## 3. RDS Backup and Snapshot Policies

### 3.1 Automated RDS Backups

RDS provides automated backups that capture the entire database instance. These backups are taken during the specified backup window and retained according to environment-specific policies. Production maintains the longest retention for compliance and recovery needs.

**RDS Backup Configuration:**

| Environment | Backup Window | Retention Period | Backup Type |
|-------------|--------------|------------------|-------------|
| **Dev** | N/A | 0 days (disabled) | None |
| **Test** | 03:00-04:00 UTC | 1 day | Automated |
| **Stage** | 03:00-04:00 UTC | 7 days | Automated |
| **Prod** | 02:00-03:00 UTC | 35 days | Automated + Manual |

**Setting Production Backup Retention:**
```bash
aws rds modify-db-instance \
  --db-instance-identifier rwcf-rds-prod \
  --backup-retention-period 35 \
  --backup-window "02:00-03:00" \
  --apply-immediately
```

### 3.2 Manual RDS Snapshots

In addition to automated backups, manual snapshots are taken before major operations like schema changes, bulk imports, or production deployments. These snapshots are retained indefinitely until manually deleted.

**Manual Snapshot Naming Convention:**
```
Format: rwcf-{environment}-{event}-{YYYYMMDD-HHMMSS}

Examples:
rwcf-prod-pre-deployment-20240115-143000
rwcf-stage-pre-migration-20240120-090000
rwcf-prod-before-bulk-import-20240125-110000
```

**Creating Manual Snapshots:**
```bash
# Before production deployment
aws rds create-db-snapshot \
  --db-instance-identifier rwcf-rds-prod \
  --db-snapshot-identifier rwcf-prod-pre-deployment-$(date +%Y%m%d-%H%M%S)

# Verify snapshot completed
aws rds wait db-snapshot-completed \
  --db-snapshot-identifier rwcf-prod-pre-deployment-20240115-143000
```

### 3.3 Point-in-Time Recovery

RDS maintains transaction logs that enable point-in-time recovery to any second within the retention period. This granular recovery option is crucial for recovering from logical errors like accidental data deletion.

**Point-in-Time Recovery Process:**

1. **Identify Recovery Time**: Determine exact time before issue occurred
2. **Create New Instance**: Restore to new RDS instance (not over existing)
3. **Verify Data**: Check recovered data is correct
4. **Switch Application**: Update connection strings to new instance
5. **Cleanup**: Delete old corrupted instance after verification

```bash
# Restore to specific point in time
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier rwcf-rds-prod \
  --target-db-instance-identifier rwcf-rds-prod-recovery \
  --restore-time 2024-01-15T14:30:00.000Z
```

---

## 4. EC2/EBS Backup Procedures

### 4.1 EBS Volume Snapshots

EBS snapshots capture the entire state of EC2 storage volumes, including the operating system, applications, and configurations. These provide the fastest way to recover from EC2 failures or to clone environments.

**Automated Snapshot Schedule:**

```bash
# Create snapshot policy for production
aws dlm create-lifecycle-policy \
  --execution-role-arn arn:aws:iam::ACCOUNT:role/AWSDataLifecycleManagerRole \
  --description "Production daily snapshots" \
  --state ENABLED \
  --policy-details file://prod-snapshot-policy.json
```

**prod-snapshot-policy.json:**
```json
{
  "ResourceTypes": ["VOLUME"],
  "TargetTags": [
    {
      "Key": "Environment",
      "Value": "Production"
    }
  ],
  "Schedules": [
    {
      "Name": "DailySnapshots",
      "CreateRule": {
        "Interval": 24,
        "IntervalUnit": "HOURS",
        "Times": ["03:00"]
      },
      "RetainRule": {
        "Count": 14
      },
      "TagsToAdd": [
        {
          "Key": "Type",
          "Value": "DailyBackup"
        }
      ]
    }
  ]
}
```

### 4.2 AMI Creation

For major releases, full AMIs (Amazon Machine Images) are created to capture the complete server state. These serve as gold images for disaster recovery or rapid environment replication.

**AMI Creation Before Major Deployments:**
```bash
# Create AMI with no-reboot option
aws ec2 create-image \
  --instance-id i-prod-instance \
  --name "rwcf-prod-v2.0-release-$(date +%Y%m%d)" \
  --description "Pre-v2.0 release backup" \
  --no-reboot

# Tag the AMI for identification
aws ec2 create-tags \
  --resources ami-xxxxx \
  --tags Key=Environment,Value=Production \
         Key=Version,Value=2.0 \
         Key=Type,Value=PreRelease
```

---

## 5. Recovery Procedures

### 5.1 File Recovery from S3

Recovering deleted or corrupted files from S3 depends on whether versioning was enabled and how long ago the deletion occurred.

**Recovery Scenarios:**

**Scenario 1: Accidental File Deletion (Versioning Enabled)**
```bash
# List all versions of deleted file
aws s3api list-object-versions \
  --bucket rwcf-media-prod \
  --prefix "items/document-001.pdf"

# Restore specific version
aws s3api get-object \
  --bucket rwcf-media-prod \
  --key "items/document-001.pdf" \
  --version-id "abc123def456" \
  restored-file.pdf

# Copy back to original location
aws s3 cp restored-file.pdf \
  s3://rwcf-media-prod/items/document-001.pdf
```

**Scenario 2: Bulk Recovery from Date**
```bash
# Restore all files deleted after specific date
aws s3 sync s3://rwcf-media-prod s3://rwcf-media-prod \
  --delete-removed \
  --restore-request Days=1
```

### 5.2 Database Recovery Procedures

Database recovery follows different procedures based on the type of failure and recovery point objective (RPO).

**Recovery Decision Tree:**

1. **Data Corruption (Logical Error)**
   - Use Point-in-Time Recovery
   - RPO: Seconds before corruption
   - RTO: 30-45 minutes

2. **Complete Database Failure**
   - Restore from latest automated backup
   - RPO: Up to 24 hours (based on backup frequency)
   - RTO: 15-30 minutes

3. **Regional Outage**
   - Promote read replica in different region
   - RPO: Minutes (replication lag)
   - RTO: 5-10 minutes

**Database Restoration Steps:**

```bash
# Step 1: Create new RDS instance from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier rwcf-rds-prod-restored \
  --db-snapshot-identifier rwcf-prod-snapshot-20240115

# Step 2: Wait for restoration to complete
aws rds wait db-instance-available \
  --db-instance-identifier rwcf-rds-prod-restored

# Step 3: Update application configuration
# Edit Omeka database.ini with new endpoint
sudo nano /var/www/html/omeka-s/config/database.ini

# Step 4: Test application functionality
curl https://rosenwald.fisk.edu/health-check

# Step 5: After verification, delete old instance
aws rds delete-db-instance \
  --db-instance-identifier rwcf-rds-prod-corrupted \
  --skip-final-snapshot
```

### 5.3 Full EC2 Recovery

When an EC2 instance fails completely, recovery involves launching a new instance from the most recent snapshot or AMI.

**EC2 Recovery Process:**

```bash
# Option 1: Launch from AMI (fastest)
aws ec2 run-instances \
  --image-id ami-backup-20240115 \
  --instance-type t3.large \
  --subnet-id subnet-prod \
  --security-group-ids sg-prod-web \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=rwcf-prod-recovered}]'

# Option 2: Create volume from snapshot and attach
aws ec2 create-volume \
  --snapshot-id snap-daily-20240115 \
  --availability-zone us-east-1a \
  --volume-type gp3

aws ec2 attach-volume \
  --volume-id vol-xxxxx \
  --instance-id i-new-instance \
  --device /dev/xvdf
```

---

## 6. Backup Monitoring and Testing

### 6.1 Automated Backup Verification

Automated scripts verify backup integrity and alert on failures. These run daily and report to CloudWatch metrics for dashboard visualization.

**Backup Health Checks:**

```bash
#!/bin/bash
# backup-health-check.sh

# Check S3 backup age
LAST_BACKUP=$(aws s3 ls s3://rwcf-backups-prod/ --recursive \
  | sort | tail -n 1 | awk '{print $1}')
DAYS_OLD=$(( ($(date +%s) - $(date -d "$LAST_BACKUP" +%s)) / 86400 ))

if [ $DAYS_OLD -gt 1 ]; then
  aws cloudwatch put-metric-data \
    --namespace RWCF/Backups \
    --metric-name BackupAge \
    --value $DAYS_OLD \
    --dimensions Environment=Production,Type=S3
fi

# Verify RDS snapshot exists
SNAPSHOT_COUNT=$(aws rds describe-db-snapshots \
  --db-instance-identifier rwcf-rds-prod \
  --snapshot-type automated \
  --query 'length(DBSnapshots[?Status==`available`])' \
  --output text)

if [ $SNAPSHOT_COUNT -lt 1 ]; then
  echo "WARNING: No available RDS snapshots found"
  # Send SNS alert
fi
```

### 6.2 Quarterly Recovery Drills

Every quarter, the team performs recovery drills to ensure procedures work and team members remain familiar with the process. These drills rotate through different failure scenarios.

**Drill Schedule:**

| Quarter | Scenario | Environment | Success Criteria |
|---------|----------|-------------|------------------|
| Q1 | Database corruption | Stage | Restore to 1 hour ago |
| Q2 | EC2 failure | Test | Full recovery < 2 hours |
| Q3 | S3 data loss | Stage | Restore 100 files |
| Q4 | Regional outage | Prod-DR | Failover < 30 minutes |

---

## 7. Cost Optimization

### 7.1 Storage Cost Management

Backup storage can become expensive without proper lifecycle management. These strategies minimize costs while maintaining recovery capabilities.

**Cost Reduction Strategies:**

1. **Use Lifecycle Transitions**: Move old backups to cheaper storage classes
2. **Delete Redundant Snapshots**: Keep only necessary manual snapshots
3. **Compress Before Storage**: Reduce backup file sizes
4. **Regional Consolidation**: Store non-critical backups in cheaper regions

**Monthly Cost Review:**
```bash
# Generate backup storage cost report
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY \
  --metrics "BlendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE \
  --filter file://backup-cost-filter.json
```

### 7.2 Retention Policy Optimization

Retention periods balance recovery needs with storage costs. Regular review ensures policies remain appropriate for actual recovery requirements.

**Retention Review Questions:**
- How often are backups actually restored?
- What's the oldest backup ever needed?
- Can older backups move to cold storage?
- Are manual snapshots being cleaned up?

---

## 8. Compliance and Audit

### 8.1 Backup Audit Trail

All backup operations are logged for compliance and troubleshooting. Logs include who initiated backups, what was backed up, and verification results.

**Audit Log Locations:**
- S3 backup operations: CloudTrail logs
- RDS snapshots: RDS event logs
- Script executions: CloudWatch Logs
- Manual operations: S3 audit bucket

### 8.2 Regulatory Compliance

The backup strategy meets common higher education and grant funding requirements for data retention and disaster recovery.

**Compliance Checkpoints:**
- ✅ 35-day database retention (exceeds 30-day requirement)
- ✅ Geographic redundancy via S3 replication
- ✅ Encryption at rest for all backups
- ✅ Access logging for audit requirements
- ✅ Tested recovery procedures documented

---

## Related Documents

- `environments.md` - Environment-specific configurations
- `deployment-checklist.md` - Pre-deployment backup requirements
- `architecture.md` - Infrastructure diagrams showing backup flows

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Nov 6 2025 | Sai Kiran Boppana | Initial backup/restore documentation |

---

*This document is part of the Rosenwald Fund Collection documentation suite.*
