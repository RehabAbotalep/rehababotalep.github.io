---
layout: post
title: "Auto-Close Sub-Issues When Parent Issue Is Closed"
date: 2025-05-27 10:00:05 +0100
categories: [DevOps & Automation, Post]
tags: [github, sub-issues]
comments: true
pin:true
---

GitHub introduced **sub-issues** at the beginning of 2025, allowing users to break down large issues into smaller, more manageable tasks. This hierarchy makes it easier to track progress and organize work, especially for complex projects.

In our organization, we actively use this feature. However, we found it repetitive to manually close each sub-issue once the parent issue is completed.

To streamline this, we automated the process: when a parent issue is closed, a GitHub Actions workflow automatically closes all of its sub-issues.

Here’s how we did it

## GitHub Workflow: Close Sub-Issues Automatically

This GitHub Actions workflow listens for the **`issues.closed`** event and performs the following:

1. Fetches all sub-issues of the closed parent.
2. Closes each sub-issue.
3. Comments on the parent issue summarizing the action.

```yaml
name: Close Sub-Issues on Parent Close

on:
  issues:
    types: [closed]

permissions:
  issues: write

jobs:
  close_sub_issues:
    runs-on: ubuntu-latest
    timeout-minutes: 5

    steps:
      - name: Get Sub-Issues
        id: get_sub_issues
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          sudo apt-get update && sudo apt-get install -y jq curl
          
          page=1
          sub_issues=()
          while :; do
            response=$(curl -s -H "Authorization: Bearer $GITHUB_TOKEN" \
              -H "Accept: application/vnd.github+json" \
              "https://api.github.com/repos/${{ github.repository }}/issues/${{ github.event.issue.number }}/sub_issues?page=$page")
            
            if [ "$(echo "$response" | jq length)" -eq 0 ]; then
              echo "No more sub-issues found on page $page."
              break
            fi

            new_sub_issues=$(echo "$response" | jq -r '.[] | select(.number) | .number')
            
            if [ -z "$new_sub_issues" ]; then
              echo "No sub-issues found on page $page."
              break
            fi

            for issue in $new_sub_issues; do
              sub_issues+=("$issue")
            done

            ((page++))
          done

          if [ ${#sub_issues[@]} -eq 0 ]; then
            echo "No sub-issues found."
            echo "HAS_SUB_ISSUES=false" >> "$GITHUB_ENV"
          else
            echo "Sub-issues found: ${sub_issues[*]}"
            echo "SUB_ISSUES<<EOF" >> "$GITHUB_ENV"
            printf "%s " "${sub_issues[@]}" >> "$GITHUB_ENV"
            echo "" >> "$GITHUB_ENV"
            echo "EOF" >> "$GITHUB_ENV"
            echo "HAS_SUB_ISSUES=true" >> "$GITHUB_ENV"
          fi

      - name: Close Sub-Issues
        if: env.HAS_SUB_ISSUES == 'true'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          closed_issues=""
          for issue_number in $SUB_ISSUES; do
            echo "Closing sub-issue #$issue_number..."

            response=$(curl -s -w "\n%{http_code}" -X PATCH -H "Authorization: Bearer $GITHUB_TOKEN" \
              -H "Accept: application/vnd.github+json" \
              -d '{"state": "closed"}' \
              "https://api.github.com/repos/${{ github.repository }}/issues/$issue_number")

            http_code=$(echo "$response" | tail -n1)
            body=$(echo "$response" | sed '$d')

            if [ "$http_code" -eq 200 ]; then
              echo "Sub-issue #$issue_number closed successfully."
              closed_issues="$closed_issues #$issue_number"
            else
              echo "Failed to close sub-issue #$issue_number (HTTP $http_code): $body"
            fi
          done

          echo "CLOSED_ISSUES<<EOF" >> "$GITHUB_ENV"
          echo "$closed_issues" >> "$GITHUB_ENV"
          echo "EOF" >> "$GITHUB_ENV"

      - name: Comment on Parent Issue
        if: env.HAS_SUB_ISSUES == 'true' && env.CLOSED_ISSUES != ''
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          if [ -n "$CLOSED_ISSUES" ]; then
            body="✅ Automatically closed the following sub-issues:$CLOSED_ISSUES"
            curl -s -X POST -H "Authorization: Bearer $GITHUB_TOKEN" \
              -H "Accept: application/vnd.github+json" \
              -d "$(jq -nc --arg body "$body" '{"body": $body}')" \
              "https://api.github.com/repos/${{ github.repository }}/issues/${{ github.event.issue.number }}/comments"
          fi
```

## How to Use It

1. **Create a new workflow file**  
   In your repository, create a new file under `.github/workflows/`, for example: `.github/workflows/close-sub-issues.yml`

2. **Paste the workflow code**  
Copy and paste the full workflow YAML content into this file.

3. **Commit the file**  
Commit and push it to your repository.

Now, whenever a parent issue is closed, all of its sub-issues will automatically be closed as well.

![Auto Close Sub-Issues](/assets/img/auto-close-sub-issue/close-sub-issues.gif)

## Limitations and Investigation

Notable limitation of this GitHub Actions workflow is the **requirement of a runner for every job**, even if the task is lightweight and only involves making API calls.

In **Azure DevOps**, we often use **agentless jobs** (via `pool: server`) to handle such orchestration tasks—like approvals, REST API calls without consuming a runner or compute resources.

Unfortunately, **GitHub Actions does not currently support agentless jobs**. This means:

- A runner must be provisioned and spun up even for simple API operations.
- This consumes **workflow minutes**, even for non-compute-heavy tasks.

We've raised a **feature request** with GitHub to support **agentless jobs** for such scenarios:
[GitHub Community Discussion #159471](https://github.com/orgs/community/discussions/159471)

We hope GitHub considers adding this feature to improve cost-efficiency and better align with other CI/CD platforms.



