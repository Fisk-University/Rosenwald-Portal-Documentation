# Rosenwald Project: Documentation

This repository contains **technical documentation and implementation resources** for replicating the Julius Rosenwald Fund Digital Portal developed at Fisk University.

It is designed for **engineers, developers, and technical consultants** supporting archivists, librarians, and digital-humanities teams at **HBCUs or similar institutions** with the funding and infrastructure to deploy complex digital collections.

These materials include AWS infrastructure guidance, OmekaS installation and configuration procedures, CI/CD workflows, and custom module and theme documentation used in the Fisk University Rosenwald Portal implementation.

Institutions may implement this system at different levels of complexity depending on staffing, funding, and long-term sustainability goals, but in **all cases an engineer will be required**.

## About the Rosenwald Project

The Rosenwald Project is a Mellon Foundation-funded initiative that digitizes the history of **Julius Rosenwald**, a philanthropist who helped fund thousands of schools for African Americans in the early 20th century.

Fisk University’s digital portal not only houses these materials but also provides **a model for replication** - a complete, open framework that other HBCUs can use to digitize, describe, and share their own collections securely and sustainably.

This documentation represents the **technical backbone of that blueprint**, ensuring that future institutions can extend, customize, and deploy the system with confidence.

## Documentation Table of Contents

### 00-templates/
*Foundation templates and guidelines for all documentation*

| Document | Description |
|----------|-------------|
| `sop-template.md` | Standard Operating Procedure template for all repeatable tasks |
| `style-guide.md` | Documentation style guide for consistency across all materials |

### 01-infrastructure/
*AWS infrastructure setup and LAMP stack configuration*

| Document | Description |
|----------|-------------|
| `aws-setup.md` | EC2, S3, RDS, IAM, Route 53, SSL configuration guide |
| `lamp-stack.md` | LAMP installation and hardening SOP |
| `dns-nameservers-route53.md` | Connecting .edu domains to AWS Route 53 via nameservers |
| `php-performance-and-upload-limits.md` | Increasing PHP upload/memory limits securely |
| `imagemagick-pdf-thumbnails.md` | Secure ImageMagick policy.xml configuration for PDF thumbnails |

### 02-omeka-installation/
*Omeka S installation and core configuration procedures*

| Document | Description |
|----------|-------------|
| `omeka-install.md` | Omeka S installation and configuration |
| `database-ini.md` | Database connection setup and environment variables |

### 03-modules/
*Configuration and usage guides for essential Omeka S modules*

| Document | Description |
|----------|-------------|
| `zipimport.md` | ZIP importer usage, metadata schema, and nested file example |
| `unitedsearch.md` | Unified search configuration and customization |
| `dualpropertysearch.md` | Dual-property search module usage and caching strategy |

### 04-theme/
*Theme customization and optional enhancements*

| Document | Description | Status |
|----------|-------------|--------|
| `rosenwald-theme.md` | Theme customization, SCSS structure, and responsive design rules | Required |
| `custom-vocabulary.md` | Define a unique controlled vocabulary in Omeka-S | Optional |
| `lat-long-calculator.md` | Excel Macro to print lat/long coordinates of counties provided | Optional |

### 05-cicd/
*Continuous Integration/Deployment and operational procedures*

| Document | Description |
|----------|-------------|
| `environments.md` | Multi-environment (dev/stage/prod) overview |
| `backup-restore.md` | Automated S3 backups and restore procedures |
| `deployment-checklist.md` | Pre-launch deployment and verification checklist |

### 06-troubleshooting/
*Common issues and their resolutions*

| Document | Description |
|----------|-------------|
| `common-errors.md` | Frequent Apache, PHP, and Omeka S issues |
| `zipimporter-errors.md` | How to read and troubleshoot ZipImporter logs |

### 07-replication-blueprint/
*Guidelines for replicating this infrastructure at other institutions*

| Document | Description |
|----------|-------------|
| `overview.md` | Replication overview for other HBCUs |
| `cost-model.md` | Estimated infrastructure costs and funding tiers |
| `security-practices.md` | Security baseline and IAM best practices |


## Contribution Guidelines

### Before Contributing
1. Read the [Style Guide](00-templates/style-guide.md) for documentation standards
2. Use the [SOP Template](00-templates/sop-template.md) when creating new procedures
3. Ensure all commands and procedures have been tested in a development environment

### Contribution Process

1. **Branch Creation**: Create a feature branch from `main`
```bash
   git checkout -b docs/your-feature-name
```

2. **Documentation Standards**:
   - Follow the established style guide
   - Include all required SOP sections when applicable
   - Test all commands and procedures
   - Add screenshots where helpful

3. **Commit Messages**: Use conventional commit format
   - docs: Add AWS RDS configuration guide
   - fix: Correct PHP memory limit in LAMP setup
   - update: Revise SSL certificate renewal procedure

4. **Pull Request**:
   - Title: Clear description of changes
   - Description: List all modifications and their purpose
   - Reviewers: Tag appropriate team members

### Review Criteria
- [ ] Follows style guide formatting
- [ ] Technical accuracy verified
- [ ] Commands tested and working
- [ ] Links are valid
- [ ] No sensitive information exposed
- [ ] Proper file naming conventions


## Quick Start

### For New Team Members
| Step | Action | Document |
|------|--------|----------|
| 1 | Understand infrastructure | `01-infrastructure/aws-setup.md` |
| 2 | Review application setup | `02-omeka-installation/omeka-install.md` |
| 3 | Reference for issues | `06-troubleshooting/common-errors.md` |

### For HBCU Partners Looking to Replicate
| Step | Action | Document |
|------|--------|----------|
| 1 | Start here | `07-replication-blueprint/overview.md` |
| 2 | Budget planning | `07-replication-blueprint/cost-model.md` |
| 3 | Follow SOPs in sequence | Start with infrastructure setup |

### For Daily Operations
| Task | Document |
|------|----------|
| Backup procedures | `05-cicd/backup-restore.md` |
| Pre-deployment checks | `05-cicd/deployment-checklist.md` |
| Issue resolution | `06-troubleshooting/` folder |

## Documentation Status

| Category | Status | Items |
|----------|--------|-------|
| **Completed** | ✅ | SOP Template, Style Guide, Repository Structure |
| **In Progress** | 🔄 | Infrastructure SOPs, Omeka Installation Guides, Module Configuration |
| **Planned** | 📅 | Complete Troubleshooting Guides, Replication Blueprint Details, Video Walkthroughs |


## Related Resources

| Resource | Link |
|----------|------|
| Omeka S Documentation | [Official Omeka S Docs](https://omeka.org/s/docs/) |
| AWS Documentation | [AWS Documentation](https://docs.aws.amazon.com/) |


## Support

| Issue Type | Contact Method |
|------------|----------------|
| Technical Issues | Create an issue in this repository |
| General Inquiries | Contact the project maintainer |
| Security Concerns | Email security team directly (do not post publicly) |


## License

This documentation is published under the [Server Side Public License (SSPL-1.0)](https://www.mongodb.com/legal/licensing/server-side-public-license).


*Last Updated: 2025-11-03*
*Authors: LaTaevia Berry, & Sai Kiran Boppana*
