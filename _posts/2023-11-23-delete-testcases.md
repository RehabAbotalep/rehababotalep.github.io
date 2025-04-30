---
layout: post
title:  "PowerShell Script: Automate Test Cases Deletion in Azure DevOps"
date:   2023-11-23 07:35:05 +0100
categories: [DevOps & Automation, Script]
tags: [azure devops, test cases, powershell]
---

While working on a particular use case, I encountered a requirement to delete all Test Cases within my Azure DevOps project. Unfortunately, the user interface (UI) does not provide an option to accomplish this task. Upon exploring the available API options, I discovered that there is no direct API for deleting multiple Test Cases at once; only one Test Case can be deleted at a time.

To address this limitation, I developed a PowerShell script to streamline the process. The script is designed to retrieve Test Cases based on a specific query, iterate through them, and invoke the deletion API for each Test Case. This approach ensures an automated and efficient process for mass deletion, overcoming the absence of a built-in feature in the UI.

![](/assets/img/delete-testcases/multi-testcases-options.png)

## Script

```powershell
# Variables (Update these as needed)
$organizationUrl = "ORGANIZATION_URL"
$projectName = "PROJECT_NAME"
$pat = "PAT"

function New-AuthorizationHeader {
    param (
        [string]$Token
    )

    $headerValue = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes(":$($Token)"))
    return @{ Authorization = "Basic $headerValue" }
}

function Get-AzureDevOpsTestCases {
    param (
        [string]$OrganizationUrl,
        [string]$ProjectName,
        [string]$Pat
    )

    $header = New-AuthorizationHeader -Token $Pat

    $query = "Select [System.Id] From WorkItems Where [System.WorkItemType] = 'Test Case' and [System.TeamProject] = '$ProjectName'"
    $body = @{ query = $query } | ConvertTo-Json
    $url = "$OrganizationUrl/$ProjectName/_apis/wit/wiql?api-version=7.1"

    try {
        $response = Invoke-RestMethod -Uri $url -Method POST -ContentType "application/json" -Headers $header -Body $body
        return $response.workItems
    } catch {
        $Host.UI.WriteErrorLine("Error retrieving test cases: $_")
    }
}

function Delete-AzureDevOpsTestCase {
    param (
        [string]$OrganizationUrl,
        [string]$ProjectName,
        [string]$Pat,
        [string]$TestCaseId
    )

    $header = New-AuthorizationHeader -Token $Pat
    $uri = "$OrganizationUrl/$ProjectName/_apis/test/testcases/$TestCaseId/?api-version=7.1"

    try {
        Write-Host "Deleting Test Case $TestCaseId"
        Invoke-RestMethod -Uri $uri -Method DELETE -ContentType "application/json" -Headers $header
        Write-Host "Deleted Test Case: $TestCaseId"
    } catch {
        $Host.UI.WriteErrorLine("Error deleting test case ${TestCaseId}: $_")
    }
}


function Remove-AllAzureDevOpsTestCases {
    param (
        [string]$OrganizationUrl,
        [string]$ProjectName,
        [string]$Pat
    )

    $testCases = Get-AzureDevOpsTestCases -OrganizationUrl $OrganizationUrl -ProjectName $ProjectName -Pat $Pat

    foreach ($testCase in $testCases) {
        $testCaseId = $testCase.id
        Delete-AzureDevOpsTestCase -OrganizationUrl $OrganizationUrl -ProjectName $ProjectName -Pat $Pat -TestCaseId $testCaseId
    }

    Write-Host "All test cases deleted."
}
```

# Remove all test cases
Remove-AllAzureDevOpsTestCases -OrganizationUrl $organizationUrl -ProjectName $projectName -Pat $pat

## Dive into the script

**1. Variable Initialization:**

- $organizationUrl: The URL of the Azure DevOps organization.
- $projectName: The name of the Azure DevOps project where the test cases are located.
- $pat: Personal Access Token (PAT) used for authentication and authorization.
<br>

**2. Functions:**

- **New-AuthorizationHeader:** This function takes a token as a parameter and returns an authorization header. It uses Basic authentication with the provided token.

- **Get-AzureDevOpsTestCases:** This function retrieves test cases from Azure DevOps based on the specified organization URL, project name, and PAT. It uses a `WIQL` (Work Item Query Language) query to select test cases and returns the list of work items.
    You can modify the `$query` variable to meet your specific filtering requirements, ensuring the returned list aligns precisely with your needs.

- **Delete-AzureDevOpsTestCase:** This function deletes a specific test case identified by its ID. It takes the organization URL, project name, PAT, and the ID of the test case to be deleted as parameters.

- **Remove-AllAzureDevOpsTestCases:** This function retrieves all test cases using `Get-AzureDevOpsTestCases` and then iterates through each one, deleting it using `Delete-AzureDevOpsTestCase`. Finally, it prints a message indicating that all test cases have been deleted.

**3. Execution:**

The last section of the script calls `Remove-AllAzureDevOpsTestCases` with the specified organization URL, project name, and PAT. This triggers the removal of all test cases in the specified Azure DevOps project.

> Ensure that you have the necessary permissions in Azure DevOps to query and delete test cases.
{: .prompt-info }

> Make sure to replace the placeholder values for $organizationUrl, $projectName, and $pat with your actual Azure DevOps organization URL, project name, and PAT before running the script.
{: .prompt-info }

> Always be cautious when running scripts, especially those that modify or delete data. Test the script in a safe environment before using it in a production setting.
{: .prompt-danger }

## References

[Wiql - Query By Wiql](https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/wiql/query-by-wiql?view=azure-devops-rest-7.0&tabs=HTTP)

[Test Cases - Delete](https://learn.microsoft.com/en-us/rest/api/azure/devops/test/test-cases/delete?view=azure-devops-rest-7.1&tabs=HTTP)

[Delete test artifacts in Azure Boards](https://learn.microsoft.com/en-us/azure/devops/boards/backlogs/delete-test-artifacts?view=azure-devops#delete-a-test-case-test-suite-or-test-plan)

