---
layout: post
title:  "PowerShell Script: List Work Items Moved Between Projects in Azure DevOps"
date:   2023-09-22 07:20:05 +0100
categories: [DevOps & Automation, Script]
tags: [azure devops, powershell script, migration]
---

This PowerShell script is designed to identify and list work items that have been moved from one project to another within a given Azure DevOps organization.

## Use Case

I was working on migrating Work Items from one project to another in an Azure DevOps organization, I was using [azure-devops-migration-tools](https://github.com/nkdAgility/azure-devops-migration-tools), an awesome tool.

While using the tool, There is an issue with Work Items that have been moved from one project to another. There are invalid Area/Iteration paths in history with a different project name. 

The recommended solution is to add a mapping from old Area/Iteration paths to a new one.

The problem is the migration process pauses every time it finds an item `moved` from another project, so I needed to rerun the migration again, add mapping and repeat that for every moved item which is time-consuming.

So I created this PowerShell script that helps me to list all moved Work Items. I run the script before the migration, and if there are any moved items it lists their IDs in a txt file, I exclude these IDs from the query and run the migration.

Once the migration of all the other items is completed, I include the previously excluded moved work items in the query while mapping their old Area/Iteration paths to a new one appropriately or while setting `ReplayRevisions` to false.

![](/assets/img/list-moved-work-items/1-exclude-items.png)

![](/assets/img/list-moved-work-items/2-include-items.png)

![](/assets/img/list-moved-work-items/3-include-false-replay-revisions.png)

Here is the issues link on the GitHub repo for more details.

[This path is not anchored in the source project name](https://github.com/nkdAgility/azure-devops-migration-tools/issues?q=This+path+is+not+anchored+in+the+source+project+name)

## Script

```powershell
param (

    [Parameter(Position = 0, mandatory = $false)]
    [string]  $PAT = "PAT" ,

    [Parameter(Position = 1, mandatory = $false)]
    [string]  $orgURL = "ORGANIZATIONURL",

    [Parameter(Position = 2, mandatory = $false)]
    [string]  $projectName = "PROJECT",

    [Parameter(Position = 3, mandatory = $false)]
    [string]  $workItemType = "WORKITEMTYPE",

    [Parameter(Position = 4, mandatory = $false)]
    [string]  $filePath = "PATH"
)

$token = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes(":$($PAT)"))
$header = @{authorization = "Basic $token" }
$query = "Select [System.Id], [System.Title] From WorkItems Where [System.WorkItemType] = '$workItemType' and [System.TeamProject] = '$projectName'"
$body = @{query = $query} | ConvertTo-Json

$url = "$orgURL/$projectName/_apis/wit/wiql?api-version=7.0"

$response = Invoke-RestMethod -Uri $url -Method POST -ContentType "application/json" -Headers $header -Body $body

# Loop through each item and check revisions for changes in team project

$movedItems = @()

foreach ($item in $response.workItems) {
    
    $itemUrl = $item.url
    $itemResponse = Invoke-RestMethod -Uri $itemUrl -Method Get -Headers $header

    $currentTeamProject = $itemResponse.fields."System.TeamProject"

    $id = $item.id

    $revisionsUrl = "$orgURL/$projectName/_apis/wit/workitems/$id/revisions?api-version=7.0"
    $revisions = Invoke-RestMethod -Uri $revisionsUrl -Method Get -Headers $header

    foreach ($revision in $revisions.value) {
        $oldTeamProject = $revision.fields."System.TeamProject"
        if ($oldTeamProject -ne $currentTeamProject) {
            $movedItems += $revision.id
            $movedItems = "$movedItems, "
            Write-Host "This item: $id has been moved from $oldTeamProject to $currentTeamProject"
            break
        }
    }
}


# Check if any user stories have changed team projects
if ($movedItems.Count -eq 0) {
    Write-Host "No items have been moved from another project."
}
else {
    # Trim the , in the end
    $movedItems = $movedItems.TrimEnd(', ')
    # Output array of changed user story IDs to a file
    $outputFileName = "Moved-$workItemType.txt"
    $outputFilePath = Join-Path -Path $filePath -ChildPath $outputFileName
    $movedItems | Out-File -FilePath $outputFilePath -Encoding utf8

    # Display a message indicating where the output file was written
    Write-Host "The moved items have been written to '$outputFilePath'."
}
```

## Let's dive into the script to understand how it operates:

1. **Input Parameters:**

    The script accepts several parameters:
    - $PAT           = Personal Access token to connect on Azure DevOps
    - $orgURL        = The URL of your organization in Azure DevOps
    - $projectName   = The name of the project you want to work with
    - $workItemType  = The type of work item you need to check
    - $filePath      = The path where you want to save output file


2. **Authentication:**

    It creates an authentication token using the provided PAT and sets up the required headers for making REST API requests.

3. **Query for Work Items:**

    It constructs a query to retrieve work items of a specific type in the given project using the WIQL (Work Item Query Language).

    It sends the query to retrieve a list of work items.

4. **Check for Moved Items:**

    It loops through each retrieved work item and checks its revision history to determine if it has been moved from a different team project within the organization.

    If a work item has a history of being in different team project, it adds it to the `movedItems` list and displays a message indicating the move.

5. **Output Results:**

    If there are moved items, it writes the list to a text file named `Moved-<workItemType>.txt` in the specified file path ($filePath).

    It also displays a message indicating where the output file has been written.

6. **No Moved Items:**

    If there are no moved items, it displays a message indicating that no items have been moved.

## References

[Work Items - Get Work Item](https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/work-items/get-work-item?view=azure-devops-rest-7.1&tabs=HTTP)

[Revisions - List](https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/revisions/list?view=azure-devops-rest-7.0&tabs=HTTP)

[Wiql - Query By Wiql](https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/wiql/query-by-wiql?view=azure-devops-rest-7.0&tabs=HTTP)