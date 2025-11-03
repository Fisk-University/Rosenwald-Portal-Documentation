# MODULE-002: ZipImport Configuration and Usage Guide

## Purpose

ZipImport is a custom Omeka-S module that revolutionizes how archivists handle bulk uploads of digital collections. Instead of uploading items one by one through the web interface (which could take weeks for large collections), ZipImport allows you to prepare hundreds or thousands of items offline, package them in a ZIP file with a CSV metadata spreadsheet, and import everything in one operation. This is essential for institutions digitizing large archival collections where manual entry isn't feasible.

## Applies To
- Omeka-S installations handling bulk digitization projects
- Archives with collections of 100+ items to import
- Institutions using AWS S3 for media storage via AnyCloud
- Projects requiring nested media (multiple images per item)

## Maintainer
RWCF Development Team

## Last Updated
Nov 3 2025

---

## 1. Prerequisites

Before using ZipImport, your server environment must be properly configured to handle large file uploads and long-running processes. Each requirement addresses a specific limitation that would otherwise prevent successful bulk imports.

| Requirement | Description | Why It's Critical |
|------------|-------------|-------------------|
| **Omeka-S Core** | Version 4.0+ installed and running | Module uses modern Omeka APIs |
| **CSV Import Module** | Official Omeka module installed | ZipImport extends CSV Import's functionality |
| **AnyCloud Module** | Installed and configured with AWS credentials | Enables S3 storage for large media collections |
| **PHP Limits** | Increased to handle 1GB+ uploads | Default 8MB limit blocks bulk imports |
| **Memory Allocation** | At least 1GB RAM available to PHP | Processing large ZIPs requires memory |
| **Execution Time** | PHP timeout set to 300+ seconds | Large imports take several minutes |
| **Composer** | Installed on server | Required for module dependencies |

---

## 2. Understanding ZipImport's Role

### 2.1 What ZipImport Does

ZipImport bridges the gap between offline collection preparation and online publication. Archivists can organize their materials using familiar tools like Excel and file folders, then upload everything at once. The module unpacks the ZIP file, reads the CSV metadata, matches images to metadata rows, creates Omeka items, uploads media to S3 (if configured), and generates all thumbnails automatically.

The power of ZipImport is in handling the tedious parts automatically. It creates proper Omeka relationships between items and media, maintains your folder organization as context, handles special characters in filenames, and provides detailed logs of what was imported or skipped. This automation saves hundreds of hours of manual work.

### 2.2 Relationship with AnyCloud and AWS S3

While ZipImport can work with local storage, it truly shines when paired with AnyCloud and AWS S3. Local server storage quickly becomes expensive and hard to manage with thousands of high-resolution images. S3 provides virtually unlimited storage at low cost, automatic backups and redundancy, CDN capabilities for fast global access, and separation of media from your application server.

The workflow with S3 is seamless. ZipImport extracts files from the ZIP, AnyCloud automatically uploads them to your S3 bucket, Omeka stores just the S3 URLs in its database, and visitors' browsers download directly from S3. This keeps your server responsive even with massive collections.

---

## 3. Installation and Configuration

### 3.1 Installing the Module

The installation process requires command-line access because ZipImport has PHP dependencies that must be installed via Composer. This is more complex than typical Omeka modules but necessary for the advanced functionality.

```bash
# Navigate to Omeka's modules directory
cd /var/www/html/omeka-s/modules

# Download the module from GitHub
sudo wget https://github.com/Fisk-University/ZipImport/archive/refs/heads/CI/CD-Testing.zip

# Unzip and rename to standard name
sudo unzip CI/CD-Testing.zip
sudo mv ZipImport-CI-CD-Testing ZipImport

# Install PHP dependencies
cd ZipImport
sudo composer install

# Set proper permissions
cd ..
sudo chown -R www-data:www-data ZipImport
sudo chmod -R 755 ZipImport
```

After installation, activate the module in Omeka's admin panel under Modules. If you see errors about missing dependencies, the composer install step likely failed. Run it again with verbose output to see what went wrong.

### 3.2 Configuring PHP for Large Uploads

The default PHP configuration assumes you're uploading small files like profile pictures. For archival imports, you need to modify several limits. These changes must be made in your PHP configuration file, not in Omeka settings.

```bash
# Find your active php.ini file
php -i | grep "Loaded Configuration File"

# Edit the configuration
sudo nano /etc/php/8.1/fpm/php.ini
```

Update these critical values and understand why each matters:

```ini
; Maximum size for uploaded files (per file)
; Set to 1000M to handle large archival TIFFs
upload_max_filesize = 1000M

; Maximum size for all POST data combined
; Must be >= upload_max_filesize
post_max_size = 1000M

; Memory available to PHP scripts
; Needed for processing large ZIPs in memory
memory_limit = 1024M

; How many files can be uploaded at once
; ZIP might contain hundreds of images
max_file_uploads = 500

; How long a script can run (seconds)
; Large imports need time to process
max_execution_time = 600

; How long to wait for POST data
; Uploading 1GB takes time on slower connections
max_input_time = 600
```

After making changes, restart PHP and Apache:
```bash
sudo systemctl restart php8.1-fpm
sudo systemctl restart apache2
```

Verify the changes took effect by checking System Information in Omeka's admin footer. If values haven't updated, you may have edited the wrong php.ini file or forgotten to restart services.

### 3.3 Configuring AnyCloud for S3 Storage

AnyCloud connects Omeka to AWS S3, but it needs your AWS credentials and bucket information. This setup is crucial for production sites with large collections. First, create an S3 bucket in AWS with appropriate permissions, then configure AnyCloud with those credentials.

In Omeka admin, navigate to Modules → AnyCloud → Configure:

**AWS Credentials Section:**
- **Access Key ID**: Your AWS IAM user's access key (starts with AKIA...)
- **Secret Access Key**: The secret key (keep this secure!)
- **Default Region**: Where your bucket is located (e.g., us-east-1)

**S3 Bucket Settings:**
- **Bucket Name**: Your unique bucket name (e.g., rosenwald-media-files)
- **Path Prefix**: Optional folder within bucket (e.g., omeka-uploads/)
- **Storage Class**: STANDARD for frequently accessed files

**Important Security Note**: Create a dedicated IAM user for Omeka with permissions only for your specific bucket. Never use root AWS credentials. The IAM policy should allow s3:PutObject, s3:GetObject, s3:DeleteObject, and s3:ListBucket actions only.

---

## 4. Preparing Files for Import

### 4.1 Understanding the ZIP Structure

The ZIP file structure is critical for successful imports. ZipImport expects a specific organization that mirrors how items relate to media files. The basic principle is that your CSV file provides metadata while your folder structure provides the actual files, and they must align perfectly.

**Basic Structure for Single Media Items:**
```
import-package.zip
├── metadata.csv
├── image001.jpg
├── image002.jpg
├── image003.tiff
└── document001.pdf
```

In this structure, each row in metadata.csv corresponds to one file. The CSV must have a column (usually called "filename" or "file") that exactly matches each filename including the extension.

**Nested Structure for Multi-Media Items:**
```
import-package.zip
├── metadata.csv
└── items/
    ├── school001/
    │   ├── front-view.jpg
    │   ├── rear-view.jpg
    │   └── architectural-plans.pdf
    ├── school002/
    │   ├── main-building.jpg
    │   ├── classroom.jpg
    │   ├── students-1950.jpg
    │   └── dedication-ceremony.jpg
    └── school003/
        └── exterior.jpg
```

With nested structure, each folder represents one item (one row in the CSV), and all files within that folder become media attached to that single item. The CSV should reference the folder name, not individual files.

### 4.2 CSV Metadata Format

The CSV file is the brain of your import - it tells Omeka what metadata to associate with each item. The structure must be precise, with column headers exactly matching Omeka's expected format. The first row must contain field names that map to Dublin Core or custom vocabularies.

**Essential columns your CSV should include:**

| Column Name | Purpose | Example |
|-------------|---------|---------|
| **dcterms:title** | Item title | "Rosenwald School - Madison County" |
| **dcterms:description** | Full description | "Two-teacher school built in 1925..." |
| **dcterms:date** | Creation date | "1925" |
| **dcterms:spatial** | Location | "Madison County, Alabama" |
| **file** | Filename or folder | "school001" or "image001.jpg" |
| **dcterms:type** | Resource type | "Image" or "Document" |

**Critical CSV Requirements:**
- Must be saved as UTF-8 encoding (Excel often saves as Windows-1252, which breaks special characters)
- Use consistent date formats throughout
- Don't use commas within fields unless the field is properly quoted
- Empty cells are fine - not every item needs every field
- Column order doesn't matter, but names must be exact

### 4.3 File Naming Best Practices

File naming seems trivial but causes many import failures. Omeka and web servers have restrictions on what characters can appear in filenames. Following these conventions prevents mysterious failures during import.

**Safe naming conventions:**
- Use only letters, numbers, hyphens, and underscores
- No spaces (use hyphens or underscores instead)
- No special characters (é, ñ, etc.)
- Keep names under 200 characters
- Always include file extensions
- Be consistent with capitalization

**Examples:**
- ✅ Good: `rosenwald-school-madison-001.jpg`
- ✅ Good: `1925_dedication_ceremony.pdf`
- ❌ Bad: `Rosenwald School (Madison) #001.jpg`
- ❌ Bad: `école-française.jpg`

---

## 5. Import Process

### 5.1 Using the Import Interface

The import interface is accessible from the Omeka admin dashboard in the lower-left menu as "Zip Import". The process is straightforward but understanding each option helps ensure successful imports.

**Step-by-step process:**

1. **Navigate to Zip Import**: Found in the left sidebar of admin dashboard

2. **Select your ZIP file**: Use the file browser to select your prepared ZIP. The upload begins immediately after selection, so ensure you've selected the correct file.

3. **Choose Import Settings**:
   - **Item Set**: Select which collection these items belong to
   - **Item Template**: Apply a metadata template if you have one
   - **Automap with Simple Labels**: Check this box - it helps match CSV columns to Omeka fields automatically

4. **Review Field Mapping**: After upload, you'll see how CSV columns map to Omeka fields. Verify that essential fields like title and description are mapped correctly. Adjust any incorrect mappings using the dropdowns.

5. **Start Import**: Click Import and monitor the progress bar. Don't navigate away or close the browser during import.

### 5.2 Monitoring Import Progress

Large imports can take 10-30 minutes depending on size and server speed. The progress interface shows several important metrics that help you understand what's happening. The progress bar shows percentage complete, but more importantly, the status messages show which file is currently being processed.

Watch for warning messages about skipped files. These aren't necessarily errors - the import continues, but those files won't be included. Common reasons for skips include unsupported file formats, missing files referenced in CSV, or corrupted image files.

The import log is your best friend for troubleshooting. It records every action taken, including successful item creation, file upload results, and any errors encountered. Save this log after import for reference.

### 5.3 Post-Import Validation

After import completes, don't assume everything worked perfectly. Systematic validation helps catch issues early before the collection goes public.

**Validation checklist:**
1. **Check item count**: Does the number of items created match your CSV rows?
2. **Spot-check metadata**: Open several items and verify metadata imported correctly
3. **Verify media attachments**: Ensure images are attached to the right items
4. **Test S3 uploads**: If using AnyCloud, confirm media files are in your S3 bucket
5. **Review import log**: Look for any warnings or errors you might have missed

---

## 6. Troubleshooting Common Issues

### 6.1 File-Image Mismatch Scenarios

This is the most common issue with ZipImport. Your CSV references files that don't exist in the ZIP, or files exist that aren't referenced in the CSV. Each scenario requires different solutions.

**Scenario 1: CSV references missing image**

Your CSV has a row with `file: school001.jpg` but school001.jpg isn't in the ZIP file. This happens when files are renamed after creating the CSV or forgotten during ZIP creation.

*Symptoms*: Item gets created but has no media attached. Import log shows "File not found: school001.jpg"

*Solutions*:
- Verify exact filename match including extension (.jpg vs .jpeg)
- Check for case sensitivity (School001.jpg vs school001.jpg)
- Ensure file is actually in the ZIP (not just in your folder)
- Look for spaces or special characters in filenames

**Scenario 2: Image exists but no CSV entry**

Your ZIP contains image files that aren't referenced in any CSV row. These orphaned files are ignored during import.

*Symptoms*: Files remain in ZIP but never appear in Omeka. No errors in log since ZipImport only processes files referenced in CSV.

*Solutions*:
- Add CSV rows for orphaned images
- Remove unnecessary images from ZIP
- Use nested folders if images belong to existing items

**Scenario 3: Nested folder structure problems**

When using folders for multi-image items, the CSV must reference the folder name exactly, not the files inside.

*Symptoms*: Items created but missing some or all images from their folders

*Example fix*:
```
Correct CSV: file column contains "school001"
ZIP structure: school001/image1.jpg, school001/image2.jpg

Incorrect CSV: file column contains "school001/image1.jpg"
```

### 6.2 Memory and Timeout Errors

Large imports can exhaust server resources, causing failures partway through import. These errors are frustrating because you lose progress and must restart.

**Fatal error: Allowed memory size exhausted**

This means PHP ran out of RAM while processing your ZIP. Large ZIP files are extracted entirely into memory, so a 500MB ZIP might need 1GB+ of RAM.

*Solutions*:
- Increase memory_limit in php.ini to 2048M
- Split large imports into smaller batches
- Compress images before creating ZIP
- Use web-optimized formats (JPEG) instead of archival (TIFF)

**Maximum execution time exceeded**

The script took longer than PHP's time limit. This happens with slow servers or very large imports.

*Solutions*:
- Increase max_execution_time to 1200 (20 minutes)
- Process fewer items per import
- Optimize images to reduce processing time
- Check server load - import during off-peak hours

### 6.3 Character Encoding Issues

Character encoding problems cause garbled text in metadata, especially with historical collections containing names with diacritics or non-English text. Excel is particularly problematic as it doesn't handle UTF-8 well by default.

**Symptoms:**
- Accented characters appear as � or strange symbols
- Apostrophes become â€™
- Em-dashes become â€"

**Solutions:**

1. **Save CSV correctly from Excel:**
   - Use "Save As" → "CSV UTF-8 (Comma delimited)"
   - Not just "CSV" which uses Windows-1252

2. **Use Google Sheets instead:**
   - Download as CSV automatically uses UTF-8
   - Better handling of special characters

3. **Convert existing CSV to UTF-8:**
   ```bash
   # Check current encoding
   file -i metadata.csv
   
   # Convert to UTF-8
   iconv -f WINDOWS-1252 -t UTF-8 metadata.csv > metadata-utf8.csv
   ```

### 6.4 S3 Upload Failures

When using AnyCloud, files might import to Omeka but fail to upload to S3. This creates items with broken media links.

**Common causes and solutions:**

**Invalid AWS credentials:**
- Verify Access Key and Secret Key are correct
- Ensure IAM user has PutObject permission
- Check credentials haven't expired

**Bucket permissions:**
- Bucket must allow uploads from your IAM user
- CORS configuration must permit your domain
- Bucket policy shouldn't block uploads

**Network timeouts:**
- Large files might timeout on slow connections
- Implement retry logic in AnyCloud settings
- Consider uploading large files directly to S3

**Region mismatches:**
- Ensure bucket region matches configuration
- Use correct regional endpoint in settings

---

## 7. Advanced Usage

### 7.1 Batch Processing Strategies

For collections with thousands of items, single massive imports become unwieldy. Strategic batch processing makes imports manageable and recoverable if issues arise.

**Recommended batch sizes:**
- 100-200 items for initial testing
- 500-1000 items for production imports
- Never exceed 2000 items in one import

**Organizing batches effectively:**
- Group by collection or sub-collection
- Separate by media type (images vs documents)
- Chronological batches for dated materials
- Geographic batches for location-based collections

This approach allows you to validate each batch before proceeding, recover from failures without losing everything, and track progress through large projects.

### 7.2 Automating Import Preparation

For repeated imports, automation saves time and reduces errors. Simple scripts can prepare your import packages consistently.

**Python script example for CSV generation:**
```python
import csv
import os

# Scan folder for images and create CSV
with open('metadata.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerow(['dcterms:title', 'file', 'dcterms:type'])
    
    for filename in os.listdir('images'):
        if filename.endswith(('.jpg', '.tiff', '.pdf')):
            title = filename.replace('-', ' ').replace('_', ' ')
            writer.writerow([title, filename, 'Image'])
```

This basic script creates a CSV from a folder of files. Expand it to extract metadata from filenames, read EXIF data from images, or pull data from existing databases.

---

## 8. Best Practices

### 8.1 Pre-Import Checklist

Run through this checklist before every import to prevent common issues:

- [ ] CSV saved as UTF-8 encoding
- [ ] All filenames match exactly between CSV and ZIP
- [ ] No special characters in filenames
- [ ] ZIP file is under 1GB (or PHP limits adjusted)
- [ ] Test import with 5-10 items first
- [ ] Backup exists of current Omeka database
- [ ] S3 bucket has sufficient space (if using AnyCloud)
- [ ] Server has low load for processing

### 8.2 Documentation Requirements

Document every import for future reference and troubleshooting:

1. **Import manifest**: List of what was imported, when, and by whom
2. **Source documentation**: Where original files came from
3. **Processing notes**: Any modifications made to files or metadata
4. **Error log**: Problems encountered and how they were resolved
5. **Validation results**: Confirmation that import succeeded

This documentation proves invaluable when questions arise months later about specific items or collections.

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Nov 3 2025 | Sai Kiran Boppana | Initial ZipImport documentation |
| 1.1 | Nov 3 2025 | Sai Kiran Boppana  | Added troubleshooting scenarios |

---

*This document is part of the Rosenwald-Fund Collection documentation suite.*
