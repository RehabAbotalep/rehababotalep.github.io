---
layout: post
title: "Step-by-Step Guide: Deploying or Restoring WordPress on Azure VM"
date: 2024-10-23 22:00:05 +0100
categories: [Blogging, Demo]
tags: [azure, wordpress] 
---

**Looking to deploy or restore your WordPress site on an Azure VM?** This comprehensive guide walks you through every detail of setting up a LAMP stack, configuring essential components, and deploying or restoring WordPress on an Azure-based Ubuntu VM. Whether you're restoring from a backup or setting up a fresh WordPress instance, these steps ensure a smooth and efficient process.

## 1. Preparing a LAMP Stack Environment for WordPress on Azure

To begin deploying or restoring WordPress on Azure, you must first prepare your LAMP stack environment. A properly configured LAMP stack is essential for running WordPress efficiently on any Linux-based server. Follow this detailed guide to **prepare your LAMP environment on an Azure VM**:

[Step-by-Step Guide: LAMP Stack Setup on Azure for Q2A and WordPress Hosting](https://rehababotalep.github.io/posts/setup-lamp-stack/)

## 2. Download and Set Up WordPress

Next, you’ll need the WordPress application. You can either **download the latest version of WordPress** or restore a previous backup from your production environment.

### Steps to Extract WordPress Files:

- **Extract WordPress from a backup file:**
  
```bash
unar /home/azureuser/Downloads/backup.tgz
```
*Use the `unar` command to extract your backup to the current directory. Ensure `unar` is installed on your Azure VM.*

## 3. Restoring WordPress Files to Your Azure VM

To deploy your WordPress site, you’ll need to remove any existing files from your server’s web root directory and copy over the restored files.

- **Remove current web root files:**
  
```bash
sudo rm -r /var/www/html/*
```
*Clears the current web directory to prepare for the new WordPress files.*

- **Copy backup files to the web root:**
  
```bash
sudo cp -a /home/azureuser/backup/public_html/. /var/www/html
```
*Restores the WordPress site by copying the backup files to the web server's root directory.*

- **Set correct file permissions:**
  
```bash
sudo chown -R www-data:www-data /var/www/html/
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
```
*Ensures correct ownership and permission settings to allow WordPress to function properly.*

## 4. Setting Up Your MySQL Database for WordPress

Your WordPress site needs a MySQL database. Here’s how to configure one:

- **Log in to MySQL:**

```bash
sudo mysql -p
```
*Access the MySQL interface with administrative privileges.*

- **Create a new MySQL user for WordPress:**
  
```sql
CREATE USER 'wordpressUser'@'localhost' IDENTIFIED BY 'YourStrongPassword';
```
*Creates a new user to handle WordPress database connections.*

- **Create the WordPress database:**
  
```sql
CREATE DATABASE wordpress;
```

- **Grant necessary privileges to the new user:**
  
```sql
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressUser'@'localhost';
FLUSH PRIVILEGES;
```

## 5. Importing the WordPress Database and Configuring Settings

If you're restoring a site, you may have a backup of your MySQL database:

- **Import your database:**
  
```sql
USE wordpress;
source /home/azureuser/backup/backup.sql;
```
*Imports the SQL backup into the newly created WordPress database.*

- **Update the site URL:**
  
```sql
UPDATE wp_options SET option_value = 'http://test.example.com/' WHERE option_name = 'siteurl';
UPDATE wp_options SET option_value = 'http://test.example.com/' WHERE option_name = 'home';
exit;
```
*Adjusts your WordPress site's URL settings to match the new domain or IP.*

## 6. Removing Unnecessary WordPress Plugins

To ensure optimal performance, it’s recommended to remove unnecessary plugins, especially after restoring from a backup:

- **Remove unused plugins:**

```bash
sudo rm -r /var/www/html/wp-content/plugins/object-cache-pro
```

## 7. Installing Required PHP Extensions for WordPress on Azure

Some plugins, especially LMS plugins like MasterStudy, require additional PHP extensions:

- **Install the `mbstring` extension:**
  
```bash
sudo apt-get install php-mbstring
```
*This extension handles multibyte strings, which are necessary for many LMS features.*

## 8. Configuring Apache for WordPress

Ensure Apache is configured properly to run WordPress:

- **Enable URL rewriting for pretty permalinks:**
  
```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

- **Edit Apache configuration to allow .htaccess overrides:**
  
```bash
sudo nano /etc/apache2/apache2.conf
```
- **Add the following configuration:**

```
<Directory /var/www/html/>
  Options Indexes FollowSymLinks
  AllowOverride All
  Require all granted
</Directory>
```

## 9. Installing and Using WP-CLI for WordPress Management

WP-CLI simplifies WordPress management. Here's how to install and use it:

- **Download WP-CLI:**
  
```bash
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
```

- **Make WP-CLI executable:**
  
```bash
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp
wp --info
```

## 10. Replacing URLs and Flushing the Cache

When restoring or migrating a WordPress site, it’s essential to update all URLs:

- **Replace all instances of the old URL with the new one:**
  
```bash
wp search-replace 'https://example.com/' 'http://test.example.com/' --all-tables
wp cache flush
```

## 11. Adjusting WordPress Plugin Permissions

Make sure the plugins directory has the correct ownership and permissions:

- **Fix plugin ownership:**
  
```bash
sudo chown -R www-data:www-data /var/www/html/wp-content/plugins/
sudo chmod -R 755 /var/www/html/wp-content/plugins/
```

## 12. Final Checks and Testing

After completing the deployment or restoration, verify that everything is working correctly:

- Test your WordPress site by visiting the URL.
- Log in to the admin dashboard to check plugins and settings.
- Ensure that the **MasterStudy LMS plugin** (if installed) is functioning properly.

## Conclusion

Deploying or restoring WordPress on an Azure VM involves several steps, from setting up a LAMP stack to configuring MySQL and Apache. By following this guide, you'll ensure a seamless WordPress installation, whether for a fresh site or restoring an existing one. Don't forget to regularly back up your WordPress site and databases to make future restorations easier!

You can watch the following video that walks you through all the steps explained in this post.

{% include embed/youtube.html id='GGy0mtGQapU' %}
