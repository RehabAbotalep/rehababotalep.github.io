---
layout: post
title:  "Step-by-Step Guide: LAMP Stack Setup on Azure for Q2A and WordPress Hosting"
date:   2024-09-04 20:00:05 +0100
categories: [Blogging, Demo]
tags: [azure, question2answer, wordpress] 
---

This guide will walk you through the process of setting up a testing environment on Azure using an Ubuntu virtual machine (VM). We will cover the installation of necessary software and the configuration of the environment to host a Question2Answer (Q2A) or WordPress site. By the end of this guide, you’ll have a fully operational server environment, ready to deploy and manage your site efficiently.

## 1. Create a Resource Group

To begin, you’ll need to create a resource group in Azure, which serves as a logical container for your resources. Use Azure Cloud Shell and execute the following command:

```bash
az group create --name rg-q2a-test-uksouth-001 --location uksouth
```

This command sets up a resource group called `rg-q2a-test-uksouth-001` in the `uksouth` region.

## 2. Set Up a Virtual Machine (VM)

Next, create an Ubuntu 22.04 VM that will act as the server for your Q2A or WordPress application:

```bash
az vm create --resource-group rg-q2a-test-uksouth-001 --name vm-q2a-test-uksouth-001 --image Ubuntu2204 --admin-username azureuser --admin-password 'password'
```

- **VM Name:** `vm-q2a-test-uksouth-001`
- **Image:** Ubuntu 22.04
- **Admin Username:** `azureuser`
- **Admin Password:** Replace `'password'` with a secure password of your choice.

After running the command, Azure will provide details about the VM, including its `publicIpAddress`. Keep this information handy as it will be needed for SSH access and to navigate to the site later.

## 3. Allow HTTP Traffic on Port 80

To make your web application accessible, you'll need to open port 80 on your VM:

```bash
az vm open-port --port 80 --resource-group rg-q2a-test-uksouth-001 --name vm-q2a-test-uksouth-001
```

This command allows HTTP traffic to reach your VM, enabling users to access your web server via port 80.

## 4. Open Port 3389 for RDP Access (Optional)

If you wish to connect to your VM using Remote Desktop Protocol (RDP), open port 3389:

```bash
az vm open-port --port 3389 --priority 1100 --resource-group rg-q2a-test-uksouth-001 --name vm-q2a-test-uksouth-001
```

- **Priority:** `1100` (This ensures there is no conflict with other rules.)

## 5. Connect to the VM via SSH

With the VM running, connect to it using SSH. Use the command below, replacing `yourPublicIpAddress` with the VM's public IP address provided earlier:

```bash
ssh azureuser@yourPublicIpAddress
```
> Replace `yourPublicIpAddress` with the VM's public IP address and `azureuser` with the username you specified when creating the VM. 
{: .prompt-info }

## 6. Update and Upgrade the System

It’s important to keep your system updated. Start by updating the package lists and upgrading installed packages:

```bash
sudo apt upgrade -y
```

```bash
sudo apt upgrade -y
```

- `sudo apt update` updates the list of available packages and their versions.
- `sudo apt upgrade` installs the latest versions of all currently installed packages.

The `-y` flag automatically answers "yes" to prompts, making the upgrade process smooth and unattended.

## 7. Install Xfce and XRDP for GUI (Optional)

If a graphical user interface (GUI) is needed, install the Xfce desktop environment and XRDP for remote desktop access:

```bash
sudo DEBIAN_FRONTEND=noninteractive apt-get -y install xfce4
```
*This command installs the XFCE4 desktop environment on your system. XFCE4 is a lightweight graphical user interface suitable for remote desktop use.*

```bash
sudo apt install xfce4-session -y
```
*This command installs the XFCE4 session management package, which handles user sessions within the XFCE4 desktop environment.*

```bash
sudo apt-get -y install xrdp
```
*This command installs XRDP, a server that enables remote desktop access to your Linux machine using the Remote Desktop Protocol (RDP).*

```bash
sudo adduser xrdp ssl-cert
```
*This command adds the xrdp user to the ssl-cert group, allowing XRDP to use SSL certificates for encrypted connections.*

```bash
echo xfce4-session >~/.xsession
```
*This command configures XFCE4 to be the default desktop session when logging in via XRDP.*

```bash
sudo service xrdp restart
```
*This command restarts the XRDP service to apply the new configuration settings.*

## 8. Install Google Chrome or Firefox

For web browsing or file downloads directly on the VM, install a browser like Google Chrome or Firefox.

**To Install Google Chrome:**

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y
```

**To Install Firefox:**

```bash
sudo apt install firefox -y
```

## 9. Install Unzip or Unrar 

To manage compressed files, install `unzip` or `unar`:

**To Install Unzip:**

```bash
sudo apt install unzip
```

**To Install Unrar:**

```bash
sudo apt install unar
```

## 10. Install the LAMP Stack

For hosting Q2A or WordPress, you need to set up a LAMP stack (Linux, Apache, MySQL, PHP):

```bash
sudo apt update && sudo apt install lamp-server^ -y
```

## 11. Verify LAMP Installation

To ensure everything is installed correctly, check the versions of Apache, MySQL, and PHP:

```bash
apache2 -v
mysql -V
php -v
```

These commands verify that Apache, MySQL, and PHP are installed and running as expected.

## 12. Secure MySQL Installation

For added security, run the MySQL secure installation script.
This script prompts you to set a root password, remove anonymous users, disallow root login remotely, and more, to enhance your MySQL security.

```bash
sudo mysql_secure_installation
```
*This command is used to improve the security of your MySQL installation by guiding you through several important security-related steps*

## 13. Configure MySQL Root User

Finally, update the MySQL root user to use native password authentication, setting a strong password:

```bash
sudo mysql
```
```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourSecurePassword';
```
```bash
exit
```
> Replace `YourSecurePassword` with a strong password of your choice to secure your MySQL installation.
{: .prompt-info }

Following this guide will set up a ready-to-use environment for hosting your Q2A or WordPress site, optimized for performance and security on Azure. Adjust the configurations as needed to suit your specific requirements.

You can watch the following video that walks you through all the steps explained in this post.

{% include embed/youtube.html id='GGy0mtGQapU' %}