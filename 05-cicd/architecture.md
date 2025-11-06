# CICD-004: CI/CD Architecture and Setup Guide

## Purpose

This document explains the Fisk-Rosenwald CI/CD architecture and provides setup instructions for GitHub self-hosted runners and AWS Systems Manager (SSM). The architecture uses a tag-based deployment strategy that eliminates SSH dependencies, providing secure and auditable deployments through AWS SSM.

## Applies To
- Teams implementing GitHub Actions with AWS SSM
- DevOps engineers setting up self-hosted runners
- Developers needing access to EC2 instances via SSM
- Institutions replicating the Rosenwald CI/CD framework

## Maintainer
RWCF Engineers

## Last Updated
Nov 6 2025

---

## 1. CI/CD Architecture Overview

### 1.1 Architecture Components

The Fisk-Rosenwald CI/CD architecture integrates GitHub Actions with AWS Systems Manager to provide secure, automated deployments without SSH access. The system uses Git tags to trigger deployments, with the environment determined by the tag suffix (e.g., v1.0.0-dev, v1.0.0-prod).

**Key Components:**
- **GitHub Repository**: Stores code with branch protection on main
- **GitHub Actions**: Orchestrates the deployment workflow
- **Self-Hosted GitHub Runner**: EC2 instance that executes GitHub Actions jobs
- **AWS Systems Manager (SSM)**: Secure command execution on target EC2s
- **S3 Buckets**: Storage for deployment artifacts and backups
- **Target EC2 Instances**: Omeka-S application servers per environment

 ![Fisk-Rosenwald CI/CD Architecture](./Fisk-Rosenwald-CI_CD.drawio-2%20(1).png)

### 1.2 Deployment Flow Explanation

The deployment process follows a strict sequence to ensure security and reliability:

**Step 1: Code Integration**
Developers push code to feature branches and create pull requests. After review and approval, code is merged into the main branch. No deployment occurs at this stage - the main branch serves as the source of truth for all deployments.

**Step 2: Tag-Based Deployment Trigger**
Authorized users create Git tags following the format `v{version}-{environment}`. The tag serves two purposes: it marks a specific version for deployment and indicates the target environment through its suffix. Only users with appropriate permissions can create production tags.

**Step 3: GitHub Actions Workflow**
When a tag is pushed, GitHub Actions automatically triggers the deployment workflow. The workflow checks out the repository, creates a ZIP archive of the module or theme (named Module_TAG.zip), and uploads it to the appropriate S3 bucket based on the environment.

**Step 4: SSM Command Execution**
The GitHub Runner sends an SSM command to the target EC2 instance. This command downloads the ZIP from S3, extracts it to /tmp/deployed-module/, makes the deploy.sh script executable, and runs it with environment and tag parameters.

**Step 5: Target Server Deployment**
The EC2 instance receives the SSM command and executes the deployment script. This script backs up the current version, deploys the new code, sets appropriate permissions, and runs validation checks.

**Step 6: Validation and Rollback**
Post-deployment validation checks verify that the application is functioning correctly. If validation passes, logs and backups are uploaded to S3 and success is reported. If validation fails, the system automatically rolls back to the previous version and sends failure notifications.

### 1.3 Security Controls

The architecture implements multiple security layers:

- **No SSH Keys**: All deployments use AWS IAM roles and SSM, eliminating SSH key management
- **Private Subnets**: EC2 instances are not directly accessible from the internet
- **IAM Role Separation**: GitHub Runner and target EC2s have minimal, separate permissions
- **Audit Logging**: All SSM commands are logged in CloudTrail
- **Encrypted Artifacts**: Deployment packages in S3 are encrypted at rest

---

## 2. Setting Up GitHub Self-Hosted Runner

### 2.1 Overview

A self-hosted GitHub runner provides a dedicated environment for executing GitHub Actions workflows. Unlike GitHub-hosted runners, self-hosted runners remain in your AWS infrastructure, allowing access to private resources without exposing them to the internet.

### 2.2 Configuration Steps

**Note**: The below setup has already been completed for Fisk-Rosenwald's Project

**Step 1: Navigate to Your GitHub Organization**
Go to your organization's GitHub page where the repositories are hosted.

**Step 2: Open the Runners Configuration Page**
Click on Settings → Actions → Runners.

**Step 3: Add a New Self-Hosted Runner**
Click "New Runner" and then select "New self-hosted runner".

**Step 4: Select the Runner Image and Architecture**
- Operating System: Choose Linux (used in the Rosenwald Project)
- Architecture: Select x64

**Step 5: Start a Session to the GitHub Runner EC2 (Private Instance)**
Open your terminal and run the following command to connect via AWS SSM:
```bash
aws ssm start-session --target i-xxxxxxxxxxxxxxxxx
```
Replace `i-xxxxxxxxxxxxxxxxx` with the instance ID of your GitHub Runner EC2.

**Step 6: Run the Setup Commands from GitHub**
Follow the commands displayed on the GitHub runner page (typically includes downloading the runner, configuring it with your token, and setting it up as a service).

**Step 7: Start the Runner**
After setup, make the runner active with:
```bash
./svc.sh install
./svc.sh start
```
Alternatively, use `./run.sh` if not installing as a service.

### 2.3 Runner Maintenance

The GitHub runner should be configured as a system service to ensure it starts automatically after reboots. Regular maintenance includes updating the runner software when new versions are released and monitoring the runner's health through GitHub's interface.

---

## 3. Accessing AWS Systems Manager (SSM)

### 3.1 Overview

AWS Systems Manager (SSM) allows secure, direct access to EC2 instances through Session Manager, without requiring SSH or opening any inbound ports. This is ideal for environments like the Rosenwald Project where private EC2 instances should not be exposed to the public internet.

### 3.2 Who Needs SSM Access

Each team member who needs access to EC2 instances via SSM must perform this setup on their local machine. SSM does not use shared SSH keys — it authenticates and authorizes each person based on their AWS IAM identity. If you're part of the deployment or debugging team, this is required for your role.

### 3.3 Prerequisites

- Your local machine must have AWS CLI installed
- You must have an IAM user or role with `ssm:StartSession` permissions
- EC2 instance must have SSM Agent installed and running
- EC2 instance IAM role must have `AmazonSSMManagedInstanceCore` policy attached

### 3.4 Setup Instructions

**Step 1: Install AWS CLI (If Not Already Installed)**

Check if you have it installed:
```bash
aws --version
```

If not, follow instructions: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

**Step 2: Configure AWS CLI**

Run the following command and provide your credentials:
```bash
aws configure
```

You'll be prompted to enter:
- AWS Access Key ID
- AWS Secret Access Key
- Default region (e.g., `us-east-1`)
- Output format (e.g., `json`)

To create a new AWS access key: IAM → Users → Open your user ID → Under summary create new access key

**Step 3: Start a Session to the EC2 Instance**

To connect to a specific instance:
```bash
aws ssm start-session --target i-xxxxxxxxxxxxxxxxx
```

Replace `i-xxxxxxxxxxxxxxxxx` with your EC2 instance ID. In our case, it's the GitHub EC2 private runner. This command opens an interactive shell on the EC2 through a secure, tunneled AWS connection.

**Step 4: Verify SSM Access**

To see which instances have been attached to AWS SSM, use:
```bash
aws ssm describe-instances
```

This will list all EC2 instances that are registered with Systems Manager and available for session connections.

### 3.5 Security Notes

Every SSM session is logged in CloudTrail, making this method more secure and auditable than SSH. It is safe to use in environments that require strict credential separation and logging. Session activity is recorded including:
- Who initiated the session
- When the session started and ended
- Commands executed during the session
- Session termination reason

---

## 4. IAM Roles and Permissions

### 4.1 GitHub Runner EC2 Role

The GitHub Runner requires permissions to upload artifacts to S3 and send commands via SSM:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::rwcf-artifacts-*/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ssm:SendCommand",
        "ssm:GetCommandInvocation"
      ],
      "Resource": "*"
    }
  ]
}
```

### 4.2 Target EC2 Instance Role

Target EC2 instances only need permissions to download artifacts from S3:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::rwcf-artifacts-*/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ssm:UpdateInstanceInformation",
        "ssmmessages:*",
        "ec2messages:*"
      ],
      "Resource": "*"
    }
  ]
}
```

### 4.3 Developer IAM Permissions

Developers need SSM session permissions to access EC2 instances:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:StartSession",
        "ssm:TerminateSession",
        "ssm:DescribeInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 5. Troubleshooting

### 5.1 Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| **SSM session fails to start** | SSM agent not running | Verify SSM agent status on EC2 |
| **GitHub runner offline** | Service stopped | Restart runner service via SSM |
| **Deployment hangs** | SSM command timeout | Check CloudWatch logs for errors |
| **S3 access denied** | IAM permissions missing | Verify role has s3:GetObject permission |
| **Validation fails repeatedly** | Application error | Check Omeka logs on target EC2 |

### 5.2 Debugging Commands

```bash
# Check SSM agent status on EC2
sudo systemctl status amazon-ssm-agent

# View GitHub runner logs
journalctl -u actions.runner.*.service -f

# Check deployment logs
tail -f /var/log/deployment.log

# Verify S3 access
aws s3 ls s3://rwcf-artifacts-dev/
```

---

## Related Documents

- `environments.md` - Environment-specific configurations
- `deployment-checklist.md` - Pre and post-deployment validation
- `backup-restore.md` - Backup procedures and recovery

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Nov 6 2025 | Sai Kiran Boppana | Initial CI/CD architecture documentation |

---

*This document is part of the Rosenwald Fund Collection documentation suite.*
