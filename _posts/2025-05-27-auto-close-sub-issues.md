---
layout: post
title: "Auto-Close Sub-Issues When Parent Issue Is Closed"
date: 2025-05-27 10:00:05 +0100
categories: [DevOps & Automation, Post]
tags: [github, github-actions, sub-issues]
comments: true
---

GitHub introduced sub-issues at the beginning of 2025, allowing users to break down large issues into smaller, more manageable tasks. This hierarchy makes it easier to track progress and organize work, especially for complex projects.

I worked on projects that adopted this feature early, but found it repetitive and easy to miss manually closing each sub-issue once the parent issue was completed. To solve this problem, I automated the process and created **[Auto-Close Sub-Issues](https://github.com/marketplace/actions/close-sub-issues-automatically)** custom action. Now, when a parent issue is closed, a GitHub Actions workflow automatically closes all of its open sub-issues and posts a summary comment for visibility.

## Why Auto-Close Sub-Issues?

The benefits are simple:

- **Save time** — no need to manually close each sub-issue when the parent is done.
- **Avoid mistakes** — sub-issues won't get accidentally left open after the parent is resolved.
- **Keep your tracker accurate** — open means open, closed means closed.
- **Clean audit trail** — sub-issues are closed with `state_reason: completed` for a cleaner issue timeline.

## How It Works

The action listens to the `issues.closed` event and does the following:

1. Fetches all sub-issues of the closed parent using GitHub's Sub-Issues API, with pagination to handle any number of sub-issues.
2. Filters to only **open** sub-issues — already-closed ones are left untouched.
3. Closes each open sub-issue concurrently in chunks, setting `state_reason: completed`.
4. Posts a summary comment on the parent issue listing what was closed — and flags any failures.
5. Exposes `closed_count` and `total_count` as outputs for downstream steps.

{% include embed/video.html src='/assets/img/auto-close-sub-issue/close-sub-issues.mp4' title='Auto Close Sub-Issues in action' %}


## Getting Started

Add a workflow file at `.github/workflows/close-sub-issues.yml`:

```yaml
name: Close Sub-Issues

on:
  issues:
    types: [closed]

permissions:
  issues: write
  contents: read

jobs:
  close-sub-issues:
    runs-on: ubuntu-latest
    steps:
      - name: Close Sub-Issues
        uses: RehabAbotalep/auto-close-sub-issues@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          issue_number: ${{ github.event.issue.number }}
          repository: ${{ github.repository }}
```

That's it. No extra secrets, no configuration needed.

> The `permissions` block is required. Without `issues: write`, the action will fail with a `403 - Resource not accessible by integration` error.
{: .prompt-info }


## Configuration Options

| Input | Description | Required |
|-------|-------------|----------|
| `github_token` | GitHub token for authentication | Yes |
| `issue_number` | The issue number of the parent issue | Yes |
| `repository` | The repository in `owner/repo` format | Yes |

| Output | Description |
|--------|-------------|
| `closed_count` | Number of sub-issues successfully closed |
| `total_count` | Total number of sub-issues found (including already-closed ones) |


## What the Comment Looks Like

When the action runs successfully, it posts to the parent issue:

On success:

```
✅ Automatically closed the following sub-issues: #12 #13 #14
```

On partial failure:

```
✅ Automatically closed the following sub-issues: #12 #13

⚠️ Failed to close 1 sub-issue(s): #14
```

If all closures fail:

```
⚠️ Failed to close all sub-issue(s): #12 #13 #14
```

## Under the hood

A few implementation decisions worth noting:

- **Pagination** — the action uses `per_page: 100` and loops until all pages are exhausted, so it handles repos with many sub-issues without silently dropping any.
- **Chunked concurrency** — sub-issues are closed in parallel batches of 10 to keep throughput high without hammering the API rate limit.
- **Partial failure handling** — a single failed closure doesn't abort the rest; every sub-issue is attempted independently and failures are reported in aggregate.
- **`state_reason: completed`** — sub-issues are closed with an explicit reason rather than the default `null`, which produces a cleaner audit trail in the issue timeline.

## A notable limitation

One thing worth calling out: GitHub Actions requires a runner for every job, even lightweight ones that only make API calls.

In Azure DevOps, we often use **agentless jobs** (`pool: server`) to handle orchestration tasks — like approvals or REST API calls — without consuming compute resources. GitHub Actions doesn't currently support this, which means even simple API operations like this one spin up a runner and consume workflow minutes.

I've raised a feature request with GitHub to support agentless jobs for these scenarios: [GitHub Community Discussion #159471](https://github.com/orgs/community/discussions/159471).

## Try It Out

The action is available on the [GitHub Marketplace](https://github.com/marketplace/actions/auto-close-sub-issues). The full source is on [GitHub](https://github.com/RehabAbotalep/auto-close-sub-issues).

Give it a try, [open an issue](https://github.com/RehabAbotalep/auto-close-sub-issues/issues) if you run into anything, or submit a PR — contributions are welcome.



