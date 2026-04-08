# Cost Model and Infrastructure Budgeting

This document provides a practical cost model for operating an Omeka S digital collections platform using the Rosenwald architecture.

It is intended to help institutions estimate monthly infrastructure costs and understand what drives those costs.

---

## Monthly Cost Baseline (Rosenwald Reference)

A production-ready deployment aligned with this blueprint operates at approximately:

**$1,500 – $2,000 per month**

This range reflects:
- A production environment
- At least one non-production environment
- External storage for media assets
- Managed database infrastructure that has been right-sized for actual usage

This baseline should be used as a reference point for institutions planning a similar deployment.

---

## Cost Breakdown by Component

### 1. Database (RDS)

**Estimated Monthly Cost**
$700 – $1,100

**What You Are Paying For**
- Managed MySQL database
- Automated backups
- Storage allocation (gp3)
- Optional high availability configuration

**Primary Cost Drivers**
- Instance size (db.t3.small vs db.t3.medium)
- Storage allocation and growth over time
- Multi-AZ configuration (significant cost increase)

**Guidance**
- Use gp3 storage unless high IOPS are required
- Disable Multi-AZ outside production environments
- Right-size instances based on observed usage
- Re-evaluate storage allocation quarterly

**Note**
RDS remains the largest contributor to total cost, but proper sizing significantly reduces unnecessary spend.

---

### 2. Compute (EC2)

**Estimated Monthly Cost**
$500 – $650

**What You Are Paying For**
- Web server hosting Omeka S
- CPU and memory resources
- Attached storage volumes (EBS)

**Primary Cost Drivers**
- Instance type (t3.small vs t3.medium)
- Number of environments (dev, stage, prod)
- Storage volume size

**Guidance**
- Start with smaller instance types and scale up as needed
- Reduce duplicate environments where possible
- Monitor utilization before increasing capacity

---

### 3. Storage (S3 and EBS)

**Estimated Monthly Cost**
$5 – $75+

**What You Are Paying For**
- Storage of media assets (images, PDFs)
- Backup storage
- Data transfer

**Primary Cost Drivers**
- Total number of files
- File sizes (high-resolution scans increase cost)
- Retention policies

**Guidance**
- Use S3 for media storage instead of local disk
- Implement lifecycle policies for long-term cost control
- Expect storage cost to grow gradually with ingestion

---

### 4. Supporting Services

**Estimated Monthly Cost**
$50 – $100

Includes:
- VPC networking
- Load balancing (if enabled)
- Monitoring (CloudWatch)
- DNS (Route 53)

These costs are relatively stable and low compared to database and compute.

---

## Total Cost Model

| Deployment Level              | Estimated Monthly Cost |
|------------------------------|------------------------|
| Minimal Deployment           | $200 – $600            |
| Moderate Deployment          | $600 – $1,200          |
| Full Production Deployment   | $1,500 – $2,000        |

---

## What Increases Cost

- Enabling Multi-AZ database deployments
- Over-provisioning RDS instance size
- Running multiple full environments (dev, test, stage, prod)
- Storing large volumes of high-resolution media
- Using higher-cost storage types unnecessarily

---

## What Reduces Cost

- Using gp3 storage for both RDS and EBS
- Right-sizing database and compute instances
- Limiting environments to what is actively used
- Starting with a minimal deployment and scaling gradually
- Deferring high-availability features until required

---

## Recommended Starting Configuration (Cost-Conscious)

For most institutions, the following configuration provides a balanced starting point:

- 1 EC2 instance (t3.small or t3.medium)
- 1 RDS instance (db.t3.small or db.t3.medium, gp3 storage)
- 1 S3 bucket for media storage
- Basic backup configuration
- No Multi-AZ initially

**Expected Monthly Cost**
$400 – $900

This configuration can support:
- Moderate ingestion volume
- Public access to collections
- Incremental scaling over time

---

## Scaling Strategy

Institutions should plan to increase spending in response to:

- Growth in collection size
- Increased user traffic
- Need for redundancy and uptime guarantees
- Expansion of development workflows

Cost should scale with institutional need, not upfront assumptions.

---

## Key Takeaways

- The database is the primary cost driver, but can be controlled through proper sizing
- Compute costs are predictable and scale with environment count
- Storage costs grow gradually and should be monitored over time
- A functional deployment does not require a full production architecture on day one

This model allows institutions to begin at a manageable cost level and expand as capacity increases.