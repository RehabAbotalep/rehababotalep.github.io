---
layout: post
title: "Restrict Work Item Access to Azure DevOps Team Members"
date: 2025-05-30 10:00:05 +0100
categories: [DevOps & Automation, Post]
tags: [azure devops]
comments: true
---

In Azure DevOps, **all project members can access work items by default**, regardless of their team membership. To secure team work items and restrict access to only specific users (typically team members), you can choose one of the following two approaches:

## Option 1: Use a Custom Group to Explicitly **Deny Access**

This approach uses the **“Deny overrides Allow”** principle. In Azure DevOps, if a user is denied a permission at any level, that denial takes precedence over any allowed permissions elsewhere.

### Steps:

1. Go to **Project Settings > Permissions**, then click **New Group**.

   ![Project Settings](/assets/img/restrict-work-items-access/1-settings.png)

2. Name the group (e.g., `NonTeamMembers - Deny Access`), add users you want to restrict, and click **Create**.

   ![Create Group](/assets/img/restrict-work-items-access/2-create-group.png)

3. Navigate to **Project Settings > Areas**, select the desired team area, click the **ellipses (⋯)** and choose **Security**.

   ![Area Settings](/assets/img/restrict-work-items-access/3-area-settings.png)

4. Search for the custom group you created. Initially, all permissions are **Not set**.

   ![Initial Permissions](/assets/img/restrict-work-items-access/4-initial-permissions.png)

5. Users in the project even those outside the team, can still view team work items unless explicitly denied.

   ![Visible Work Items](/assets/img/restrict-work-items-access/5-visible-work-items.png)

6. Set the required permissions (e.g., **View work items in this node**) to **Deny** for the group.

   ![Set Deny](/assets/img/restrict-work-items-access/6-set-deny.png)

7. Now, these users will be restricted from seeing the work items in this area.

   ![Restricted View](/assets/img/restrict-work-items-access/7-restricted-view.png)


## Option 2: Remove Inherited Groups and Grant Access Only to Team Members

Instead of managing denials, this method removes default groups (like Contributors or Readers) from the area and assigns permissions **only** to the team members you want to allow access.

### Steps:

1. Navigate to the team area security panel (same as in Step 3 above), select the **Contributors** group, and click **Remove**.

   ![Delete Contributors](/assets/img/restrict-work-items-access/8-delete-contributors.png)

2. Users in this group will lose access to work items after removal.

   ![Visible Before Removal](/assets/img/restrict-work-items-access/9-visible-before-removal.png)

3. Repeat this for the **Readers** group if they are listed.

   ![Delete Readers](/assets/img/restrict-work-items-access/10-delete-readers.png)

4. Now only explicitly granted users will have access.

   ![Restricted Access](/assets/img/restrict-work-items-access/11-restricted-access.png)

5. Finally, add your team members individually or through a dedicated group and **set permissions to Allow**.

   ![Allow Access to Members](/assets/img/restrict-work-items-access/12-allow-access-members.png)
