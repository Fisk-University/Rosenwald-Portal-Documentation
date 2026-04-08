# Common Errors

This document outlines frequently encountered issues within the Rosenwald Omeka S deployment and their corresponding resolutions. These errors are based on observed behavior across development, staging, and production environments.

---

## Application Errors

### 1. Apache Default Page ("It Works") Appears Instead of Omeka

**Cause**
Apache is serving the default document root instead of the Omeka S application directory.

**Resolution**
- Confirm Omeka is installed at `/var/www/html/omeka-s`
- Update Apache configuration:


### 2. error-router-no-match

**Cause**
Incorrect base URL or broken rewrite rules.

**Resolution**

-   Verify `.htaccess` exists in the Omeka root directory
-   Ensure mod_rewrite is enabled:

### 3. Infinite Redirect Loop

**Cause**
Misconfigured rewrite rules or duplicated base paths.

**Resolution**

-   Check for duplicated paths such as:
    `/omeka-s/s/site/page/homeomeka-s/...`
-   Reset Apache config to clean DocumentRoot
-   Clear browser cache and test in incognito

* * * * *

Database Issues
---------------

### 4\. Cannot Connect to Database (RDS)

**Cause**

-   Security group misconfiguration
-   Incorrect credentials
-   RDS instance not publicly accessible when required

**Resolution**

-   Verify RDS inbound rules allow EC2 instance access
-   Confirm credentials in `database.ini`
-   Check endpoint and port (default 3306)

* * * * *

### 5\. High Database Cost or Performance Degradation

**Cause**

-   Use of io2 storage without requirement
-   Multi-AZ enabled unnecessarily
-   Over-provisioned instance type

**Resolution**

-   Convert storage from io2 to gp3
-   Disable Multi-AZ for non-production environments
-   Right-size instance (e.g., db.t3.medium → db.t3.small if applicable)

* * * * *

File Handling Issues
--------------------

### 6\. Image Thumbnails Not Generating

**Cause**\
ImageMagick security policy blocks PDF or image processing.

**Resolution**\
Edit policy file:

sudo nano /etc/ImageMagick-6/policy.xml

Locate and modify:

<policy domain="coder" rights="none" pattern="PDF" />

Change to:

<policy domain="coder" rights="read|write" pattern="PDF" />

Restart Apache after changes.

* * * * *

### 7\. File Upload Fails Silently

**Cause**\
PHP upload limits too low.

**Resolution**\
Update `php.ini`:

upload_max_filesize = 128M\
post_max_size = 128M\
max_execution_time = 300

Restart Apache.

* * * * *

Infrastructure Issues
---------------------

### 8\. EC2 Disk Full

**Cause**\
Default root volume too small for ingestion workload.

**Resolution**

-   Increase EBS volume size via AWS console
-   Extend filesystem:

sudo growpart /dev/xvda 1\
sudo resize2fs /dev/xvda1

* * * * *

### 9\. SSL Certificate Not Applied

**Cause**\
Certbot did not properly bind certificate to Apache config.

**Resolution**\
Verify:

/etc/apache2/sites-available/000-default-le-ssl.conf

Run:

sudo certbot --apache

* * * * *

Observability Issues
--------------------

### 10\. No Traffic Data Available

**Cause**\
Analytics not configured or ALB access logs disabled.

**Resolution**

-   Enable ALB access logs
-   Configure CloudWatch metrics
-   Optionally integrate external analytics (GA4)

* * * * *

Summary
-------

Most system issues fall into the following categories:

-   Misconfigured Apache routing
-   Incorrect AWS permissions or networking
-   Insufficient PHP or storage configuration
-   Over-provisioned or misconfigured RDS instances

Resolving these systematically ensures stable long-term operation.