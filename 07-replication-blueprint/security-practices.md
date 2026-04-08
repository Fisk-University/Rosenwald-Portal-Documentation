# Security Practices

This document outlines baseline security practices for operating an Omeka S deployment on AWS.

---

## Network Security

- Deploy EC2 instances within a VPC
- Use private subnets for development environments
- Restrict SSH access via bastion host
- Limit inbound traffic using security groups

---

## Access Control

- Use IAM roles instead of static credentials
- Apply least privilege principles
- Rotate credentials regularly

---

## Data Protection

- Enable automated RDS backups
- Use encrypted storage for RDS and EBS
- Store media assets in S3 with controlled access policies

---

## Application Security

- Keep Omeka S and modules updated
- Validate all user input
- Restrict admin panel access

---

## Infrastructure Hardening

- Disable unused ports
- Monitor logs via CloudWatch
- Implement alerting for abnormal activity

---

## Summary

Security should be treated as a continuous process, not a one-time configuration. Even minimal deployments should implement baseline protections to ensure data integrity and system stability.