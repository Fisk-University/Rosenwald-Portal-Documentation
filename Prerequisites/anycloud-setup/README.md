# AnyCloud Module Setup for Omeka-S

## Step 1: Install the Module
- Download the ZIP from https://omeka.org/s/modules/
- Unzip and place in `/modules` directory of Omeka-S

## Step 2: Enable
- Go to Admin dashboard > Modules > Install AnyCloud

## Step 3: Configure
- Click **Configure** and enter:
  - AWS Access Key
  - AWS Secret Key
  - Bucket name
- Or update `config/local.config.php` for server-side setup

## Usage
Once configured, new media uploads will be stored in your selected cloud bucket (e.g., S3).
