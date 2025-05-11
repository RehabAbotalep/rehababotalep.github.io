---
layout: post
title: "How GitHub Handles Attachments in Issues, PRs, Discussions, and Wikis"
date: 2025-05-11 08:00:05 +0100
categories: [DevOps & Automation, Post]
tags: [github]
comments: true
---

I’ve been working extensively on GitHub Discussions and frequently need to upload images. During this process, I came across some questions: Where exactly are these images stored, and why do I see different links one in the discussion's markdown and another when clicking on the image?
To understand more, I did some investigation, and in this blog post, I share my findings.

This blog post explains how attachments (such as images, logs, or other files) work when added to GitHub Issues, Pull Requests (PRs), Discussions, and Wikis.

## Where Are Attachments Stored?

When a file is uploaded to a GitHub Issue, Pull Request, or Discussion, it is not stored inside the Git repository. Instead, GitHub stores these attachments on its **Content Delivery Network (CDN)**. 

This separation ensures that attachments do not impact your repository’s Git history or size limits.

## What Is a CDN?

A Content Delivery Network, is a geographically distributed network of servers designed to deliver content quickly and reliably to users.

This has several benefits:
- Faster load times for end users
- Reduced load on GitHub's main servers
- High availability of content

## URL Formats for Accessing Attachments

GitHub uses different URL formats for accessing attachments, depending on whether the repository is public or private. Here are the different formats:

### 1. **`https://github.com/user-attachments/assets/<unique-id>`**

This URL format is used internally by GitHub to reference and manage assets uploaded to Issues, PRs, or Discussions. When you upload an image (via drag-and-drop or file upload), GitHub generates a URL in this format, and you’ll see this link automatically inserted into the markdown of the Issue or PR.

- **Usage**: Internal reference in the GitHub UI. This URL is used to track the file within GitHub’s system and is inserted into the markdown when you upload an image or other files.
- **Access**: Not typically accessible outside GitHub’s system, but can be referenced internally. If the repository is private, only authenticated users with access to the repository can view the image. If the repository is public, anyone with the URL can access it.
- **Example**: `https://github.com/user-attachments/assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

### 2. **`https://user-images.githubusercontent.com/<unique-id>`**

This URL format is used for **publicly accessible** attachments uploaded to **public repositories**.

- **Usage**: Public URL for accessing images, videos, or other media uploaded to public repositories.
- **Access**: Anyone with the URL can access the file, even without authentication.
- **Example**: `https://user-images.githubusercontent.com/xxxxxxxx/xxxxxxxxx-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

### 3. **`https://private-user-images.githubusercontent.com/<unique-id>`**

This URL format is used for **privately accessible** attachments uploaded to **private repositories**.

- **Usage**: Private URL for accessing images, videos, or other media uploaded to private repositories.
- **Access**: Only users with **authenticated access** to the private repository can view the file.
- **Example**: `https://private-user-images.githubusercontent.com/xxxxxxxx/xxxxxxxxx-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

## Do Attachments Count Toward Repository Size?

No, attachments uploaded to Issues, Pull Requests, Discussions, or Wikis do not count toward the **100 GB** storage limit for the Git repository. They are stored separately in GitHub's CDN and are not part of the Git history or LFS (Large File Storage) system.

## Size Limits

### Per File Upload Limit
- 10MB for images and gifs
- 10MB for videos uploaded to a repository owned by a user or organization on a free GitHub plan
- 100MB for videos uploaded to a repository owned by a user or organization on a paid GitHub plan
- 25MB for all other files

## Access Control

### For Private Repositories:
- Attachments uploaded after recent changes are now **protected** and can only be accessed by users who are authenticated and have access to the repository.
- Email notifications sent from private repositories will no longer display images; each image is replaced by a link to view it on the web.

### For Public Repositories:
- Attachments uploaded to public repositories are **publicly accessible**. Anyone with the link can view the files, even without authentication.
- This applies to both existing and new attachments, meaning the files are accessible to anyone who knows the URL, without the need for a GitHub account.

## Summary

| Feature | Behavior |
|--------|----------|
| Storage location | GitHub CDN (`user-images.githubusercontent.com`) |
| Repository size impact | No impact |
| Max file size per upload | 25 MB |
| Access control for private repos | Login required for new attachments |
| Access control (public repos)   | Publicly accessible via direct URL   

## Links
- [Attaching files](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/attaching-files)
- [More secure private attachments](https://github.blog/changelog/2023-05-08-more-secure-private-attachments/)
