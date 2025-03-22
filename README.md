# UNIT3D Backup Restoration Tutorial

_Whether you've made an error and need to restore from a backup, or you're just looking to learn something new, this tutorial has got you covered._

This guide explains how to restore a UNIT3D backup on your server. It covers installing required tools, uncompressing the backup using your app key, copying files to their correct locations, fixing file permissions, and resetting caches with PHP Artisan.

> **Note:** This tutorial assumes you have already created a backup using PHP Artisan. We also assume you are familiar with basic terminal commands and have sudo privileges on your system.

## Using Built-In Backups

Built-in backups, located in `.../storage/backups/UNT3D`, offer an efficient way to migrate your development codebase to production using Git. Simply pick one of the three most recent backups, copy it to your home directory, and retrieve your site master key from your `.env` file (the `APP_KEY`). Next, uncompress the backup using `p7zip` (you'll be prompted for the key) and extract any additional ZIP files (typically containing files and a database). Finally, restore the files to your server and manually import the database to ensure your server runs only the committed code.

---

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
Create a Temporary Directory for the Backup:

```bash
mkdir ~/tempBackup
cd ~/tempBackup
```

Copy the Backup File, located in: `.../storage/backups/UNT3D`, to the Temporary Directory:

Adjust the file name and path if necessary.

```bash
cp /var/www/html/storage/backups/UNIT3D/\[UNIT3D\]2025-03-05-00-00-51.zip ./
```

Extract the Backup File with 7z:

When prompted, enter the APP_KEY you retrieved earlier.

```bash
7z x \[UNIT3D\]2025-03-05-00-00-51.zip
```

If the Archive Contains a ZIP File, Unzip It:

```bash
unzip backup.zip
```

The 7z command will extract the encrypted file using your APP_KEY. After extraction, if you have a standard ZIP file (named backup.zip in this example), you can unzip it to get the backup files.

---

## Step 4: Restore the Files
After extraction, your backup files will be available in the temporary folder. Now, copy them back to your web server directory.

Copy the Entire html Folder:

```bash
sudo cp -rf ./var/www/html /var/www/html
```

Alternatively, Depending on Your Setup, You Might Need to Copy at a Higher Level:

```bash
sudo cp -rf ./var/www /var/www
```

Or Restore Specific Files:

```bash
sudo -u www-data cp -rf ./var/www/html/someFile /var/www/html/someFile
```

Tip:
Choose the command that best fits your restoration needs. If you only need to restore specific files, copy only those.

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

chown changes the owner to www-data, which is typically the web server user.
chmod commands set the proper permissions: files are set to 664 and directories to 775.
The additional commands ensure that the storage and bootstrap/cache directories have the correct group and permissions for writing.

---

## Step 6: Reset PHP Artisan and PHP-FPM
Finally, reset caches and queues with PHP Artisan and restart your PHP-FPM service.

```bash
cd /var/www/html
sudo php artisan set:all_cache && sudo systemctl restart php8.4-fpm && sudo php artisan queue:restart
```

Explanation:

php artisan set:all_cache clears and rebuilds your application's cache.
Restarting PHP-FPM ensures that any changes are recognized by the PHP process.
Restarting the queue allows any queued jobs to continue without issues.


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



