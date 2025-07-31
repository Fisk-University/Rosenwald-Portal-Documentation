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

#### You can also use SSH to clone a module directly from that module's git repository. Do this only if you are comfortable with git, GitHub, and working with SSH.

## Installation
- Add the zip to the Omeka-S modules folder and unzip it.
- Rename the unzipped folder to "ZipImport" if it's not already named as such.
- Open a terminal, navigate to the "ZipImport" folder, and run `composer install`.
** Sudo or appropriate permissions may be necessary **

## Update php.ini
- Find your php.ini file with the `php -r "phpinfo();" | grep php.ini` command, or with an info.php page using `<?php phpinfo(); ?>`
- Adjust the following fields:
```
upload_max_filesize = 1000M
max_file_uploads = 100
post_max_size = 1000M
```
- Restart your server with `service apache2 restart` or the relevant command.
- Restart php with `systemctl restart php` or the relevant command.


## Check PHP Limits 
- In the Omeka-S Dashboard, click "System Information" in the lower right to ensure PHP limits are updated.

# Installing the Module in Omeka S
- Log into the Omeka S Admin Dashboard.

- Navigate to the Modules from the left-hand menu.

- Find ZipImport in the list and click Install.

If the module has configuration options, you will be taken to its setup page otherwise, it will install and activate immediately with a success message.

