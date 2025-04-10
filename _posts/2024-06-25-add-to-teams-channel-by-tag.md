---
layout: post
title:  "PowerShell Script: Adding Members to MS Teams private Channels by Tag"
date:   2024-07-15 01:00:05 +0100
categories: [Blogging, Script]
tags: [azure, ms teams, powershell script, productivity] 
comments: true
---

I was working on a real use case scenario to add a large list of users to multiple private channels in MS Teams.
The first option that came to my mind was adding them manually by adding each user to each channel, but after some thought, I asked myself these questions:

1. Doing it manually will be time-consuming? The answer was "yes" as it's a large list and it involves not just one channel but several channels.
2. Is this a repetitive task? The answer was yes as new members are constantly joining our team. Additionally, if we create a new channel and want to add these members to the newly created one we will have to repeat the process.

Therefore, doing it manually will be time-consuming and this is a repeated task, that is why I thought about another solution to be more productive.

Since these users are assigned tags, what if we add them to the channels based on their tags? I couldn't find this option in MS Teams. This led me to think about developing a script that automate the process of adding users to the channels based on their tags.

All you need is to provide the channel link, and the tag name. The script will list all users that are assigned to the provided tag and add them to the specified channel.

## Script

```powershell
<#
.SYNOPSIS
Adds users assigned to a specified tag within a Microsoft Teams team to a specified channel.

.DESCRIPTION
This script performs operations including module import, authentication, and user management within Teams by using Microsoft Graph. It handles:
- Importing and installing necessary modules.
- Authenticating to Microsoft Graph.
- Retrieving team and channel IDs based on input links.
- Listing users assigned to a specified tag.
- Adding users to a specified channel.

.PARAMETER channelLink
The Microsoft Teams link from which the channel ID will be extracted.

.PARAMETER tagName
The name of the tag to retrieve users assigned to it.

.EXAMPLE
PS> .\AddTagMembersToChannel.ps1
This example prompts the user for input parameters and executes the script to add users to a channel.

.NOTES
Author: Rehab Ragab
Date: 11 Jul 2024
#>

# Imports and installs a PowerShell module if not already available
function Import-ModuleIfNeeded {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory = $true)]
        [string]$ModuleName
    )

    # Check if module is available and install if not
    if (-not (Get-Module -Name $ModuleName -ListAvailable)) {
        Write-Host "$ModuleName module is not installed. Installing from the PowerShell Gallery..."
        Install-Module -Name $ModuleName -Scope CurrentUser -Force -AllowClobber
    }
    Import-Module $ModuleName
}

# Authenticates the user to Microsoft Graph with necessary permissions
function Connect-MicrosoftGraph {
    Connect-MgGraph -Scopes "TeamworkTag.ReadWrite", "ChannelMember.ReadWrite.All", "User.Read.All"
}

# Extracts an ID from a URL based on a provided regex pattern
function Get-IdFromLink {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory = $true)]
        [string]$Link,
        [Parameter(Mandatory = $true)]
        [string]$Pattern,
        [string]$ErrorOutput = "Invalid or missing ID in the provided link."
    )

    if ($Link -match $Pattern) {
        return $Matches[1]
    } else {
        Write-Error $ErrorOutput
        return $null
    }
}

# Retrieves the ID of a tag by its display name within a specified team
function Get-TagId {
    param (
        [string]$TeamId,
        [string]$TagName
    )
    $tags = Get-MgTeamTag -TeamId $TeamId
    $tag = $tags | Where-Object { $_.DisplayName -eq $TagName }
    if ($tag) { return $tag.Id }
    else { Write-Error "Tag '$TagName' not found in the team."; return $null }
}

# Adds all users assigned to a specific tag to a channel in Microsoft Teams
function Add-UsersToChannel {
    param (
        [string]$TeamId,
        [string]$TagId,
        [string]$ChannelId,
        [string]$TagName
    )
    $tagMembers = Get-MgTeamTagMember -TeamId $TeamId -TeamworkTagId $TagId
    $iteration = 1  # Initialize the iteration counter
    foreach ($member in $tagMembers) {
        $user = Get-MgUser -UserId $member.UserId
        if ($user) {
            $params = @{
                "@odata.type"      = "#microsoft.graph.aadUserConversationMember"
                roles              = @("member")
                "user@odata.bind" = "https://graph.microsoft.com/v1.0/users('$($user.Id)')"
            }
            New-MgTeamChannelMember -TeamId $TeamId -ChannelId $ChannelId -BodyParameter $params
            Write-Host "$iteration. Added $($user.DisplayName) to the channel."
            $iteration++  # Increment the iteration counter
        } else {
            Write-Host "$iteration. Failed to retrieve user with ID: $($member.UserId)"
            $iteration++  # Increment the iteration counter even in case of failure
        }
    }
}

# Main execution block
Import-ModuleIfNeeded -ModuleName "Microsoft.Graph.Teams"
Connect-MicrosoftGraph 

$channelLink = Read-Host -Prompt "Please enter the Microsoft Teams channel link to add users to:"
$teamId = Get-IdFromLink -Link $channelLink -Pattern "groupId=([a-z0-9\-]+)" -ErrorOutput "Team ID not found."
$channelId = Get-IdFromLink -Link $channelLink -Pattern 'teams\.microsoft\.com/l/channel/([^/]+)' -ErrorOutput "Channel ID not found."

if ($teamId -and $channelId) {
    $tagName = Read-Host -Prompt "Please enter the tag name"
    $tagId = Get-TagId -TeamId $teamId -TagName $tagName
    if ($tagId) {
        Add-UsersToChannel -TeamId $teamId -TagId $tagId -ChannelId $channelId -TagName $tagName
    }
}

```
## Dive into the script

This script adds users assigned to a specified tag within a Microsoft Teams team to a specified private channel. It uses Microsoft Graph to perform operations.

### Functions

The script contains the following functions:

- **Import-ModuleIfNeeded:** This function checks if a specified PowerShell module is installed. If not, it installs the module from the PowerShell Gallery and then imports it.

- **Connect-MicrosoftGraph:** This function authenticates the user to Microsoft Graph with necessary permissions.

- **Get-IdFromLink:** This function extracts an ID from a URL based on a provided regex pattern. We use it to extract teamId and channelId from the provided Channel Link.

- **Get-TagId:** This function retrieves the ID of a tag by its display name within a specified team.

- **Add-UsersToChannel:** This retrieves all users that assigned the provided tag and add them to the provided channel.

### Parameters

- **channelLink:** The channel link in which users will be added to.
  
- **tagName:** The name of the tag to retrieve users assigned to it.

When you run the script for the first time, it will prompt you to log in and grant the required permissions to the app to access specific resources.

![](/assets/img/add-to-teams-channel-by-tag/13-pick-account.png)

![](/assets/img/add-to-teams-channel-by-tag/14-permission-requested.png)

During this process, Azure AD automatically creates an Enterprise Application. This Application handles permissions and maintains the authentication context for accessing Microsoft Graph resources.

![](/assets/img/add-to-teams-channel-by-tag/15-created-app.png)

In subsequent runs, the script uses the previously created Enterprise Application to authenticate and perform its tasks, provided the required permissions have already been granted.

## Running the script

1. Create a new powershell file, and paste the script inside it
2. Start PowerShell from your Start menu.
3. Navigate to Your Script: `cd path\to\your\script`
4. Execute the Script: `.\YourScriptName.ps1`
5. The script will prompt you to log in and grant the required permissions 
6. The script will prompt you for the channel link, and the tag name.
7. To get the channel link, navigate to the desired MS teams channel that you would like to add users to and click on the three dots ..., then get link to channel
![Get channel link](/assets/img/add-to-teams-channel-by-tag/12-get-channel-link.png) 
