# 📃 Rosenwald Project: Documentation

This repository helps **archivists, librarians, and digital humanities professionals** at HBCUs and similar institutions replicate the digital archive built at Fisk University for the **Julius Rosenwald Fund** collection. It contains clear, non-technical guidance on how to set up, configure, and launch your own Omeka S-based site with minimal overhead and long-term sustainability.

> ✨ This is the public-friendly version of our internal developer documentation. If you're a developer, please request access to the private repo for system architecture and credentials.

---

## 📆 About the Rosenwald Project

The Rosenwald Project is a Mellon Foundation-funded initiative that digitizes the history of **Julius Rosenwald**, a philanthropist who helped fund thousands of schools for African Americans in the early 20th century. 

Fisk University launched this digital archive to preserve and share Rosenwald-related documents with the world. Just as importantly, we documented every step to help **other HBCUs and cultural memory institutions** replicate this effort with low cost and little technical burden.

This guide is part of our mission to make digital preservation more accessible and sustainable across the archival community.

---

📚 Documentation Table of Contents
00-templates/
Foundation templates and guidelines for all documentation.

sop-template.md – Standard Operating Procedure template for all repeatable tasks
style-guide.md – Documentation style guide for consistency across all materials

01-infrastructure/
AWS infrastructure setup and LAMP stack configuration.

aws-setup.md – EC2, S3, RDS, IAM, Route 53, SSL configuration guide
lamp-stack.md – LAMP installation and hardening SOP
dns-nameservers-route53.md – Connecting .edu domains to AWS Route 53 via nameservers
php-performance-and-upload-limits.md – Increasing PHP upload/memory limits securely
imagemagick-pdf-thumbnails.md – Secure ImageMagick policy.xml configuration for PDF thumbnails

02-omeka-installation/
Omeka S installation and core configuration procedures.

omeka-install.md – Omeka S installation and configuration
database-ini.md – Database connection setup and environment variables

03-modules/
Configuration and usage guides for essential Omeka S modules.

zipimport.md – ZIP importer usage, metadata schema, and nested file example
unitedsearch.md – Unified search configuration and customization
dualpropertysearch.md – Dual-property search module usage and caching strategy

04-theme/
Theme customization and optional enhancements.

rosenwald-theme.md – Theme customization, SCSS structure, and responsive design rules
custom-vocabulary.md – Optional: Define a unique controlled vocabulary in Omeka-S
lat-long-calculator.md – Optional: Excel Macro to print lat/long coordinates of counties provided

05-cicd/
Continuous Integration/Deployment and operational procedures.

environments.md – Multi-environment (dev/test/stage/prod) overview
backup-restore.md – Automated S3 backups and restore procedures
deployment-checklist.md – Pre-launch deployment and verification checklist

06-troubleshooting/
Common issues and their resolutions.

common-errors.md – Frequent Apache, PHP, and Omeka S issues
zipimporter-errors.md – How to read and troubleshoot ZipImporter logs

07-replication-blueprint/
Guidelines for replicating this infrastructure at other institutions.

overview.md – Replication overview for other HBCUs
cost-model.md – Estimated infrastructure costs and funding tiers
security-practices.md – Security baseline and IAM best practices


🤝 Contribution Guidelines
Before Contributing

Read the Style Guide for documentation standards
Use the SOP Template when creating new procedures
Ensure all commands and procedures have been tested in a development environment

Contribution Process

Branch Creation: Create a feature branch from main

bash   git checkout -b docs/your-feature-name

Documentation Standards:

Follow the established style guide
Include all required SOP sections when applicable
Test all commands and procedures
Add screenshots where helpful


Commit Messages: Use conventional commit format

   docs: Add AWS RDS configuration guide
   fix: Correct PHP memory limit in LAMP setup
   update: Revise SSL certificate renewal procedure

Pull Request:

Title: Clear description of changes
Description: List all modifications and their purpose
Reviewers: Tag appropriate team members



Review Criteria

 Follows style guide formatting
 Technical accuracy verified
 Commands tested and working
 Links are valid
 No sensitive information exposed
 Proper file naming conventions


🚀 Quick Start
For New Team Members

Start with [01-infrastructure/aws-setup.md] to understand the infrastructure
Review [02-omeka-installation/omeka-install.md] for application setup
Reference [06-troubleshooting/common-errors.md] when issues arise

For HBCU Partners Looking to Replicate

Begin with [07-replication-blueprint/overview.md]
Review [07-replication-blueprint/cost-model.md] for budget planning
Follow the SOPs in sequence, starting with infrastructure setup

For Daily Operations

Use [05-cicd/backup-restore.md] for backup procedures
Reference [05-cicd/deployment-checklist.md] before any deployment
Check [06-troubleshooting/] for issue resolution


📋 Documentation Status
Completed Documents

✅ SOP Template
✅ Style Guide
✅ Repository Structure

In Progress

🔄 Infrastructure SOPs
🔄 Omeka Installation Guides
🔄 Module Configuration

Planned

📅 Complete Troubleshooting Guides
📅 Replication Blueprint Details
📅 Video Walkthroughs


🔗 Related Resources

Project Repository: GitHub - RWCF Main
Omeka S Documentation: Official Omeka S Docs
AWS Documentation: AWS Documentation


📞 Support
For questions or assistance:

Technical Issues: Create an issue in this repository
General Inquiries: Contact the project maintainer
Security Concerns: Email security team directly (do not post publicly)



## License

This documentation is published under the [Server Side Public License (SSPL-1.0)](https://www.mongodb.com/legal/licensing/server-side-public-license).