---
layout: post
title:  "Migrating Inherited Process in Azure DevOps"
date:   2023-10-21 16:40:05 +0100
categories: [Blogging, Post]
tags: [azure-devops, process-migrator, migration]
comments: true
---

In this post, we'll explore how to migrate a customized inherited process from one Azure DevOps organization to another. We'll walk through the process using the powerful [process-migrator](https://github.com/microsoft/process-migrator) tool, developed by Microsoft.

## Prerequisites

Before we dive into the process migration journey with the process-migrator tool, let's make sure your environment is ready. 
To get started, you'll need to have Node.js and NPM installed on your machine.

1- Verify Node.js Installation

- Open your command prompt (cmd).
- Type node --version and check if Node.js is installed. If it's not installed, we'll need to proceed with the installation.

![Verify Node.js Installation](/assets/img/process-migration/1-node-not-found.png)

2- Download Node.js

Visit [node.js.org](https://nodejs.org/en/download) to download the latest LTS version, which includes npm.

3- Select Your Installer

Choose the installer that matches your machine. For example, if you're using Windows, select the Windows Installer.

![Download Node.js](/assets/img/process-migration/2-download-node.js.png)

4- Installation

Locate the downloaded Node.js installer and double-click it to begin the installation, as shown in the screenshot below.

![Install Node.js](/assets/img/process-migration/3-install-node.js.gif)

5- Verification

To confirm the successful installation of Node.js, repeat Step (1) by opening your command prompt and running `node --version`. This time, you should see the installed Node.js version displayed.
 
![Verification](/assets/img/process-migration/4-node-installed.png)

## Installing process-migrator

Now that your environment is ready, install the process-migrator tool by running the following command in your command prompt (cmd):

`npm install process-migrator -g`

![Install Process Migrator](/assets/img/process-migration/5-install-process-migrator.png)

## Creating the configuration file

1. Navigate to the desired location where you want to create the configuration file.

2. Run the command `process-migrator`. This will generate the configuration file in the specified location.

![Configuration File Creation](/assets/img/process-migration/6-init-config.gif)

This is how the created configuration file looks like:

![Configuration File](/assets/img/process-migration/7-config-file.png)

## Preparing the configuration

To configure the process-migrator tool, you'll need to specify the following parameters in the configuration file:

- **sourceAccountUrl:** This is the URL of your source organization in Azure DevOps.
- **sourceAccountToken:** Your Personal Access Token (PAT) for the source organization.
- **targetAccountUrl:** This is the URL of your target organization in Azure DevOps.
- **targetAccountToken:** Your Personal Access Token (PAT) for the target organization.
- **sourceProcessName:** The name of the source process to be migrated.


1- Copy the URL of your Source Organization and paste it for sourceAccountUrl.

2- Copy the URL of your Target Organization and paste it for targetAccountUrl.

3- Create PAT for both the source and target organizations.

Here's how to set up your PAT:

**For the Source Organization:**

1- Click on your user settings and select "Personal Access Token".

![Create PAT Step (1)](/assets/img/process-migration/8-create-pat-1.png)

2- Click `New Token`.

3- Enter the following details:

- Name

- Ensure you've selected the Source organization.

- For PAT scope, choose custom-defined, and ensure the "Work Items" permission has `Read, Write, & Manage` checked.

- Click Create

![Create PAT Step (2)](/assets/img/process-migration/9-create-pat-2.png)

3- Copy the value of the PAT, and paste it for sourceAccountToken

![Create PAT Step (3)](/assets/img/process-migration/10-create-pat-3.png)

Repeat these steps for the Target Organization to create a PAT and paste its value for targetAccountToken.

4- Go to the Source organization to copy the source process name, and then paste it for sourceProcessName.

![Source Org Process](/assets/img/process-migration/11-source-org-process.png)

After defining all the required parameters, your configuration file should look something like this:

![Config File](/assets/img/process-migration/12-config-after-updating-values.png)

**NOTE:** The configuration file is in `JSONC` format, which means you don't have to remove comment lines for it to work. 

## Running the Migration

You're all set to start the migration process. Simply open the command prompt (cmd) and type: 

`process-migrator`

**NOTES:** 

1. You can also specify the following switches:

   - `--mode=<migrate (default)/import/export>`: Choose the migration mode.
   - `--config=<your-configuration-file-path>`: Specify the path to your configuration file.

2. When running the command from the configuration file location, it is optional to specify the configuration file path. If the configuration file is located elsewhere, you should add its path using the `--config` switch. The full command would then be: `process-migrator --config=<your-configuration-file-path>`.

3. The tool's default mode is migrate, so it is optional to specify it. However, if you intend to use the tool for import or export only, you should specify the mode using the `--mode` switch. The complete command would be: `process-migrator --mode=import/export`.


**Migration Process:**

1. The tool will start by exporting the Source process template to a JSON file.
2. Subsequently, it will import the exported template into the Target organization.

![Migrated](/assets/img/process-migration/13-process-migrated-successfully.png)

**Verification:**

1. Once the migration is successfully completed, you can review all work item types.
2. Verify that their fields, states, and rules have been migrated successfully.

Here are the Target organization processes before and after the Migration:

**Before Migration:**

![Processes before Migration](/assets/img/process-migration/14-target-processes-before-migration.png)

**After Migration:**

![processes after Migration](/assets/img/process-migration/15-target-processes-after-migration.png)

The exported JSON and log files are located within the `Output` folder, in the same directory as your configuration file.

![Output Folder Location](/assets/img/process-migration/16-output-folder.png)

![View from Inside the Output Folder](/assets/img/process-migration/17-output-from-inside.png)

**[⬆ Back To Top](#in-this-post)**
