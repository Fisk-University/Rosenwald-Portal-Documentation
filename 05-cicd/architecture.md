# CI/ CD Architecture for Rosenwald Fund Collection
## Overview
#### This document outlines the CI/CD architecture for deploying Omeka S modules and themes using GitHub Actions and AWS Systems Manager (SSM). The process follows a main-branch development strategy with manual tag-based deployment triggers to isolate release flows into Dev, Test, Stage, and Prod environments. This approach ensures secure, repeatable deployments with built-in backup, rollback, and validation mechanisms. All deployments happen inside a private subnet via a secure Bastion and SSM command execution pipeline. 

# Flow Chart
![Flow Chart Image](/Users/kosisochukwuogbuanya/Desktop/Screenshot 2025-10-27 at 1.51.02 PM.png)