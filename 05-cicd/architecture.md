# CI/ CD Architecture for Rosenwald Fund Collection
## Overview
#### This document outlines the CI/CD architecture for deploying Omeka S modules and themes using GitHub Actions and AWS Systems Manager (SSM). The process follows a main-branch development strategy with manual tag-based deployment triggers to isolate release flows into Dev, Test, Stage, and Prod environments. This approach ensures secure, repeatable deployments with built-in backup, rollback, and validation mechanisms. All deployments happen inside a private subnet via a secure Bastion and SSM command execution pipeline. 

## Flow Chart
![Flow Chart Image](file:///Users/kosisochukwuogbuanya/Desktop/Screenshot%202025-10-27%20at%201.51.02%E2%80%AFPM.png) 

## **Step 1: Developer Pushes Code to main** 

Development activity begins when a contributor pushes their changes to the main branch of the module or theme repository. This branch acts as the canonical source of truth and is protected by GitHub branch policies that enforce pull requests and peer review. No deployment is triggered at this stage — the push is simply the first phase of code integration into the project’s central version of history. 

## **Step 2: Merge Approved → Code Lands in main** 

After review and approval, a pull request is merged into the main branch. This action finalizes the CI (Continuous Integration) portion of the pipeline but does not yet initiate deployment. By decoupling merger and deployment, the team maintains strict control over what gets released and when allowing time for tagging, testing, and review before actual deployment begins. 

## **Step 3: Tag Created by Authorized User** 

To initiate deployment, a Git tag is created manually using the format vX.Y.Z-env (e.g., v1.0.2-dev). These tags serve as explicit version markers and indicate both the release version and target environment. Tagging is limited to authorized users only — especially for sensitive environments like prod to ensure accountability and prevent accidental overwrites. 

## **Step 4: GitHub Actions Triggered on Tag Push** 

Once the tag is pushed, GitHub Actions kicks off the deployment workflow (deploy.yml). This automated process is responsible for identifying the module or theme to be deployed, parsing the environment context from the tag, and launching the necessary build and upload steps. The use of tags ensures that deployments are intentional, auditable, and reproducible. 

## **Step 5: Environment Parsed from Tag Suffix** 

The GitHub Action parses the tag suffix (e.g., -dev, -stage, -prod) to determine which infrastructure stack the deployment should target. This dynamic extraction drives the loading of environment-specific secrets, such as EC2 instance IDs, S3 bucket names, and IAM roles — all securely stored in GitHub Secrets and mapped accordingly within the CI/CD pipeline. 

## **Step 6: Deploy Job Runs on GitHub Runner EC2** 

A dedicated self-hosted GitHub Runner (an EC2 instance behind Bastion) performs the actual deployment. It packages the module or theme into a ZIP artifact (Module_TAG.zip), uploads it to S3, and then uses AWS SSM to instruct the appropriate Omeka EC2 instance to deploy it. This setup keeps internal EC2s private while enabling safe orchestration from GitHub. 

## **Step 7: AWS SSM Command Issued to EC2** 

Deployment on the Omeka server is initiated by executing an SSM send-command (RunShellScript document). This command tells the EC2 to download the ZIP file from S3 into /tmp, unzip it into /tmp/deployed-module/, make the deploy.sh script executable, and finally run it with environment and tag parameters. The use of SSM avoids any need for direct SSH access from the GitHub runner. 

## **Step 8: Target EC2 Executes deploy.sh** 

On the Omeka EC2 instance, deploy.sh handles the core deployment logic. It first checks whether the deployment is for a theme or a module, then creates a timestamped backup of the current version. It removes any existing folder, unzips the new version into the correct location, runs composer install if required, and adjusts ownership/permissions. This ensures a clean and atomic deployment every time. 

## **Step 9: Post-Deploy Validations Run** 

Once deployment is complete, a series of validation checks are performed. These include verifying file existence, checking the HTTP response from the Omeka front-end, and scanning logs for runtime issues. These checks provide a final assurance that the deployment is completed successfully, and the site is operational. 

## **Step 10: Monitoring Execution via GitHub** 

Back on the GitHub runner, the workflow polls the status of the SSM command using get-command-invocation. It retrieves stdout and stderr output from the EC2, streams logs into the GitHub workflow logs, and captures the final execution status. This live feedback loop helps developers debug issues or confirm success without logging into any server directly. 

## **Step 11: Successful Deploy → Notify and Archive** 

If all validations pass, uploads deployment logs and backup to the configured S3 bucket. These backups are versioned and timestamped to allow for historical auditing or manual recovery if needed. The deployment is marked as successful in the GitHub summary. 

## **Step 12: Failure → Rollback and Alert** 

If any post-deploy validation fails, the system automatically triggers a rollback using the timestamped backup. The prior version is restored, logs are written to both EC2 and S3, and a failure alert is issued with details about what went wrong. This ensures that broken deployments do not affect users, and developers are quickly notified of the failure state. 

## **Step 13: IAM Roles and Security Controls** 

The architecture uses tightly scoped IAM roles: 

- The GitHub runner EC2 role allows ssm:SendCommand, s3:PutObject, and s3:GetObject 

- The Omeka EC2 role allows s3:GetObject for pulling artifacts 
Access to both EC2s is restricted via Bastion. Private subnets, key-based SSH, and SSM execution help ensure that no direct public access exists. All SSM logs are preserved for auditing and debugging. 

## **One-Time Deployment Items** 

These tasks are performed **once per EC2 environment** (Dev, Test, Stage, Prod) and are **explicitly excluded from GitHub Actions**. They are considered stable unless the environment is rebuilt or significantly changed. 

**Manual Tasks Per Environment**: 

- Upload site logo and favicon via Omeka Admin UI 

- Set site title and description in Omeka Admin UI 

- Create Omeka admin user account (if not pre-provisioned) 

- Click **Install** and **Activate** for required modules via Admin UI 

- Select default public theme and basic navigation structure (if applicable) 

- Verify that necessary PHP extensions are installed and configured 

- Ensure .htaccess overrides and Apache directives are active 

- Set file system permissions for /files and other storage paths 

- Verify or create Omeka storage directory (storage.local.dir) if used 

**One-Time Patched Items**: 

- .htaccess: Set SetEnv APPLICATION_ENV to match environment (development, production, etc.) --sudo nano /var/www/html/omeka-s/.htaccess 

- config.ini: Toggle logger setting per environment (On in Dev/Test, Off in Stage/Prod) 

  --sudo nano /var/www/html/omeka-s/config/local.config.php 

These actions are deliberately **excluded from the CI/CD pipeline** to prevent override of manual configurations. All one-time setup activities should be logged in docs/env-init.md or included in handoff documentation for that environment. 
