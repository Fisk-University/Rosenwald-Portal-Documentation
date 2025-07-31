# ZipImport Module Installation Tutorial

This tutorial walks you through how to install and activate the **ZipImport** module in your Omeka S installation.

---

## Requirements

- Omeka S version **4.0.0 or higher**
- Access to your server’s **terminal or SSH**
- Access to your Omeka S `/modules` directory

---

## Download the Module

1. Download the latest release of the **ZipImport** module from its source (e.g., GitHub).
2. Save the ZIP file to your local machine.

---

##  Adding a module to the site

### Option 1: Using FTP

1. Download the zipped module to your computer.
2. Open your FTP client, log on to the server that houses the Omeka S install, and navigate to the /modules folder (this should be located in the main folder of the install).
3. Upload the zipped module to the /modules folder .
4. Unzip the module.

### Option 2: Using SSH connection

You can also use SSH to clone a module directly from that module's git repository. Do this only if you are comfortable with git, GitHub, and working with SSH.

Install Composer Dependencies
Open a terminal and navigate into the unzipped ZipImport folder:

bash
Copy code
cd /path/to/omeka-s/modules/ZipImport
composer install
⚠️ You may need to use sudo depending on your permissions.

⚙️ Update PHP Settings
Locate your php.ini file by running:

bash
Copy code
php -r "phpinfo();" | grep php.ini
Open the file and update the following settings:

ini
Copy code
upload_max_filesize = 1000M
max_file_uploads = 100
post_max_size = 1000M
Save and restart your server:

bash
Copy code
sudo service apache2 restart  # For Apache
sudo systemctl restart php8.1-fpm  # Replace with your PHP version
✅ Verify PHP Limits in Omeka S
Log into the Omeka S Admin Dashboard.

Scroll to the bottom-right corner and click System Information.

Confirm that the PHP settings reflect your changes.

# Installing the Module in Omeka S
- Log into the Omeka S Admin Dashboard.

- Navigate to the Modules from the left-hand menu.

- Find ZipImport in the list and click Install.

If the module has configuration options, you will be taken to its setup page otherwise, it will install and activate immediately with a success message.

![ZipImport Install](/Users/Kosi/Documents/Screenshot 2025-07-31 at 12.57.04 AM.png)