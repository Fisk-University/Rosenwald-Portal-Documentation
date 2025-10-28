# DATABASE-001: Omeka-S Database Configuration and Environment Variables

## Purpose
This guide provides comprehensive instructions for configuring Omeka-S database connections securely across development, staging, and production environments. It emphasizes security best practices, troubleshooting common connection issues, and managing credentials without exposing them in version control.

## Applies To
- All Omeka-S installations version 3.x and above
- Multi-environment deployments (dev/staging/production)
- Teams using version control (Git) for configuration management
- Installations requiring enhanced security compliance

---

## 1. Prerequisites

| Requirement | Description |
|------------|-------------|
| **MySQL Database** | MySQL 5.7+ or MariaDB 10.3+ installed and running |
| **Database Created** | Empty database with UTF8MB4 character set |
| **User Credentials** | Database username and password with appropriate privileges |
| **File Access** | Write permissions to Omeka's config directory |
| **PHP Version** | PHP 7.4+ with PDO MySQL extension |

---

## 2. Understanding database.ini

### 2.1 File Location and Purpose

The `database.ini` file is located at `/var/www/html/omeka-s/config/database.ini` and contains the credentials Omeka uses to connect to your MySQL database. This file is critical - without proper configuration, Omeka cannot function.

**Important Security Note:** This file contains sensitive credentials and should never be committed to public repositories.

### 2.2 Basic Configuration Structure

```ini
; Database Configuration File
; Lines starting with semicolon are comments

user     = "omeka_user"        ; MySQL username
password = "SecurePassword123"  ; MySQL password  
dbname   = "omeka_db"          ; Database name
host     = "localhost"         ; Database server location
port     = "3306"              ; MySQL port (optional, defaults to 3306)
charset  = "utf8mb4"           ; Character encoding (optional, defaults to utf8mb4)
```

### 2.3 Configuration Parameters Explained

- **user**: The MySQL username created specifically for Omeka
- **password**: The password for the MySQL user (should be strong and unique)
- **dbname**: The name of the database you created for Omeka
- **host**: Where the database server is located
  - `localhost` or `127.0.0.1` for same server
  - RDS endpoint for AWS RDS databases
  - IP address or hostname for remote databases
- **port**: MySQL connection port (usually 3306, can be omitted if standard)
- **charset**: Character encoding (utf8mb4 recommended for full Unicode support)

---

## 3. Basic Configuration Methods

### 3.1 Direct Configuration (Development Only)

For development environments, you can directly edit the database.ini file:

```bash
# Edit the configuration file
sudo nano /var/www/html/omeka-s/config/database.ini
```

Add your credentials:
```ini
user     = "omeka_user"
password = "DevPassword123"
dbname   = "omeka_dev"
host     = "localhost"
```

**Warning**: Never use this method for production servers where the file might be exposed or backed up with credentials.

### 3.2 Using Environment Variables (Recommended)

For production environments, use environment variables to keep credentials out of files:

#### Step 1: Set Environment Variables

Add to `/etc/environment` or create a systemd environment file:

```bash
# Create environment file for Apache
sudo nano /etc/apache2/envvars
```

Add at the end:
```bash
export OMEKA_DB_USER="omeka_user"
export OMEKA_DB_PASSWORD="SecurePassword123"
export OMEKA_DB_NAME="omeka_db"
export OMEKA_DB_HOST="localhost"
```

#### Step 2: Configure PHP to Read Environment Variables

Create a PHP configuration loader:

```bash
# Create a secure configuration loader
sudo nano /var/www/html/omeka-s/config/database.php
```

```php
<?php
// Load environment variables for database configuration
return [
    'user'     => getenv('OMEKA_DB_USER') ?: 'omeka_user',
    'password' => getenv('OMEKA_DB_PASSWORD') ?: '',
    'dbname'   => getenv('OMEKA_DB_NAME') ?: 'omeka_db',
    'host'     => getenv('OMEKA_DB_HOST') ?: 'localhost',
    'port'     => getenv('OMEKA_DB_PORT') ?: '3306',
    'charset'  => getenv('OMEKA_DB_CHARSET') ?: 'utf8mb4'
];
```

#### Step 3: Modify database.ini to Use PHP Configuration

```ini
; This file now references environment variables
; Actual credentials are stored in environment
user     = "${OMEKA_DB_USER}"
password = "${OMEKA_DB_PASSWORD}"
dbname   = "${OMEKA_DB_NAME}"
host     = "${OMEKA_DB_HOST}"
```

---

## 4. Multi-Environment Configuration

### 4.1 Development Environment

```ini
; database.ini.development
user     = "omeka_dev"
password = "DevPassword123"
dbname   = "omeka_development"
host     = "localhost"
; Debug mode enabled for development
debug    = "1"
```

### 4.2 Staging Environment

```ini
; database.ini.staging
user     = "omeka_stage"
password = "StagePassword456"
dbname   = "omeka_staging"
host     = "staging-db.example.com"
port     = "3306"
```

### 4.3 Production Environment

```ini
; database.ini.production
; Uses environment variables for security
user     = "${OMEKA_DB_USER}"
password = "${OMEKA_DB_PASSWORD}"
dbname   = "${OMEKA_DB_NAME}"
host     = "${OMEKA_DB_HOST}"
; Connection pooling for performance
persistent = "1"
```

### 4.4 Environment Switching Script

Create a script to switch between environments:

```bash
#!/bin/bash
# switch-environment.sh

ENVIRONMENT=$1
CONFIG_DIR="/var/www/html/omeka-s/config"

case $ENVIRONMENT in
    dev|development)
        cp $CONFIG_DIR/database.ini.development $CONFIG_DIR/database.ini
        echo "Switched to development environment"
        ;;
    staging)
        cp $CONFIG_DIR/database.ini.staging $CONFIG_DIR/database.ini
        echo "Switched to staging environment"
        ;;
    production|prod)
        cp $CONFIG_DIR/database.ini.production $CONFIG_DIR/database.ini
        echo "Switched to production environment"
        ;;
    *)
        echo "Usage: $0 {dev|staging|production}"
        exit 1
        ;;
esac

# Restart Apache to apply changes
sudo systemctl restart apache2
```

---

## 5. AWS RDS Configuration

### 5.1 Using Amazon RDS for Production

When using AWS RDS for your database, the configuration differs:

```ini
; Configuration for AWS RDS
user     = "omeka_rds_user"
password = "RDSPassword789"
dbname   = "omeka_production"
host     = "omeka-db.c9xyz123.us-east-1.rds.amazonaws.com"
port     = "3306"

; RDS-specific optimizations
charset  = "utf8mb4"
persistent = "0"  ; Don't use persistent connections with RDS
```

### 5.2 RDS with Read Replicas

For high-traffic sites using RDS read replicas:

```php
<?php
// config/database-replicas.php
return [
    'primary' => [
        'host' => 'omeka-db-primary.region.rds.amazonaws.com',
        'user' => getenv('RDS_PRIMARY_USER'),
        'password' => getenv('RDS_PRIMARY_PASSWORD'),
    ],
    'replicas' => [
        [
            'host' => 'omeka-db-read-1.region.rds.amazonaws.com',
            'user' => getenv('RDS_REPLICA_USER'),
            'password' => getenv('RDS_REPLICA_PASSWORD'),
        ],
        [
            'host' => 'omeka-db-read-2.region.rds.amazonaws.com',
            'user' => getenv('RDS_REPLICA_USER'),
            'password' => getenv('RDS_REPLICA_PASSWORD'),
        ]
    ]
];
```

---

## 6. Security Best Practices

### 6.1 File Permissions

```bash
# Set restrictive permissions on database.ini
sudo chown www-data:www-data /var/www/html/omeka-s/config/database.ini
sudo chmod 400 /var/www/html/omeka-s/config/database.ini
```

### 6.2 Using .env Files

Create a `.env` file for local development:

```bash
# .env file (add to .gitignore)
OMEKA_DB_USER=omeka_user
OMEKA_DB_PASSWORD=SecurePassword123
OMEKA_DB_NAME=omeka_db
OMEKA_DB_HOST=localhost
```

Load with PHP dotenv library:

```php
<?php
// In application bootstrap
$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();

// Now use $_ENV['OMEKA_DB_USER'] etc.
```

### 6.3 Secrets Management

For production, use AWS Secrets Manager or similar:

```bash
# Retrieve password from AWS Secrets Manager
DB_PASSWORD=$(aws secretsmanager get-secret-value \
    --secret-id omeka/db/password \
    --query SecretString --output text)

# Export for application use
export OMEKA_DB_PASSWORD=$DB_PASSWORD
```

### 6.4 Git Security

Add to `.gitignore`:

```gitignore
# Never commit database credentials
config/database.ini
config/database.ini.*
.env
*.env
config/local.config.php
```

---

## 7. Troubleshooting Database Connections

### 7.1 Common Connection Errors

#### Error: "SQLSTATE[HY000] [1045] Access denied for user"

**Cause**: Incorrect username or password

**Solution**:
```bash
# Test credentials directly
mysql -u omeka_user -p omeka_db

# If fails, reset password in MySQL
sudo mysql
ALTER USER 'omeka_user'@'localhost' IDENTIFIED BY 'NewPassword';
FLUSH PRIVILEGES;
```

#### Error: "SQLSTATE[HY000] [2002] Connection refused"

**Cause**: MySQL not running or wrong host/port

**Solution**:
```bash
# Check MySQL status
sudo systemctl status mysql

# Start if not running
sudo systemctl start mysql

# Check port
sudo netstat -tlnp | grep 3306
```

#### Error: "SQLSTATE[HY000] [1049] Unknown database"

**Cause**: Database doesn't exist

**Solution**:
```bash
# Create database
sudo mysql -e "CREATE DATABASE omeka_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

### 7.2 Testing Database Connection

Create a test script:

```php
<?php
// test-db.php
$config = parse_ini_file('/var/www/html/omeka-s/config/database.ini');

try {
    $dsn = "mysql:host={$config['host']};dbname={$config['dbname']};charset=utf8mb4";
    $pdo = new PDO($dsn, $config['user'], $config['password']);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    
    echo "✓ Database connection successful!\n";
    
    // Test query
    $result = $pdo->query("SELECT VERSION()");
    $version = $result->fetchColumn();
    echo "✓ MySQL Version: $version\n";
    
} catch (PDOException $e) {
    echo "✗ Connection failed: " . $e->getMessage() . "\n";
    exit(1);
}
```

Run test:
```bash
php test-db.php
```

### 7.3 Debug Mode

Enable debug mode for detailed error messages (development only):

```ini
; Add to database.ini for debugging
debug = "1"
log_errors = "1"
error_log = "/var/log/omeka/db-errors.log"
```

### 7.4 Connection Monitoring

Monitor database connections:

```bash
# Check current connections
mysql -u root -p -e "SHOW PROCESSLIST;"

# Check max connections setting
mysql -u root -p -e "SHOW VARIABLES LIKE 'max_connections';"

# Monitor connection errors
tail -f /var/log/mysql/error.log
```

---

## 8. Performance Optimization

### 8.1 Connection Pooling

```ini
; Enable persistent connections (carefully)
persistent = "1"
max_persistent = "10"
max_connections = "20"
```

### 8.2 Query Caching

```ini
; Enable query cache (MySQL 5.7 and below)
query_cache = "1"
query_cache_size = "64M"
```

### 8.3 Connection Timeout Settings

```ini
; Timeout settings for long operations
connect_timeout = "10"
wait_timeout = "28800"
interactive_timeout = "28800"
```

---

## 9. Migration Between Environments

### 9.1 Export from Development

```bash
# Export database structure and data
mysqldump -u omeka_dev -p omeka_development > omeka_dev_backup.sql

# Export structure only
mysqldump -u omeka_dev -p --no-data omeka_development > omeka_structure.sql
```

### 9.2 Import to Production

```bash
# Import to production (careful!)
mysql -u omeka_prod -p omeka_production < omeka_structure.sql

# Import sample data if needed
mysql -u omeka_prod -p omeka_production < sample_data.sql
```

### 9.3 Configuration Sync

```bash
# Sync configuration files (exclude database.ini)
rsync -av --exclude='database.ini' \
    ./config/ production-server:/var/www/html/omeka-s/config/
```

---

## 10. Validation Checklist

Before deploying, verify:

- [ ] Database connection successful
- [ ] Correct character encoding (UTF8MB4)
- [ ] Environment variables properly set
- [ ] File permissions restrictive (400 or 600)
- [ ] No credentials in version control
- [ ] Backup of working configuration
- [ ] Test user can perform CRUD operations
- [ ] Connection pooling configured appropriately
- [ ] Error logging enabled and working
- [ ] Monitoring alerts configured
---

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | Oct 27 2025 | Sai Kiran Boppana| Initial database configuration guide |
