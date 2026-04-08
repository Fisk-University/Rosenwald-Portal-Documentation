# Replication Blueprint Overview

This document defines the architectural model, implementation strategy, and decision framework used to build the Rosenwald digital collections platform.

It is intended to function as a repeatable blueprint for institutions seeking to deploy a sustainable, cloud-based Omeka S environment for digital collections.

---

## Purpose of This Blueprint

This blueprint is designed to:

- Provide a clear starting point for institutions with some, but limited, technical infrastructure
- Document a scalable architecture that supports long-term collection growth
- Explain the rationale behind key technology decisions
- Enable phased adoption based on institutional capacity

This is not a one-size-fits-all implementation. Institutions are expected to adapt this model based on staffing, funding, and collection size.

---

## System Architecture Model

The Rosenwald platform is built as a modular system composed of the following layers:

### 1. Application Layer - Provided At No Cost
- Omeka S (PHP-based content management system)
- Fisk Custom modules for ingestion and search
- Fisk Custom theme for public interface

### 2. Data Layer - Low Or No Cost If Local
- MySQL database (hosted on RDS or locally)
- Structured metadata aligned with institutional standards

### 3. Storage Layer - Low Cost
- S3 object storage for media assets
- Optional local storage for minimal deployments

### 4. Infrastructure Layer - Low Cost
- EC2 instances for application hosting
- VPC for network isolation
- Security groups and IAM roles for access control

### 5. Operational Layer - Low Cost
- Backup strategy (snapshots and automated backups)
- Deployment workflow (manual or CI/CD)
- Monitoring and logging

Each layer can be independently scaled or simplified depending on institutional needs.

---

## Deployment Paths

Institutions should select a deployment path based on available resources.

### Path A: Full Production Architecture

This model prioritizes stability, redundancy, and controlled deployment workflows.

**Characteristics**
- Multiple environments (dev, test, stage, prod)
- Dedicated database instances or schemas
- Separate storage buckets for production and non-production
- IAM role segmentation
- Bastion host for secure access
- Automated backups and monitoring

**Tradeoffs**
- Higher cost
- Increased operational overhead
- Requires technical staffing

---

### Path B: Minimal Deployment Architecture

This model prioritizes accessibility and reduced cost.

**Characteristics**
- Single EC2 instance
- Single database
- Local or shared S3 storage
- Manual deployment process
- Basic backup strategy

**Tradeoffs**
- Limited isolation between environments
- Increased risk during updates
- Reduced scalability

---

## Phased Adoption Model

Institutions are not expected to implement the full architecture immediately.

Recommended progression:

### Phase 1: Initial Deployment
- Single EC2 instance
- Local database or small RDS instance
- Manual ingestion workflows

### Phase 2: Stabilization
- Introduce S3 for asset storage
- Implement regular backups
- Improve security group configurations

### Phase 3: Expansion
- Separate staging environment
- Introduce module-based ingestion workflows
- Begin performance tuning

### Phase 4: Production Maturity
- Full environment separation
- Automated deployment pipelines
- Monitoring and alerting systems

---

## Technology Selection Rationale

### Omeka S
- Widely adopted in the digital humanities community
- Flexible metadata modeling
- Extensible through modules and themes
- No licensing cost

### AWS Infrastructure
- Scalable and widely supported
- Enables remote management without reliance on local IT infrastructure
- Supports both minimal and advanced deployment models

### S3 for Storage
- Durable and cost-effective for large media collections
- Reduces storage pressure on application servers

### RDS for Database
- Managed database service reduces administrative overhead
- Supports backup and recovery out of the box

---

## Cost vs Complexity Tradeoff

The architecture is intentionally designed to allow institutions to balance:

- Cost
- Technical complexity
- Operational risk

Institutions with limited resources should prioritize:

- Simplicity
- Stability
- Incremental growth

Institutions with greater capacity can adopt:

- Redundancy
- Automation
- Environment isolation

---

## Replication Considerations

When adopting this blueprint, institutions should evaluate:

- Available technical staff
- Budget constraints
- Expected collection size
- Long-term maintenance capacity

The system should be implemented at a level that can be maintained consistently over time.

---

## Key Takeaways

- This blueprint is modular and adaptable
- Institutions should start small and scale intentionally
- Cost optimization is a continuous process
- Long-term sustainability is more important than initial complexity

The Rosenwald implementation demonstrates that a production-grade digital collections platform can be achieved using open-source tools and cost-conscious cloud infrastructure.