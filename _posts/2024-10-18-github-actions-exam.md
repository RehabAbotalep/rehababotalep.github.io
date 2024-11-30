---
layout: post
title:  "How to Pass GitHub Actions Certification Exam: My Own Certification Journey"
date:   2024-10-18 02:25:05 +0100
categories: [Blogging, Post]
tags: [github, certifications]
pin: true
comments: true
---

Earlier this year (2024), GitHub shared some exciting news: certifications that were once only for their employees and partners are now open to everyone worldwide!

These certifications cover different skills and knowledge areas related to GitHub:

- GitHub Foundations
- GitHub Actions
- GitHub Advanced Security
- GitHub Administration 

![GitHub Certification Exams](/assets/img/github-actions-exam/exams.png)

In this blog post, I'll be sharing my experience with the GitHub Actions exam, detailing how to schedule, prepare for, and successfully pass it.

## About GitHub Actions Certification

According to GitHub, This exam is designed for DevOps engineers, software developers, and IT professionals with intermediate level experience in GitHub Actions, including workflow creation, automation, and CI/CD pipeline
management.

It covers the following domains:

- Domain 1: Author and maintain workflows (40%)
- Domain 2: Consume workflows (20%)
- Domain 3: Author and maintain actions (25%)
- Domain 4: Manage GitHub Actions for the enterprise (15%)

For more details, checkout [Study Guide for GitHub Actions](https://assets.ctfassets.net/wfutmusr1t3h/2mMJ6nECbUAdiQMTObbPw6/275fd49301e7a0d1b4e3de0bfe95d765/github-actions-exam-preparation-study-guide__1_.pdf).

## Scheduling the Exam

To register and schedule the exam, visit the [Official Exam Website](https://examregistration.github.com/login?ReturnUrl=%2Foverview) and log in with your GitHub account. Then, select the GitHub Actions Exam.

Follow these registration steps:

1. Choose your test option (Online or On-site).
2. Provide your candidate details.
3. Select your desired date and time for the exam.
4. Complete the payment process. The exam costs $99.
5. Review your registration details.

![Registration Steps](/assets/img/github-actions-exam/registration-steps.png)

Once you've completed these steps, you'll receive an email from PSI Online containing your exam date, time, and important instructions. Make sure to read through them carefully.

## Preparing for the Exam

Preparation is key for exams and certifications. It includes planning, understanding the exam content, and dedicating focused study time. Remember to **Avoid Relying on Exam Dumps**, as they don't guarantee success. Embrace the process of studying and expanding your knowledge.

For effective preparation, leverage resources such as the [GitHub Actions learning path on MS Learn](https://learn.microsoft.com/en-us/collections/n5p4a5z7keznp5) and the official [GitHub Docs](https://docs.github.com/en), along with other learning and training sources.

Personally, I found both the MS Learning path and GitHub documentation to be valuable resources for my preparation.

For quick reference, below is a list of exam topics along with the corresponding links to relevant GitHub documentation:

### Domain 1: Introduction to Git and GitHub 

#### Work with events that trigger workflows

- [Configure workflows to run for one or more events](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows)
- [Configure workflows to run for scheduled events](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#schedule)
- [Configure workflows to run for manual events](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#workflow_dispatch)
- [Configure workflows to run for webhook events](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#workflow_run)

#### Use the components of a workflow

- [Describe how actions, workflows, jobs, steps, runs, and the marketplace work together](https://docs.github.com/en/actions/writing-workflows/about-workflows)
- [Identify the correct syntax for workflow jobs](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#jobs)
- [Use job steps for actions and shell commands](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/workflow-commands-for-github-actions)
- [Use conditional keywords for steps](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/using-conditions-to-control-job-execution)
- [GitHub-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners)
- [GitHub-self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
- [Demonstrate the use of dependent jobs](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/using-jobs-in-a-workflow)

#### Use encrypted secrets and environment variables as part of a workflow

- [Use encrypted secrets to store sensitive information](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)
- [Identify the available default environment variables during the construction of the workflow](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/store-information-in-variables#default-environment-variables)
- [Identify the location to set custom environment variables in a workflow](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/store-information-in-variables#defining-environment-variables-for-a-single-workflow)
- [Identify when to use the GITHUB_TOKEN secret](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)
- [Demonstrate how to use workflow commands to set environment variables](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/workflow-commands-for-github-actions#setting-an-environment-variable)

#### Create a workflow for a particular purpose

- [Add a script to a workflow](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/adding-scripts-to-your-workflow)
- [Demonstrate how to publish to GitHub Packages using a workflow](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions)
- [Demonstrate how to publish to GitHub Container Registry using a workflow](https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-docker-images)
- [Use database and service containers in a GitHub Actions workflow](https://docs.github.com/en/actions/use-cases-and-examples/using-containerized-services/about-service-containers)
- [Use labels to route workflows to specific runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/using-labels-with-self-hosted-runners?learn=hosting_your_own_runners&learnProduct=actions)
- [Use CodeQL as a step in a workflow](https://docs.github.com/en/enterprise-cloud@latest/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/running-codeql-code-scanning-in-a-container)
- [Deploy a release to a cloud provider using a GitHub Actions workflow](https://docs.github.com/en/actions/use-cases-and-examples/deploying/deploying-with-github-actions)

#### Manage workflow runs

- [Configure caching of workflow dependencies](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows)
- [Identify steps to pass data between jobs in a workflow](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/storing-and-sharing-data-from-a-workflow)
- [Remove workflow artifacts from GitHub](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/removing-workflow-artifacts)
- [Add a workflow status badge](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/monitoring-workflows/adding-a-workflow-status-badge)
- [Add environment protections](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-deployments/configuring-custom-deployment-protection-rules)
- [Define a matrix of different job configurations](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/running-variations-of-jobs-in-a-workflow)
- [Implement workflow approval gates](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-deployments/reviewing-deployments)

### Domain 2: Consume workflows

#### Interpret the effects of a workflow

- [Diagnose a failed workflow run](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/monitoring-workflows/using-workflow-run-logs#viewing-logs-to-diagnose-failures)
- [Identify ways to access the workflow logs from the user interface](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/monitoring-workflows/using-workflow-run-logs#searching-logs)
- [Identify ways to access the workflow logs from GitHub’s REST API](https://docs.github.com/en/rest/actions/workflow-runs?apiVersion=2022-11-28)
- [Enable step debug logging in a workflow](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/troubleshooting-workflows/enabling-debug-logging)

#### Locate a workflow, its logs, and artifacts

- [Describe where to locate a workflow in a repository](https://docs.github.com/en/actions/about-github-actions/understanding-github-actions#workflows)
- [Explain the difference between disabling and deleting of workflows](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/disabling-and-enabling-a-workflow)
- [Demonstrate how to download workflow artifacts from the user interface](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/downloading-workflow-artifacts)
- [Describe how to use an organization’s templated workflow](https://docs.github.com/en/actions/sharing-automations/creating-workflow-templates-for-your-organization)

#### Use actions in a workflow

- [Identify an action’s type, inputs, and outputs](https://docs.github.com/en/enterprise-cloud@latest/actions/writing-workflows/workflow-syntax-for-github-actions)
- [Demonstrate how to use the specific version of an action in a workflow](https://docs.github.com/en/actions/sharing-automations/creating-actions/about-custom-actions#using-release-management-for-actions)

### Domain 3: Author and maintain actions

#### Use available action types

- [Identify the type of action required for a given problem](https://docs.github.com/en/actions/sharing-automations/creating-actions/about-custom-actions)
- [Demonstrate how to troubleshoot JavaScript actions](https://docs.github.com/en/actions/sharing-automations/creating-actions/creating-a-java)
- [Demonstrate how to troubleshoot Docker container actions](https://docs.github.com/en/actions/sharing-automations/creating-actions/creating-a-docker-container-action)
- [Creating a composite action](https://docs.github.com/en/actions/sharing-automations/creating-actions/creating-a-composite-action)
- [Identify the metadata and syntax needed to create an action](https://docs.github.com/en/actions/sharing-automations/creating-actions/metadata-syntax-for-github-actions)

#### Distribute an action

- [Demonstrate how to create a release strategy for an action](https://docs.github.com/en/actions/sharing-automations/creating-actions/releasing-and-maintaining-actions)
- [Demonstrate how to publish an action to the GitHub marketplace](https://docs.github.com/en/actions/sharing-automations/creating-actions/publishing-actions-in-github-marketplace)

### Domain 4: Manage GitHub Actions in the enterprise

#### Distribute actions and workflows to the enterprise

- [Explain reuse templates for actions and workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows)
- [Creating workflow templates for your organization](https://docs.github.com/en/actions/sharing-automations/creating-workflow-templates-for-your-organization)
- [Define how to distribute actions for an enterprise](https://docs.github.com/en/enterprise-server@3.10/admin/managing-github-actions-for-your-enterprise/managing-access-to-actions-from-githubcom/about-using-actions-in-your-enterprise)
- [Configure organizational use policies for GitHub Actions](https://docs.github.com/en/enterprise-cloud@latest/admin/enforcing-policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-github-actions-in-your-enterprise)
- [Disabling or limiting GitHub Actions for your organization](https://docs.github.com/en/organizations/managing-organization-settings/disabling-or-limiting-github-actions-for-your-organization)

#### Manage runners for the enterprise

- [Describe the effects of configuring IP allow lists on GitHub-hosted and self-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners/about-github-hosted-runners)
- [Explain the difference between GitHub-hosted and self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners#differences-between-github-hosted-and-self-hosted-runners)
- [Configure self-hosted runners for enterprise use (i.e. including proxies, labels, networking)](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/using-a-proxy-server-with-self-hosted-runners)
- [Demonstrate how to manage self-hosted runners using groups (i.e. managing access, moving runners into and between groups)](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/managing-access-to-self-hosted-runners-using-groups)
- [Demonstrate how to monitor, troubleshoot, and update self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/monitoring-and-troubleshooting-self-hosted-runners)

#### Manage encrypted secrets in the enterprise

- [Identify the scope of encrypted secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)
- [Demonstrate how to access encrypted secrets within actions and workflows](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-an-environment)
- [Explain how to manage organization-level encrypted secrets](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-an-organization)
- [Explain how to manage repository-level encrypted secrets](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository)

## Hands-On

After getting a solid foundation of GitHub Actions, it's really important to follow it with hands-on exercises.

To help with that, I created the following repository in GitHub that has a collection of GitHub Actions and workflows to demonstrate various features and capabilities of GitHub Actions. It includes examples of workflows, composite actions, Docker-based actions, and JavaScript-based actions.

Check it out: [learning-github-actions](https://github.com/RehabAbotalep/learning-github-actions)

![Repo](/assets/img/github-actions-exam/github-actions-repo.png)

Additionally, I’ve written a blog post that outlines a step by step guide to build and deploy an .Net Core application to an Azure App Service using GitHub Actions.

Check it out: [GitHub Actions: Deploying .NET to Azure App Service](https://rehababotalep.github.io/posts/delpoy-.net-to-azure-app-service/)

![Deploying .NET to Azure App Service](/assets/img/github-actions-deploy-dotnet/workflow.gif)

## Taking the Exam (Proctored at Home)

It's important to sign in up to 30 minutes before your test start time as per the instructions provided when booking your exam. Ensure you adhere to the technical requirements and have a valid, current ID ready. The name on this ID must be in Roman (English) alphabet characters and must match the name used when scheduling the exam.

![](/assets/img/github-foundation-exam/legal-name.png)

> Make sure to carefully read the instructions sent to you when you booked the exams.
{: .prompt-info }

The exam has around 75 questions, and you have 120 minutes to finish it. Once you're done, you'll know right away if you passed or failed.
 

## After The Exam

Immediately after completing your exam, you'll receive an email containing a detailed score report. This report will outline your overall score and the percentage scores for each exam module. It will help you pinpoint areas of strength and areas that may require improvement.

Additionally, you'll receive a Credly badge that you can share on your social media profiles such as LinkedIn, Twitter, your website, and other platforms. This badge serves as recognition of your achievement and allows you to showcase your skills to others.


> If you are preparing for GitHub Foundation Exam, check out this post [How to Pass GitHub Foundations Certification Exam: My Own Certification Journey](https://rehababotalep.github.io/posts/github-foundation-exam/)
{: .prompt-tip }





