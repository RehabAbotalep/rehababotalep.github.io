---
layout: post
title: "PowerShell Script: Automate Field Mapping in Azure DevOps Migration"
date: 2025-04-10 13:00:05 +0100
categories: [Blogging, Script]
tags: [powershell script, productivity, azure devops, migration]
---

While working on real projects that involved migrating work items between Azure DevOps projects, I ran into a repetitive and time-consuming step: mapping fields between the source and target.

The tool I used [azure-devops-migration-tools](https://github.com/nkdAgility/azure-devops-migration-tools), supports field mapping through `FieldMergeMap`, but figuring out which fields are missing and writing that config manually isn’t fun, especially if you’re dealing with a lot of custom fields.

To make life easier, I developed a PowerShell script that:

- Connects to both projects
- Compares the fields of a given work item type
- Outputs the correct mapping structure in the format the migration tool expects

## The Challange

When migrating work items using azure-devops-migration-tools, you often need to define FieldMaps in your configuration file to tell the tool how to handle fields that exist in the source but not in the target project.

```json
{
  "FieldMapType": "FieldMergeMap",
  "ApplyTo": ["Bug"],
  "formatExpression": "{0} \n {1}",
  "sourceFields": [
    "Custom.FieldA",
    "Custom.FieldB"
  ],
  "targetField": "Custom.FieldC"
}
```
When you're dealing with many custom fields, manually identifying and mapping them becomes tedious and prone to mistakes. That's where this script comes in.

## The Solution

To streamline this process, I created a PowerShell script that:

- Connects to both the source and target Azure DevOps projects using their respective PATs (Personal Access Tokens)
- Retrieves field definitions for a specified work item type (e.g., Bug, Task)
- Compares the fields to find those that exist in the source but are missing in the target
- Generates a `FieldMergeMap` block in the correct format, ready to paste into your migration config file
- Outputs the result to a `.txt` file and displays it in the terminal

This automation helps reduce manual effort and ensures more accurate mapping during migration.

## Sample Output

When executed, the script produces an output like:

```json
"sourceFields": [
    'Custom.FieldA',
    'Custom.FieldB'
],
"formatExpression": "Field A: {0} <br/> Field B: {1} <br/>",
```

## Script

```powershell
param (
    [Parameter(Position = 0, mandatory = $false)]
    [string]  $sourceOrganization = "SOURCE ORGANIZATION",

    [Parameter(Position = 1, mandatory = $false)]
    [string]  $sourceProject = "SOURCE PROJECT",

    [Parameter(Position = 2, mandatory = $false)]
    [string]  $sourceAccessToken = "SOURCE ACCESS TOKEN",

    [Parameter(Position = 3, mandatory = $false)]
    [string]  $targetOrganization = "TARGET ORGNAIZATION",

    [Parameter(Position = 4, mandatory = $false)]
    [string]  $targetProject = "TAREGT PROJECT",

    [Parameter(Position = 5, mandatory = $false)]
    [string]  $targetAccessToken = "TARGET ACCESS TOKEN",

    [Parameter(Position = 6, mandatory = $false)]
    [string]  $workItemType = "WORK ITEM TYPE"
)

# Function to construct the API URL
function ConstructApiUrl {
    param (
        [string]$organization,
        [string]$project,
        [string]$workItemType
    )
    return "https://dev.azure.com/$organization/$project/_apis/wit/workitemtypes/$($workItemType)?api-version=7.0"
}

# Function to encode PAT for authentication
function GenerateBase64AuthInfo {
    param (
        [string]$pat
    )
    return [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$pat"))
}

# Function to make the API request
function Invoke-ApiRequest {
    param (
        [string]$url,
        [string]$base64AuthInfo
    )
    try {
        return Invoke-RestMethod -Uri $url -Method Get -Headers @{
            Authorization = "Basic $base64AuthInfo"
            "Content-Type" = "application/json"
        } -ErrorAction Stop
    } catch {
        Write-Host "API request to '$url' failed." -ForegroundColor Red
        Write-Host "Details: $($_.Exception.Message)" -ForegroundColor Yellow
    }
}


# Function to clean and extract fields JSON
function Get-FieldsJson {
    param (
        [string]$response
    )
    $cleanJson = $response -creplace "[^\x20-\x7E]", ""  # Remove non-printable ASCII characters
    $cleanJson = $cleanJson.Trim()

    if ($cleanJson -match '"fields"\s*:\s*(\[[^\]]+\])') {
        return $matches[1]  # Extract JSON array of fields
    } else {
        Write-Output "Error: 'fields' array not found in JSON."
        return $null
    }
}

# Function to fetch and process API data for a specific organization and project
function Get-ApiData {
    param (
        [string]$organization,
        [string]$project,
        [string]$workItemType,
        [string]$pat
    )

    $url = ConstructApiUrl -organization $organization -project $project -workItemType $workItemType
    $base64AuthInfo = GenerateBase64AuthInfo -pat $pat
    $response = Invoke-ApiRequest -url $url -base64AuthInfo $base64AuthInfo

    if ($response -is [string]) {
        $fieldsJson = Get-FieldsJson -response $response

        if ($fieldsJson) {
            try {
                return $fieldsJson | ConvertFrom-Json -ErrorAction Stop
            } catch {
                Write-Output "Failed to parse 'fields' JSON."
                Write-Output "Raw Extracted Fields JSON: $fieldsJson"
            }
        }
    } else {
        if ($response.PSObject.Properties.Name -contains "fields") {
            return $response.fields
        } else {
            Write-Output "Error: 'fields' property not found."
        }
    }
    return $null
}

# Function to compare fields from two arrays
function Compare-Fields {
    param (
        [Parameter(Mandatory)]
        [array]$sourceFields,
        [Parameter(Mandatory)]
        [array]$targetFields
    )

    # Create a HashSet for fast lookup of reference names in targetFields
    $targetFieldsSet = @{}
    foreach ($targetField in $targetFields) {
        $targetFieldsSet[$targetField.referenceName] = $true
    }

    # Find the fields in the first array but not in the second using HashSet lookup
    return $sourceFields | Where-Object { -not $targetFieldsSet[$_.referenceName] }
}

# Function to format comparison result
function Format-ComparisonResult {
    param (
        [Parameter(Mandatory)]
        [array]$unmatchedFields
    )

    # Prepare formatExpression with the correct field names and placeholders
    $formatExpression = $unmatchedFields | ForEach-Object { 
        $_.name + ": {" + ($unmatchedFields.IndexOf($_)) + "} <br/>" 
    }

    # Join the formatExpression array with space to create the final output
    $formatExpression = $formatExpression -join ' '

    # Output the result in the desired format
    $result = @(
        '"sourceFields": [' 
        $unmatchedFields | ForEach-Object { '    "' + $_.referenceName + '"' }
        '],'
        '"formatExpression": "' + $formatExpression + '",'
    ) -join "`n"

    return $result
}

# Fetch and process API data for both organizations
$sourceFields = Get-ApiData -organization $sourceOrganization -project $sourceProject -workItemType $workItemType -pat $sourceAccessToken
$targetFields = Get-ApiData -organization $targetOrganization -project $targetProject -workItemType $workItemType  -pat $targetAccessToken

# Compare fields from both organizations
$unmatchedFields = Compare-Fields -sourceFields $sourceFields -targetFields $targetFields
$comparisonResult = Format-ComparisonResult -unmatchedFields $unmatchedFields

# Write the output content to a new file in the same location as the script
$outputFilePath = Join-Path -Path (Split-Path -Parent $MyInvocation.MyCommand.Path) -ChildPath "output.txt"
$comparisonResult | Set-Content -Path $outputFilePath

Write-Output "------------------------------------------------------------------------"

# Output the content written to the file on the screen 
$comparisonResult | Write-Host -ForegroundColor Green

Write-Output "------------------------------------------------------------------------"

# Write a message indicating the output file path
Write-Output "Output file created at: $outputFilePath"
```

## How to Run the Script

To use this script in your Azure DevOps migration process, follow these steps:

1. **Prerequisites:**
   - Make sure you have **PowerShell** installed on your machine.
   - You will need **Personal Access Tokens (PATs)** for both your source and target Azure DevOps organizations. To create a PAT, follow the instructions from [Microsoft’s documentation](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate).

2. **Download the Script:**
   - Copy the entire PowerShell script from the blog.
   
3. **Customize the Script:**
   - Open the script file in any text editor (e.g., Notepad, Visual Studio Code).
   - Replace the placeholders for the `sourceOrganization`, `sourceProject`, `targetOrganization`, `targetProject`, and other parameters with your own Azure DevOps details.
     - `sourceOrganization`: Your source Azure DevOps organization name.
     - `sourceProject`: The project name from the source organization.
     - `sourceAccessToken`: The PAT for your source organization.
     - `targetOrganization`: Your target Azure DevOps organization name.
     - `targetProject`: The project name from the target organization.
     - `targetAccessToken`: The PAT for your target organization.
     - `workItemType`: The type of work item to compare (e.g., Bug, Task, etc.).

4. **Running the Script:**
   - Open **PowerShell**.
   - Navigate to the directory where the script is saved.
   - Execute the script by typing the following command:

     ```powershell
     .\your-script-name.ps1
     ```

5. **Review the Output:**
   - After the script finishes running, check the output displayed in your terminal. It will show the comparison results.
   - The script will also create an `output.txt` file in the **same directory**, containing the generated field mappings in the correct format for the migration tool.


## Conclusion

If you're working on Azure DevOps migrations and tired of manually comparing field definitions, this PowerShell automation can be a real time-saver. I hope it helps you as much as it helped me!

Feel free to drop questions or improvements in the comments. Happy migrating!
