---
layout: post
title:  "Step-by-Step Guide: Deploying or Restoring Question2Answer on Azure VM"
date:   2024-09-20 07:00:05 +0100
categories: [Blogging, Demo]
tags: [azure, question2answer] 
---

This guide provides detailed instructions for setting up a testing environment on Azure by deploying or restoring the Question2Answer (Q2A) application on an Ubuntu Virtual Machine (VM). It covers preparing a LAMP stack environment, configuring necessary components, and deploying the Q2A application. Follow these steps to ensure a seamless deployment and configuration for testing purposes.

## 1. Setting Up a LAMP Stack Environment

To get started, you’ll need to prepare a LAMP (Linux, Apache, MySQL, PHP) stack environment. For detailed instructions on setting up a LAMP stack on your Ubuntu VM, check out this guide:

[Step-by-Step Guide: LAMP Stack Setup on Azure for Q2A and WordPress Hosting](https://rehababotalep.github.io/posts/setup-lamp-stack/)

## 2. Downloading and Setting Up Question2Answer

Begin by downloading the Question2Answer zip file, either the latest version or a backup from your production environment.

- **Extract the Question2Answer Archive:** Unzip the downloaded Question2Answer zip file.

```bash
unzip /home/azureuser/Downloads/question2answer-latest.zip
```

- **Clean the Web Directory:** Remove any existing files in the Apache web directory to ensure a clean installation.

```bash
sudo rm -r /var/www/html/*
```

- **Copy Question2Answer to the Web Directory:** Move the extracted Question2Answer files to the Apache web directory.

```bash
sudo cp -r /home/azureuser/Downloads/question2answer-1.8.8/public_html/. /var/www/html
```

- **Set Ownership and Permissions:** Modify ownership and permissions to ensure the files are accessible by the web server.

```bash
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

## 3. Configuring MySQL for Question2Answer

- **Log in to MySQL:** Access the MySQL command line with the root user.

```bash
sudo mysql -p
```
*This command will prompt you to enter your MySQL root password.*

- **View All MySQL Users:**

```sql
SELECT User, Host FROM mysql.user;
```
*This query lists all existing users and their host permissions.*

- **Create a MySQL User for Question2Answer:**

```sql
CREATE USER 'q2aUser'@'localhost' IDENTIFIED BY 'securePassword';
```
*This command creates a new user `q2aUser` with a secure password. Make sure to use a strong password.*

- **Check Existing Databases:**

```sql
SHOW DATABASES;
```
*This lists all the databases on the server to avoid naming conflicts.*

- **Create a New Database for Question2Answer:**

```sql
CREATE DATABASE question2answer;
SHOW DATABASES;
```
*This creates a new database called `question2answer` and confirms its creation.*

- **Grant Permissions to the New User:**

```sql
GRANT ALL PRIVILEGES ON question2answer.* TO 'q2aUser'@'localhost';
FLUSH PRIVILEGES;
```
*Grants full privileges to `q2aUser` on the `question2answer` database and applies the changes.*

## 4. Importing a Database Backup (Optional)

If you're restoring from a backup, use the following commands to import the database:

```sql
USE question2answer;
source /home/azureuser/Downloads/backup.sql;
```

## 5. Configuring Question2Answer

- **Edit the Configuration File:** Connect Question2Answer to your MySQL database by setting up the `qa-config.php` file.

For a new installation, start by copying the sample configuration file:

```bash
sudo cp qa-config-example.php qa-config.php
```

**Edit the Configuration File:**

```bash
sudo nano qa-config.php
```

In the `qa-config.php` file, update the following parameters:

- **Database Name:** Replace `'your-database-name'` with the name of your Q2A database.
- **Database User:** Replace `'your-database-username'` with the MySQL username you created.
- **Database Password:** Replace `'your-database-password'` with the password you set for the MySQL user.

Ensure these settings match the MySQL credentials you configured earlier. Save and exit the file after editing.

## 6. Final Checks and Testing

Once you've completed the above steps, navigate to `http://<yourPublicIPAddress>` in a web browser to access your Question2Answer installation. If you have restored a backup, the application should display the restored data.

By following these steps, you will have successfully deployed or restored Question2Answer on your Azure VM. Adjust configurations and commands as needed to suit your specific environment and requirements.

You can watch the following video that walks you through all the steps explained in this post.

{% include embed/youtube.html id='GGy0mtGQapU' %}
