---
layout: post
title:  "PowerShell Script: Tracking Field Usage Across Azure DevOps Projects"
date:   2024-06-01 23:00:05 +0100
categories: [DevOps & Automation, Script]
tags: [azure devops, work items, powershell, productivity] 
---

In response to a practical need to identify and quantify the usage of a specific field across projects in Azure DevOps, I developed this PowerShell script.

This script interacts with the Azure DevOps API, to retrieve a list of projects and analyze each for the presence of a designated field. It not only identifies which projects utilize this field but also counts the work items within those projects that include it. 

Additionally, the script compiles these findings into a markdown file. This file serves as a summary, presenting a clear breakdown of the number of work items per project that make use of the field, thus providing valuable insights into field utilization across your Azure DevOps environment.

## Script

```powershell
# Prompt user for inputs
$organization = Read-Host -Prompt "Enter the Azure DevOps organization URL in format https://dev.azure.com/ORGNAME"
$personalAccessToken = Read-Host -Prompt "Enter your personal access token"
$field = "FIELD_REFERENCE_NAME"
$batchSize = 10  # Define the number of projects to process in each batch

# Function to encode the personal access token and return the authorization header
function Get-AuthorizationHeader {
    param (
        [string]$token
    )
    $base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$token"))
    return @{ Authorization = "Basic $base64AuthInfo" }
}

# Define the headers
$headers = Get-AuthorizationHeader -token $personalAccessToken

# Define the API URL template
$apiUrlTemplate = "$organization/{0}/_apis/{1}?api-version={2}"

# Function to get the list of all projects
function Get-AllProjects {
    $url = $apiUrlTemplate -f "", "projects", "7.0"
    return (Invoke-RestMethod -Uri $url -Method Get -Headers $headers).value
}

# Function to get the list of fields in a project
function Get-ProjectFields {
    param (
        [string]$projectId
    )
    $url = $apiUrlTemplate -f "$projectId", "wit/fields", "7.1-preview.3"
    return (Invoke-RestMethod -Uri $url -Method Get -Headers $headers).value
}

# Function to query work items for a project
function Query-WorkItems {
    param (
        [string]$projectName,
        [string]$field
    )
    $query = @"
    SELECT [System.Id], [System.WorkItemType]
    FROM workitems
    WHERE [System.TeamProject] = '$projectName'
    AND [System.WorkItemType] <> ''
    AND [$field] <> ''
"@
    $url = $apiUrlTemplate -f "$projectName", "wit/wiql", "6.0"
    $body = @{ query = $query } | ConvertTo-Json
    return (Invoke-RestMethod -Uri $url -Method Post -Headers $headers -Body $body -ContentType "application/json").workItems
}

# Main script to retrieve and filter projects by the specified field
$allProjects = Get-AllProjects
$projectsUsingField = @()

Write-Output "=============================================="
Write-Output "List of projects using the field '$field'"
Write-Output "=============================================="

$counter = 1

# Implement batching mechanism
for ($i = 0; $i -lt $allProjects.Count; $i += $batchSize) {
    $currentBatch = $allProjects[$i..($i + $batchSize - 1)]
    foreach ($project in $currentBatch) {
        $projectId = $project.id
        $projectName = $project.name
        $projectFields = Get-ProjectFields -projectId $projectId

        if ($projectFields | Where-Object { $_.referenceName -eq $field }) {
            $projectsUsingField += $projectName
            Write-Output "${counter}: $projectName"
            $counter++
        }
    }
}

Write-Output "=============================================="
Write-Output "Field ($field) Statistics:"
Write-Output "=============================================="

# Prepare markdown content
$markdownContent = @"
|No|Project Name| Field Usage|
|--|------------|------------|
"@

# Output the list of projects using the specified field and query work items
if ($projectsUsingField.Count -eq 0) {
    Write-Output "No projects are using the field '$field'."
} else {
    $index = 1
    foreach ($projectName in $projectsUsingField) {
        $workItems = Query-WorkItems -projectName $projectName -field $field
        $workItemCount = if ($workItems) { $workItems.Count } else { 0 }
        $markdownContent += "`n|$index|$projectName|$workItemCount|"
        $index++
        Write-Output "$projectName has $workItemCount work items using the field"
    }

    # Define the markdown file path relative to the script's directory
    $markdownFilePath = Join-Path -Path $PSScriptRoot -ChildPath "Field-Usage.md"

    # Write the markdown content to the file
    $markdownContent | Out-File -FilePath $markdownFilePath -Encoding utf8

    Write-Output "Markdown file created at: $markdownFilePath"
}
```

## Dive into the script

Let's break down each part of the script for a clearer understanding:

**1. Prompting User Inputs**

The script starts by prompting the user for the Azure DevOps organization URL and a personal access token. These are essential for authenticating API requests.

**1. Function for Authorization Header**

A function named `Get-AuthorizationHeader` is defined to generate the authorization header using the provided personal access token. This header will be utilized in subsequent REST API calls to Azure DevOps.

**1. API URL Template and Functions for Azure DevOps Interactions**

  - **$apiUrlTemplate**: This variable holds a template for constructing Azure DevOps API URLs with placeholders for project ID, resource type, and API version.
  - **Get-AllProjects**: Retrieves a list of all projects within the Azure DevOps organization.
  - **Get-ProjectFields**: Fetches the list of fields defined within a specific project.
  - **Query-WorkItems**: Executes a WIQL query to retrieve work items for a given project based on a specified field.

**1. Retrieving and Filtering Projects**

The script retrieves all projects and iterates through them in batches. For each project, it checks if the specified field is used and adds the project name to a list if it is.

**1. Outputting Project and Field Statistics**

The script outputs a list of projects using the specified field, along with the count of work items associated with that field in each project. Additionally, it generates markdown content for further documentation and analysis.

**1. File Output**

If projects are found using the specified field, the script writes markdown content to a file named "Field-Usage.md" in the script's directory, providing a structured overview of field usage across projects.

## Running the Script

**1. Set Up Your Environment**

- PowerShell: Ensure you have PowerShell installed on your computer. PowerShell 5.1 or higher is recommended for better compatibility with the script features.

**2. Prepare the Script**

- Place the Script: Save your script in a known directory.
- Customize the Script: Make sure the $field variable in your script matches the field you want to analyze.

**3. Run the Script**

- Open PowerShell: Start PowerShell from your Start menu or taskbar.
  
- Navigate to Your Script: `cd path\to\your\script`
  
- Execute the Script: `.\YourScriptName.ps1`

- Input Requirements: The script will prompt you for the Azure DevOps organization URL and your personal access token. Make sure to enter these accurately to authenticate your API requests.

  - Organization URL: Enter the URL in the format https://dev.azure.com/ORGNAME.
  - Personal Access Token (PAT): Input your generated PAT from Azure DevOps. Ensure that the token has the necessary scopes for reading projects and work items.

**4. View Results**

- Review the Markdown File: After the script finishes executing, it outputs the path to the markdown file it created (Field-Usage.md). You can open this file in any markdown viewer or text editor to see the report.

This is an example of how the generated markdown file looks like:

[![Generated Markdown File](/assets/img/field-usage/output-markdown-file.png)](/assets/img/field-usage/output-markdown-file.png)


## References

[Projects - List](https://learn.microsoft.com/en-us/rest/api/azure/devops/core/projects/list?view=azure-devops-rest-7.1&tabs=HTTP)

[Fields - List](https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/fields/list?view=azure-devops-rest-7.1&tabs=HTTP)

[Wiql - Query By Wiql](https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/wiql/query-by-wiql?view=azure-devops-rest-7.0&tabs=HTTP)

