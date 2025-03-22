# UNIT3D Backup Restoration Tutorial

![REQUIRED](https://img.shields.io/badge/REQUIRED-UNIT3D%20v9.0.1%20PHP%208.4-red)

_Whether you've made an error and need to restore from a backup, or you're just looking to learn something new, this tutorial has got you covered._

This guide explains how to restore a UNIT3D backup on your server. It covers installing required tools, uncompressing the backup using your app key, copying files to their correct locations, fixing file permissions, and resetting caches with PHP Artisan.

<div style="border: 2px solid #e74c3c; background-color: #f9e6e6; padding: 10px; border-radius: 5px; margin: 15px 0;">
  <strong>🚨 READ:</strong> This tutorial was tested on <strong>UNIT3D v9.0.1</strong> running on <strong>PHP 8.4</strong>. Adjustments may be necessary for other versions.
</div>

## Built-In Backups

Built-in backups, located in `.../storage/backups/UNT3D`, offer an efficient way to migrate your development codebase to production by leveraging these backups directly. Simply pick one of the three most recent backups, copy it to your home directory, and retrieve your site master key from your `.env` file (the `APP_KEY`). Next, uncompress the backup using `p7zip` (you'll be prompted for the key) and extract any additional ZIP files (typically containing files and a database). Finally, restore the files to your server and manually import the database to ensure your server runs only the committed code.

### Create a Backup
To run a backup of your UNIT3D installation, navigate to your project directory and execute the Artisan backup command:


```bash
cd /var/www/html
php artisan backup:run
```

### Viewing Backup List

To view available backups, run:

```bash
cd /var/www/html
php artisan backup:list
```

---

## What is PHP Artisan?

PHP Artisan is the command-line interface (CLI) tool that comes with Laravel (and by extension, many Laravel-based applications like UNIT3D). Since UNIT3D is built on Laravel, it leverages PHP Artisan to perform a wide range of tasks essential for managing and maintaining the application. These tasks include:

- Running database migrations and seeding
- Clearing and caching configuration, routes, and views
- Managing queues and scheduled tasks
- Generating boilerplate code for controllers, models, and more

### Why View All Commands?

Using the following command, you can get a raw list of all available Artisan commands. This list is particularly useful for understanding the full capabilities of PHP Artisan and for troubleshooting or automating common tasks in UNIT3D:

```bash
cd /var/www/html
php artisan list --raw
```

> **NOTE:** Make sure you are in the project directory before running any commands:
> 
> ```bash
> cd /var/www/html
> ```
> 
> This ensures that all commands operate on the correct environment.



## Maintenance Mode

Before making any modifications or performing a restoration, it is **strongly recommended** to put your site into maintenance mode. This will prevent users from encountering errors or inconsistencies during the process.

### Enable Maintenance Mode

Navigate to your project directory and run the following command:

```bash
cd /var/www/html
php artisan down
```

This command puts your site into maintenance mode.

### Disable Maintenance Mode

Once your modifications or restoration steps are complete, bring your site back up by running:

```bash
cd /var/www/html
php artisan up
```
This command restores normal site operations.

_Ensure that all critical operations (such as backups or restorations) are completed before bringing your site out of maintenance mode._


## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Install Required Tools](#step-1-install-required-tools)
- [Step 2: Retrieve Your Application Key](#step-2-retrieve-your-application-key)
- [Step 3: Uncompress the Backup](#step-3-uncompress-the-backup)
- [Step 4: Restore the Files](#step-4-restore-the-files)
- [Step 5: Fix File Permissions](#step-5-fix-file-permissions)
- [Step 6: Reset PHP Artisan and PHP-FPM](#step-6-reset-php-artisan-and-php-fpm)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

- **User Privileges:** Sudo access is required for installing packages and changing file permissions.
- **Tools:**  
  - [7-Zip](https://www.7-zip.org/) (for extracting encrypted backups)  
  - `unzip` (for handling ZIP files)  
  - A text editor (e.g., `nano` or `micro`)
- **Backup File:** A UNIT3D backup file (for example: `[UNIT3D]2025-03-05-00-00-51.zip`) stored in your UNIT3D backup folder.
- **App Key:** You need your `APP_KEY` from the `.env` file, which is used as the decryption password when extracting the backup.

> **Note:** This tutorial was tested on **UNIT3D v9.0.1** running on **PHP 8.4**. If you are using a different version, some commands and steps may require adjustments.


---

## Step 1: Install Required Tools

First, ensure you have the tools needed to extract the backup.

```bash
cd ~
sudo apt update
sudo apt install p7zip-full unzip -y
```

p7zip-full is used for handling 7z archives, which might be used to compress your backup files.
unzip is used for handling ZIP files.


---

## Step 2: Retrieve Your Application Key
Your backup file is encrypted with your app’s APP_KEY. Open the .env file to locate it.

```bash
sudo nano /var/www/html/.env
```

Copy the APP_KEY value (look for line starting with 'APP_KEY=')

or

Retrieve key from environment:

```bash
sudo grep 'APP_KEY=' /var/www/html/.env
```

You will use this key when extracting the backup.

---

## Step 3: Uncompress the Backup

### 1. Create a Temporary Directory
Create a temporary directory to work in and navigate into it:

```bash
mkdir ~/tempBackup
cd ~/tempBackup
```

### 2. Copy the Backup File
Copy your backup file from its location (e.g., `.../storage/backups/UNT3D`) to the temporary directory. Adjust the file name and path as needed:

```bash
cp /var/www/html/storage/backups/UNIT3D/\[UNIT3D\]2025-03-05-00-00-51.zip ./
```

Tip: Instead of typing the full file name, copy and paste the directory path (e.g., `.../backups/UNIT3D/\[UNIT3D`), then press the Tab key to auto-complete the backup file name:

```bash
cp /var/www/html/storage/backups/UNIT3D/\[UNIT3D
```
> [!NOTE]
> Note: Remember to add ./ at the end of the command to specify the current directory as the destination.
> 
For example: `cp /var/www/html/storage/backups/UNIT3D/\[UNIT3D\]2025-03-22-17-29-59.zip ./`



### 3. Extract the Backup File Using 7z
Use `7z` to extract the encrypted backup file. When prompted, enter your `APP_KEY`. The password prompt will not echo your input:

```bash
7z x \[UNIT3D\]2025-03-05-00-00-51.zip
```

#### Example: APP_KEY Usage
If your `.env` file contains:

```bash
APP_KEY=base64:bT70DC4Ck7taYqP6ugqKIYbAbiEFbgECSdc03MwtXg=
```

When prompted during extraction, enter the password (note: your input will not be echoed):

```bash
base64:bT70DC4Ck7taYqP6ugqKIYbAbiEFbgECSdc03MwtXg=
```

### 4. Unzip the Extracted Archive
If the extraction process produces a standard ZIP file (e.g., `backup.zip`), extract it with:

```bash
unzip backup.zip
```

---

**Explanation:**  
- The `7z` command decrypts and extracts the backup using your `APP_KEY`.  
- If the output is a ZIP file, `unzip` will further extract the backup contents.


---

## Step 4: Restore the Files
After extraction, your backup files will be available in the temporary folder. Now, copy them back to your web server directory.

Copy the Entire html Folder:

```bash
sudo cp -rv ~/tempBackup/var/www/html/* /var/www/html/
```

Alternatively, Depending on Your Setup, You Might Need to Copy at a Higher Level:

```bash
sudo cp -rv ~/tempBackup/var/www/* /var/www/
```

Or Restore Specific Files:

```bash
sudo -u www-data cp -rf ./var/www/html/someFile/* /var/www/html/someFile/
```

Tip:
Choose the command that best fits your restoration needs. If you only need to restore specific files, copy only those.

---


## Step 4.1: Alternate Restoration Method Using entire /var/www folder! 

If you prefer an alternative method, you can create a zipped backup of your current www folder, then restore it from the zip archive. Follow these steps:


<div style="border: 2px solid #e74c3c; background-color: #f9e6e6; padding: 10px; border-radius: 5px; margin: 15px 0;">
  <strong>🚨 READ:</strong> Be careful, the steps below are an alternative method for backing up the entire <code>/var/www</code> folder! Only continue if you understand what you are doing.
</div>


### 1. Create a Zip Archive of the www Folder
First, create a zipped archive of your entire /var/www folder and save it to your temporary backup directory:



```bash
sudo zip -r ~/tempBackup/www_backup_$(date +%Y%m%d%H%M%S).zip /var/www
```

This command compresses the entire /var/www folder into a zip file named with a timestamp.

### 2. Unzip the Archive into a New Restoration Folder

Create a new folder (e.g., restore_www_TIMESTAMP) in your temporary backup directory and unzip the archive into it:

The folder should be named following the format restore_www_TIMESTAMP, where TIMESTAMP corresponds to the timestamp in your backup file.

> [!NOTE]
> **Important:** Replace `$(date +%Y%m%d%H%M%S)` with the actual timestamp from your backup file.
> 
> For example, if your backup file is named `www_backup_20250322062714.zip`, then set `TIMESTAMP=20250322062714`.



```bash
TIMESTAMP=$(date +%Y%m%d%H%M%S)
mkdir ~/tempBackup/restore_www_$TIMESTAMP
unzip ~/tempBackup/www_backup_$TIMESTAMP.zip -d ~/tempBackup/restore_www_$TIMESTAMP
```

> Ensure that the same timestamp is used for both creating and unzipping the archive. Alternatively, you can manually specify a consistent name.

### 3. Copy the Restored Files to the Correct Location
Once you have verified the restored files in the new folder, copy the entire contents back to your live directory:

```bash
TIMESTAMP=$(date +%Y%m%d%H%M%S)
sudo cp -a ~/tempBackup/restore_www_$TIMESTAMP/. /var/www/
```

The -a flag preserves file permissions, ownership, and timestamps.

---

## Step 5: Fix File Permissions
Once the files are restored, adjust the file permissions to ensure your web server can access them correctly.

```bash
cd /var/www/html
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type f -exec chmod 664 {} \;
sudo find /var/www/html -type d -exec chmod 775 {} \;
sudo chgrp -R www-data storage bootstrap/cache
sudo chmod -R ug+rwx storage bootstrap/cache
```

Explanation:

_chown changes the owner to www-data, which is typically the web server user.
chmod commands set the proper permissions: files are set to 664 and directories to 775.
The additional commands ensure that the storage and bootstrap/cache directories have the correct group and permissions for writing._

### 5.1 Optional: Reinstall Dependencies and Rebuild Assets

If reinstalling Node.js dependencies and running a build process is part of your backup recovery, then execute the command below.

Run this only if you do not have a valid node_modules folder or if your assets require rebuilding.

Note: Running this command with sudo may cause permission issues (such as EACCES errors), so if possible, consider running it as a non-root user or ensure that the permissions are correctly set after installation.

When to Use:
Use the optional command if your project uses Bun for dependency management and asset building and if you need to ensure that your dependencies and built assets are updated. 

> [!NOTE]
> If your backup already contains a valid node_modules folder with pre-built assets, this step can be skipped to avoid potential permission issues.

Optional Rebuild Command:

```bash
sudo rm -rf node_modules && sudo bun install && sudo bun run build
```

sudo rm -rf node_modules: Removes the current Node.js dependencies to allow a fresh install.
sudo bun install: Reinstalls all dependencies as specified in your project’s configuration.
sudo bun run build: Runs the build process to compile or bundle assets as needed.

---

## Step 6: Reset PHP Artisan and PHP-FPM
Finally, reset caches and queues with PHP Artisan and restart your PHP-FPM service.

```bash
cd /var/www/html
sudo php artisan set:all_cache && sudo systemctl restart php8.4-fpm && sudo php artisan queue:restart
```

Explanation:

_php artisan set:all_cache clears and rebuilds your application's cache.
Restarting PHP-FPM ensures that any changes are recognized by the PHP process.
Restarting the queue allows any queued jobs to continue without issues._


## Troubleshooting

Below are some common issues and their suggested solutions:

| **Symptom**                | **Solution**                                          |
|----------------------------|-------------------------------------------------------|
| 500 Internal Server Error  | Run `php artisan optimize:clear`                      |
| Database Connection Issues | Verify your `.env` credentials                        |
| Missing Files              | Re-run `rsync` with the `--checksum` option           |
| Permission Denied          | Reapply ACL permissions                               |
| Queue Workers Inactive     | Restart all workers with `sudo supervisorctl restart all` |



## Acknowledgements

This project was made possible thanks to airclay aka [ericlay](https://github.com/ericlay).



