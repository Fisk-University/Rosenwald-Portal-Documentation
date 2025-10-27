# Complete Guide to Setting Up an Omeka-S Server on AWS

## Purpose
This documentation provides a complete deployment pathway for establishing a production-ready Omeka-S digital repository on AWS infrastructure. Each configuration step includes technical rationale and architectural context, ensuring your institution can reliably process and serve archival collections at scale, including batch imports up to 1GB through optimized PHP-FPM and ImageMagick configurations.

## Who This Guide Is For
- Archivists and librarians setting up digital collections
- IT staff at universities and cultural institutions
- Anyone implementing the Rosenwald Project framework at their institution
- Technical experience required: Basic comfort with following instructions carefully

## What You'll Build
A complete web server running Omeka-S with:
- Ubuntu Linux operating system (the foundation)
- Apache web server (delivers web pages to browsers)
- MySQL database (stores all your data)
- PHP programming language (runs Omeka-S)
- Optimized settings for large file imports
- Security configurations for production use

---

# PART 1: Creating Your Cloud Server on AWS

 ##### *For a visual walkthrough of launching and configuring an EC2 instance through the AWS console, including security groups, key pairs, and initial Ubuntu server connection, see [EC2 Setup →](./01-infrastructure/aws-setup.md)

## Understanding AWS and EC2

### Step 1: Create Your AWS Account

Navigate to aws.amazon.com and create an account if you don't have one.

### Step 2: Navigate to EC2 Service

Once logged into AWS:
1. Look for the search box at the top of the page
2. Type "EC2" and select it from the dropdown
3. You'll see the EC2 Dashboard - this is your control panel for managing virtual servers
4. Click the orange "Launch Instance" button

### Step 3: Configure Your Virtual Server

Now we'll specify exactly what kind of computer we want AWS to create for us.

#### Name Your Instance
Give it a descriptive name like "Omeka-Production-Server" or "RosenwaldArchive-Main". This name is just for your reference in the AWS console - it won't be visible to users of your site.

#### Choose Your Operating System (AMI)
AMI stands for "Amazon Machine Image" - it's a template that defines the operating system and initial software. Select **Ubuntu Server 22.04 LTS (HVM), SSD Volume Type**.

Why Ubuntu 22.04 LTS?
- **Ubuntu** is a user-friendly version of Linux with excellent documentation and community support
- **22.04** is the version number
- **LTS** means "Long Term Support" - it will receive security updates for 5 years
- **SSD** means your server uses fast solid-state drives for better performance

#### Select Instance Type
Choose **t3.medium** at minimum. This determines your server's computing power.

Understanding instance types:
- **t3.micro** (free tier): 1 CPU, 1GB RAM - too small for production Omeka-S
- **t3.small**: 2 CPU, 2GB RAM - bare minimum, may struggle with large imports
- **t3.medium**: 2 CPU, 4GB RAM - recommended minimum for reliable performance
- **t3.large**: 2 CPU, 8GB RAM - better for larger collections or multiple concurrent users

The number after 't3' indicates the size. You can change this later if needed, but it requires stopping your server briefly.

### Step 4: Create Your Security Key

This is crucial - the key pair is your secure way to access the server. Think of it like the key to your house.

#### Creating the Key Pair
1. Under "Key pair (login)" click "Create new key pair"
2. Give it a name without spaces (like "omeka-server-key")
3. Keep "RSA" selected (it's an encryption type)
4. Keep ".pem" selected (the file format)
5. Click "Create key pair"

**CRITICAL**: A file will download to your computer. This is your private key - the only way to access your server. 
- Save it somewhere safe immediately
- Make a backup copy
- Never share it with anyone
- If you lose it, you lose access to your server

### Step 5: Configure Network Security

Security Groups act like a firewall, controlling what traffic can reach your server. We need to allow three types of connections:

1. **SSH (Port 22)**: Allows you to connect via terminal to manage the server
2. **HTTP (Port 80)**: Allows regular web traffic to reach your Omeka site
3. **HTTPS (Port 443)**: Allows secure/encrypted web traffic

Under "Network Settings":
- Select "Create security group"
- Check "Allow SSH traffic from" and select "My IP" for better security
- Check "Allow HTTP traffic from the internet"
- Check "Allow HTTPS traffic from the internet"

### Step 6: Configure Storage

Under "Configure storage", you'll see "8 GiB gp3" as the default. For production use with file uploads:
- Change this to at least **30 GiB** 
- Keep "gp3" (it's a fast SSD type)

Storage considerations:
- System and software: ~5GB
- Omeka-S application: ~1GB
- Database: Grows over time, usually 1-5GB
- Uploaded files: Depends on your collection size
- Temporary space for processing: 2x your largest upload

### Step 7: Launch Your Instance

Review all settings and click "Launch Instance". AWS will now create your virtual server. This takes about 1-2 minutes.

---

# PART 2: Connecting to Your New Server

## Understanding SSH (Secure Shell)

SSH is a secure way to connect to your server's command line interface. Think of it like remote desktop, but text-only. You'll type commands on your computer, but they execute on the server.

### Step 1: Prepare Your Local Computer

First, we need to properly store and secure your key file.

#### On Mac or Linux:
Open Terminal (on Mac, find it in Applications > Utilities)

```bash
# Check if .ssh directory exists in your home folder
ls -la ~/

# If you don't see .ssh in the list, create it:
mkdir ~/.ssh

# Move your downloaded key to this secure location
mv ~/Downloads/omeka-server-key.pem ~/.ssh/

# Set secure permissions (readable only by you)
chmod 400 ~/.ssh/omeka-server-key.pem
```

Why these permissions? The chmod 400 command means:
- 4 = read permission for the owner (you)
- 0 = no permissions for your group
- 0 = no permissions for others
SSH requires these strict permissions for security.

#### On Windows:
Use PowerShell or install Windows Subsystem for Linux (WSL) for the best experience.

### Step 2: Find Your Server's Address

1. Go back to the EC2 Dashboard in AWS
2. Click on "Instances (running)"
3. Click on your instance
4. Find "Public IPv4 DNS" - it looks like: `ec2-XX-XXX-XXX-XX.compute-1.amazonaws.com`
5. Copy this address

### Step 3: Connect to Your Server

```bash
# Navigate to your .ssh directory
cd ~/.ssh

# Connect using your key and server address
ssh -i omeka-server-key.pem ubuntu@ec2-XX-XXX-XXX-XX.compute-1.amazonaws.com
```

When prompted "Are you sure you want to continue connecting?", type `yes`

You'll see a welcome message like:
```
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-1028-aws x86_64)
```

Congratulations! You're now connected to your server. Every command you type now runs on the AWS server, not your local computer.

---

# PART 3: Installing the LAMP Stack

## Understanding LAMP

LAMP is an acronym for the four software components that create a complete web application environment:
- **Linux**: The operating system (Ubuntu in our case)
- **Apache**: The web server that delivers pages to browsers
- **MySQL**: The database that stores all your data
- **PHP**: The programming language that runs Omeka-S

Think of it like building a house:
- Linux is the foundation
- Apache is the front door and windows (how people access it)
- MySQL is the storage rooms and filing cabinets
- PHP is the butler who retrieves things and serves them to visitors

### Step 1: Update Your Server

Before installing new software, we need to update the server's package lists and upgrade existing software to the latest versions. This ensures compatibility and security.

```bash
# Update the package lists
sudo apt-get update
```

This command:
- `sudo` = "super user do" - runs the command with administrator privileges
- `apt-get` = Ubuntu's package manager (like an app store)
- `update` = downloads the latest list of available software

You'll see many lines of text as it contacts various servers and downloads package information.

```bash
# Upgrade installed packages
sudo apt-get upgrade -y
```

This command:
- `upgrade` = installs newer versions of your existing software
- `-y` = automatically answers "yes" to prompts

This may take 5-10 minutes. You might see a purple screen asking about restarting services - use the spacebar to select all options, then press Enter.

### Step 2: Install Apache Web Server

Apache is one of the world's most popular web servers. It receives requests from browsers and serves back web pages.

```bash
# Install Apache
sudo apt-get install -y apache2
```

This downloads and installs Apache and all its dependencies (supporting software it needs).

```bash
# Enable the rewrite module
sudo a2enmod rewrite
```

The rewrite module allows Apache to create "clean URLs" - instead of `site.com/index.php?page=items`, you get `site.com/items`. This is essential for Omeka-S.

```bash
# Restart Apache to load the new module
sudo systemctl restart apache2
```

`systemctl` is the system control command. We're telling it to restart the Apache service.

#### Test Apache Is Working

1. In your web browser, go to: `http://[your-public-dns]`
2. You should see the "Apache2 Ubuntu Default Page"
3. This confirms your web server is running and accessible from the internet

If you don't see the page:
- Check that you typed http:// not https://
- Verify port 80 is open in your security group
- Ensure you're using the public DNS, not the private IP

### Step 3: Install MySQL Database Server

MySQL stores all your Omeka data in organized tables - think of it like a super-powered Excel spreadsheet system that can handle millions of records and complex relationships.

```bash
# Install MySQL Server
sudo apt-get install -y mysql-server
```

This installs MySQL version 8.0, which includes improved security features and better performance than earlier versions.

#### Access MySQL and Create a Database

```bash
# Log into MySQL as the root (administrator) user
sudo mysql -u root -p
```

When prompted for a password, just press Enter. With Ubuntu's default setup, using `sudo` authenticates you automatically.

You'll see the MySQL prompt change to `mysql>`. You're now inside the database system.

```sql
-- Create a new database for Omeka
CREATE DATABASE omeka_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Let's break this down:
- `CREATE DATABASE omeka_db` = makes a new database named "omeka_db"
- `CHARACTER SET utf8mb4` = ensures the database can store any character from any language, including emojis
- `COLLATE utf8mb4_unicode_ci` = defines how text is sorted and compared (ci = case-insensitive)

```sql
-- Create a user account for Omeka to use
CREATE USER 'omeka_user'@'localhost' IDENTIFIED BY 'ChooseAStrongPassword123!';
```

Important notes:
- Replace `ChooseAStrongPassword123!` with your own secure password
- Write this password down - you'll need it later
- The password should include uppercase, lowercase, numbers, and symbols
- `@'localhost'` means this user can only connect from the same server (security feature)

```sql
-- Give the user permission to do anything within the omeka database
GRANT ALL PRIVILEGES ON omeka_db.* TO 'omeka_user'@'localhost';
```

This gives your omeka_user account full control over the omeka_db database, but no access to other databases on the server.

```sql
-- Apply the permission changes
FLUSH PRIVILEGES;

-- Exit MySQL
EXIT;
```

### Step 4: Install PHP and Extensions

PHP is the programming language that Omeka-S is written in. We need PHP itself plus several extensions that add specific capabilities.

```bash
# Install PHP and all required extensions at once
sudo apt-get install -y php php-mysql php-xml php-mbstring php-zip php-gd php-curl php-imagick php-intl php-fpm
```

Each extension serves a specific purpose:
- `php` = The core PHP language (version 8.1 on Ubuntu 22.04)
- `php-mysql` = Allows PHP to communicate with MySQL database
- `php-xml` = Processes XML data (used for metadata exchange)
- `php-mbstring` = Handles multi-byte strings (for international characters like Japanese, Arabic)
- `php-zip` = Creates and extracts ZIP files (essential for batch imports)
- `php-gd` = Graphics library for basic image manipulation
- `php-imagick` = ImageMagick integration for advanced image processing (better than GD)
- `php-curl` = Fetches data from other websites (for harvesting metadata)
- `php-intl` = Internationalization support (for multiple languages)
- `php-fpm` = FastCGI Process Manager (more efficient way to run PHP)

The installation will take a few minutes and display lots of text. This is normal.

#### Verify PHP Installation

```bash
# Check PHP version
php -v
```

You should see something like "PHP 8.1.x". The exact minor version doesn't matter.

---

# PART 4: Installing Omeka-S

Now that our server foundation is ready, we can install Omeka-S itself. Omeka-S is a web application specifically designed for creating digital collections and exhibitions.

### Step 1: Download Omeka-S

We'll download the latest stable version directly from the official GitHub repository.

```bash
# Change to the temporary directory
cd /tmp
```

The `/tmp` directory is a safe place to download files temporarily. It's automatically cleaned up by the system periodically.

```bash
# Download Omeka-S (version 4.0.1 as of this writing)
wget https://github.com/omeka/omeka-s/releases/download/v4.0.1/omeka-s-4.0.1.zip
```

`wget` is a command that downloads files from the internet. You'll see a progress bar as it downloads (about 30MB).

To check for newer versions, visit: https://github.com/omeka/omeka-s/releases

```bash
# Extract the ZIP file
unzip omeka-s-4.0.1.zip
```

This creates a folder called "omeka-s" containing all the application files.

```bash
# Move Omeka to the web directory
sudo mv omeka-s /var/www/html/
```

`/var/www/html/` is the standard location where Apache looks for web files. Moving Omeka here makes it accessible via the web.

### Step 2: Set File Permissions

Omeka needs permission to write files in certain directories (for uploads, thumbnails, etc.). We need to give the web server user (www-data) ownership of these directories.

```bash
# Give Apache ownership of the files directory
sudo chown -R www-data:www-data /var/www/html/omeka-s/files
```

Breaking this down:
- `chown` = "change ownership"
- `-R` = recursive (applies to all subdirectories)
- `www-data:www-data` = user and group that Apache runs as
- The files directory stores all uploads and generated thumbnails

```bash
# Set directory permissions
sudo chmod -R 755 /var/www/html/omeka-s/files
```

Permission 755 means:
- Owner (www-data): read, write, execute (7)
- Group: read and execute (5)
- Others: read and execute (5)

This allows Apache to write files while preventing unauthorized modifications.

### Step 3: Configure Database Connection

For more detailed instructions on databases see [Database Instructions →](./database-ini.md.md)

Omeka needs to know how to connect to the MySQL database we created earlier.

```bash
# Edit the database configuration file
sudo nano /var/www/html/omeka-s/config/database.ini
```

`nano` is a simple text editor. You'll see a mostly empty file with some placeholder text.

Use arrow keys to navigate and edit the file to look like this:

```ini
user     = "omeka_user"
password = "ChooseAStrongPassword123!"  ; Use the password you created earlier
dbname   = "omeka_db"
host     = "localhost"
```

Important:
- Make sure you use the exact password you set when creating the MySQL user
- Don't include the square brackets shown in the template
- The semicolon (;) marks a comment - text after it is ignored

To save and exit nano:
1. Press `Ctrl+X`
2. Press `Y` to confirm saving
3. Press `Enter` to keep the filename

### Step 4: Configure Apache for Omeka-S

Apache needs some additional configuration to properly serve Omeka-S.

```bash
# Edit Apache's main configuration
sudo nano /etc/apache2/apache2.conf
```

Scroll down (using Page Down or arrow keys) until you find this section:
```apache
<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

Change `AllowOverride None` to `AllowOverride All`

This change allows Omeka's .htaccess files to work. These files contain important configuration for:
- Creating clean URLs
- Setting security headers
- Controlling file access
- Improving performance

Save the file (Ctrl+X, Y, Enter).

```bash
# Restart Apache to apply changes
sudo systemctl restart apache2
```

### Step 5: Complete Omeka Installation

1. Open your web browser
2. Navigate to: `http://[your-public-dns]/omeka-s`
3. You should see the Omeka-S installation page

Fill in the installation form:
- **Display Name**: Your name or institution
- **Email**: Admin email address
- **Username**: Choose an admin username (not "admin" for security)
- **Password**: Choose a strong password (different from database password)

Click "Install" and wait about 30 seconds. You'll see "Installation successful!"

---

# PART 5: Optimizing PHP for Large File Imports

Standard PHP settings limit file uploads to 8MB and give scripts only 30 seconds to run. This isn't enough for archival work where you might upload hundreds of high-resolution images at once. We'll reconfigure PHP to handle files up to 1GB.

## Understanding PHP-FPM

PHP-FPM (FastCGI Process Manager) is an advanced way to run PHP that's faster and more flexible than the standard method. It allows us to create custom configurations for different applications.

Think of it like having a dedicated lane on a highway just for Omeka traffic, with its own speed limits and rules.

### Step 1: Create a Custom PHP Pool for Omeka

We'll create a dedicated PHP configuration just for Omeka with much higher limits than default.

```bash
# Create a new PHP-FPM pool configuration
sudo nano /etc/php/8.1/fpm/pool.d/omeka.conf
```

Add this entire configuration (copy and paste carefully):

```

; CRITICAL: File upload and memory settings for ZipImport
; These settings allow handling files up to 1GB
php_admin_value[upload_max_filesize] = 1000M
php_admin_value[post_max_size] = 1000M
php_admin_value[memory_limit] = 1024M
php_admin_value[max_file_uploads] = 100
php_admin_value[max_execution_time] = 300
php_admin_value[max_input_time] = 300
```

Let's understand what each setting does:

**Memory and Upload Settings** (the most important part):
- `upload_max_filesize = 1000M` - Single files up to 1GB can be uploaded
- `post_max_size = 1000M` - Total POST data (all files combined) up to 1GB
- `memory_limit = 1024M` - PHP can use up to 1GB of RAM for processing
- `max_file_uploads = 100` - Can upload 100 files simultaneously
- `max_execution_time = 300` - Scripts can run for 5 minutes (for processing large imports)
- `max_input_time = 300` - Allow 5 minutes to receive upload data

**Process Manager Settings**:
- Controls how many PHP processes run simultaneously
- More processes = handle more concurrent users
- These settings are good for a medium-traffic site

**OPcache Settings**:
- OPcache stores compiled PHP code in memory
- Makes Omeka run 2-3x faster
- Reduces server load

Save the file (Ctrl+X, Y, Enter).

### Step 2: Create Log Directory

```bash
# Create directory for PHP-FPM logs
sudo mkdir -p /var/log/php-fpm
sudo chown www-data:www-data /var/log/php-fpm
```

This gives us a dedicated place to check for PHP errors if something goes wrong.

### Step 3: Configure Apache to Use PHP-FPM

Now we need to tell Apache to use our custom PHP configuration for Omeka.

```bash
# Enable required Apache modules
sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php8.1-fpm

# Edit the default site configuration
sudo nano /etc/apache2/sites-available/000-default.conf
```

Inside the `<VirtualHost *:80>` block (before `</VirtualHost>`), add:

```apache
    # Use our custom PHP-FPM pool for all PHP files
    <FilesMatch \.php$>
        SetHandler "proxy:unix:/run/php/php8.1-fpm-omeka.sock|fcgi://localhost"
    </FilesMatch>
    
    # Increase timeout for large file uploads
    Timeout 600
    ProxyTimeout 600
```

These timeout settings give uploads 10 minutes to complete, which is important for slow connections or very large files.

### Step 4: Restart Services

```bash
# Restart PHP-FPM to load the new configuration
sudo systemctl restart php8.1-fpm

# Restart Apache
sudo systemctl restart apache2

# Verify PHP-FPM is running
sudo systemctl status php8.1-fpm
```

You should see "active (running)" in green.

### Step 5: Verify Settings

Create a test file to confirm our settings are applied:

```bash
# Create a PHP info file
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/omeka-s/info.php
```

1. Visit: `http://[your-public-dns]/omeka-s/info.php`
2. Search for "upload_max_filesize" - should show "1000M"
3. Search for "memory_limit" - should show "1024M"
4. **Important**: Delete this file after checking (security risk):

```bash
sudo rm /var/www/html/omeka-s/info.php
```

---

# PART 6: Configuring ImageMagick for PDF Processing

ImageMagick is a powerful tool for converting and manipulating images. Omeka uses it to create thumbnails from uploaded files, including PDFs. However, PDF processing can be a security risk if not configured properly.

## Understanding the Security Concern

PDFs can contain embedded code that might be executed during processing. We need to configure ImageMagick to handle PDFs safely while still allowing thumbnail generation.

### Step 1: Install ImageMagick

```bash
# Install ImageMagick and PHP extension
sudo apt-get install -y imagemagick php-imagick

# Restart Apache to load the PHP extension
sudo systemctl restart apache2
```

### Step 2: Configure Security Policy

ImageMagick has a security policy file that controls what operations are allowed. By default, PDF processing is completely disabled. We'll modify this to allow controlled PDF access.

```bash
# First, make a backup of the original policy
sudo cp /etc/ImageMagick-6/policy.xml /etc/ImageMagick-6/policy.xml.backup

# Edit the policy file
sudo nano /etc/ImageMagick-6/policy.xml
```

Scroll down to find this line (near the bottom):
```xml
<policy domain="coder" rights="none" pattern="PDF" />
```

This line completely blocks PDF processing. Change it to:
```xml
<policy domain="coder" rights="read|write" pattern="PDF" />
```

Now find the resource limits section and update or add these lines:
```xml
<policy domain="resource" name="memory" value="512MiB"/>
<policy domain="resource" name="map" value="1GiB"/>
<policy domain="resource" name="width" value="16KP"/>
<policy domain="resource" name="height" value="16KP"/>
<policy domain="resource" name="area" value="128MB"/>
<policy domain="resource" name="disk" value="2GiB"/>
```

These settings:
- Limit memory usage to prevent runaway processes
- Set maximum image dimensions (16K pixels)
- Limit disk usage for temporary files
- Prevent processing of extremely large PDFs that could crash the server

Save the file (Ctrl+X, Y, Enter).

### Step 3: Test PDF Processing

Let's verify that ImageMagick can now process PDFs safely:

```bash
# Create a test directory
mkdir /tmp/imagetest
cd /tmp/imagetest

# Download a sample PDF
wget https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf

# Try to create a thumbnail
convert dummy.pdf[0] -thumbnail 200x200 test-thumb.jpg

# Check if thumbnail was created
ls -la test-thumb.jpg
```

If you see "test-thumb.jpg" in the listing, PDF processing is working!

```bash
# Clean up test files
cd /
rm -rf /tmp/imagetest
```

### Step 4: Configure Omeka to Use ImageMagick

Omeka should automatically detect and use ImageMagick. To verify:

1. Log into your Omeka admin panel
2. Go to Settings
3. Look for "Thumbnailer" - should show "Imagick"
4. If it shows "GD", click the dropdown and select "Imagick"
5. Save settings

---

# PART 7: Validation and Testing

Before declaring your server ready for production, let's run through a comprehensive validation checklist.

## System Validation Tests

### Test 1: Web Server Accessibility
```bash
# Check Apache status
sudo systemctl status apache2
```
Expected: "active (running)" in green

Browser test: Navigate to `http://[your-public-dns]/omeka-s`
Expected: Omeka-S login page

### Test 2: Database Connectivity
```bash
# Test database connection with Omeka credentials
mysql -u omeka_user -p omeka_db -e "SELECT VERSION();"
```
Enter the password when prompted.
Expected: Shows MySQL version number

### Test 3: PHP Configuration
```bash
# Check PHP version and modules
php -v
php -m | grep -E "imagick|zip|mysql|mbstring"
```
Expected: PHP 8.1.x and all required modules listed

### Test 4: File Upload Test

1. Log into Omeka-S admin panel
2. Go to Items → Add new item
3. Try uploading a file larger than 8MB (but less than 1GB)
4. Expected: Upload succeeds without error

### Test 5: Large ZIP Import Test

1. Create a ZIP file with multiple images (50-100MB)
2. Use the CSV Import or Bulk Import module
3. Upload the ZIP file
4. Expected: Import completes without timeout errors

## Performance Validation

### Check Memory Usage
```bash
# View current memory usage
free -h
```
Expected: At least 1GB free memory available

### Check Disk Space
```bash
# View disk usage
df -h
```
Expected: At least 20% free space on root partition

### Monitor Apache Connections
```bash
# See current Apache connections
sudo apache2ctl status
```

## Security Checklist

Run through these security verifications:

```bash
# Check that only necessary ports are open
sudo ufw status

# Verify Apache is hiding version info
curl -I http://localhost | grep Server

# Check PHP error display is off
grep display_errors /etc/php/8.1/fpm/pool.d/omeka.conf

# Ensure file permissions are correct
ls -la /var/www/html/omeka-s/files
```

---

# PART 8: Troubleshooting Common Issues

## Issue: Cannot Connect via SSH

**Symptom**: "Permission denied" or "Connection timed out"

**Solutions**:
1. Verify your key file has correct permissions: `chmod 400 ~/.ssh/your-key.pem`
2. Check you're using the right username: `ubuntu@` not `root@` or `admin@`
3. Confirm the instance is running in AWS console
4. Verify security group allows SSH from your IP

## Issue: Apache Default Page Shows Instead of Omeka

**Symptom**: You see "Apache2 Ubuntu Default Page" at your domain

**Solutions**:
1. Make sure you're going to `/omeka-s` not just the domain
2. Check Apache configuration: `sudo apache2ctl configtest`
3. Ensure Omeka files are in correct location: `ls /var/www/html/omeka-s`

## Issue: File Uploads Fail

**Symptom**: "File too large" or upload stops at 8MB

**Solutions**:
1. Verify PHP-FPM settings: `grep upload_max /etc/php/8.1/fpm/pool.d/omeka.conf`
2. Restart PHP-FPM: `sudo systemctl restart php8.1-fpm`
3. Check Apache is using custom PHP pool
4. Verify with phpinfo() that settings are applied

## Issue: Database Connection Error

**Symptom**: "Database connection error" during installation

**Solutions**:
1. Verify MySQL is running: `sudo systemctl status mysql`
2. Test credentials: `mysql -u omeka_user -p omeka_db`
3. Check database.ini has correct information
4. Ensure no extra spaces in database.ini file

## Issue: Thumbnails Not Generating

**Symptom**: Uploaded images show generic icons instead of thumbnails

**Solutions**:
1. Check ImageMagick is installed: `convert -version`
2. Verify policy.xml allows image processing
3. Check Omeka is set to use Imagick in settings
4. Ensure files directory is writable: `sudo chown -R www-data:www-data /var/www/html/omeka-s/files`

---

# PART 9: Maintenance and Best Practices

## Regular Maintenance Tasks

### Weekly Tasks
- Check disk space: `df -h`
- Review error logs: `sudo tail -100 /var/log/apache2/error.log`
- Verify backups are running

### Monthly Tasks
- Install security updates: `sudo apt-get update && sudo apt-get upgrade`
- Review user accounts in Omeka
- Check SSL certificate expiration (if using HTTPS)

### Quarterly Tasks
- Review and optimize database: `mysqlcheck -u root -p --optimize omeka_db`
- Clean old log files
- Review security group settings in AWS

## Backup Strategy

### Database Backups
```bash
# Create a database backup
mysqldump -u omeka_user -p omeka_db > omeka_backup_$(date +%Y%m%d).sql
```

### File Backups
```bash
# Backup uploaded files
tar -czf omeka_files_$(date +%Y%m%d).tar.gz /var/www/html/omeka-s/files
```

Consider using AWS S3 for automated backups - see the backup-restore.md guide for details.

## Performance Optimization Tips

1. **Enable browser caching**: Add cache headers in Apache configuration
2. **Use a CDN**: CloudFlare can speed up global access
3. **Optimize images**: Use image compression before uploading
4. **Monitor usage**: Use AWS CloudWatch to track server performance
5. **Scale when needed**: Upgrade instance type if consistently hitting limits

## Security Best Practices

1. **Regular updates**: Enable unattended-upgrades for security patches
2. **Strong passwords**: Use different passwords for SSH, MySQL, and Omeka
3. **Limit SSH access**: Restrict to specific IP addresses when possible
4. **Enable HTTPS**: Use Let's Encrypt for free SSL certificates
5. **Regular backups**: Test restore procedures quarterly
6. **Monitor logs**: Set up log analysis to detect unusual activity

---

# Conclusion

Congratulations! You now have a production-ready Omeka-S installation capable of handling large archival collections. Your server is configured with:

✅ Ubuntu 22.04 LTS with security updates 

✅ Apache web server with proper modules 

✅ MySQL database with dedicated Omeka user 

✅ PHP 8.1 with 1GB file upload capability 

✅ ImageMagick for PDF and image processing 

✅ Optimized settings for the ZipImport module 

## Next Steps

1. **Install Omeka modules**: Add CSV Import, Bulk Edit, and other needed modules
2. **Create your first collection**: Start adding items to your archive
3. **Document your customizations**: Keep notes on any changes you make

## Getting Help

- **Omeka Forums**: https://forum.omeka.org/
- **AWS Support**: Available through your AWS console
- **This Documentation**: Check related guides in this repository
- **Ubuntu Community**: https://askubuntu.com/

Remember to document any issues you encounter and their solutions - this helps the entire community!

---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Oct 27 2025 | Sai Kiran Boppana | Initial comprehensive guide |

---

*This document is part of the Rosenwald Fund Collection, supporting digital preservation at HBCUs and cultural institutions.*
