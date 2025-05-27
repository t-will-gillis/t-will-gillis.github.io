---
layout: post
title: GHA Schedule Monthly Workflow
date: 2025-05-13 13:43 -0700
description: GitHub Actions Workflow for Identity and Access Management
image: 
  path: ../assets/img/site_images/schedule_monthly_gha/schedule_monthly_0.png
  alt: Schedule Monthly Workflow
category: [Automations]
tags: [github actions, workflows, google apps script, yml, yaml, javascript]
---

<figure>
  <img src="../assets/img/site_images/schedule_monthly/base_template_flow_only.png" alt="" title='Flow diagram of "Schedule Monthly" workflow' style="max-width:90%; margin:auto; box-shadow: 4px 4px 8px rgba(192,192,192,0.5);">
  <figcaption style="font: italic small sans-serif; text-align:center">Flow diagram of "Schedule Monthly" workflow</figcaption>
</figure>

# Schedule Monthly &<br>WR Schedule Monthly<br><sub>Workflows</sub>

## Summary
The `schedule-monthly.yml` and `wr-schedule-monthly.yml` workflows together are intended to monitor the individual activities of each member of the 'website-write' team. Members that have been inactive (as described below) for over two months are removed from the 'website-write' team. Members that have been inactive for over one month (and shy of two months) are notified that the bot will remove their team membership in the next month if the member does not resume activity. Specific details and functionalities are described below. 

### Trigger
- Workflow runs at 11:00 UTC/ 4:00 PDT, on the 1st day of every month (except January and August)

### Labels Used by Workflow
- labelKey: 'complexity2', `Complexity: Small`
- labelKey: 'size025pt',`size: 0.25pt`
- labelKey: 'readyForDevLead',`ready for dev lead` 
- labelKey: 'roleDevLeads',`role: dev leads` 
- labelKey: 'featureAdministrative',`Feature: Administrative` 


### Tokens Used by Workflow
- `HACKFORLA_ADMIN_TOKEN` ( used in schedule-monthly.yml )
- `HACKFORLA_BOT_PA_TOKEN` ( used in wr-schedule-monthly.yml )

### Workflow Files

- [schedule-monthly.yml](https://github.com/hackforla/website/blob/gh-pages/.github/workflows/schedule-monthly.yml)
  - [get-contributors-data.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/trigger-schedule/list-inactive-members/get-contributors-data.js) (in list-inactive-members folder)
    - [get-timeline.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/get-timeline.js) (in utils folder)
    - [get-team-members.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/get-team-members.js) (in utils folder)
  - [trim-inactive-members.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/trigger-schedule/list-inactive-members/trim-inactive-members.js) (in list-inactive-members folder)
    - [get-team-members.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/get-team-members.js) (in utils folder)
    - [add-team-member.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/add-team-member.js) (in utils folder)
    - [inactive-members.json](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/_data/inactive-members.json) (in utils/data folder)
- [wr-schedule-monthly.yml](https://github.com/hackforla/website/blob/gh-pages/.github/workflows/wr-schedule-monthly.yml)
  - [create-new-issue.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/trigger-schedule/list-inactive-members/create-new-issue.js) (in list-inactive-members folder)
    - [issue-template-parser.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/issue-template-parser.js) (in utils folder)
    - [inactive-members.md](https://github.com/hackforla/website/blob/gh-pages/github-actions/trigger-schedule/list-inactive-members/inactive-members.md) (in list-inactive-members folder)
    - [post-issue-comment.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/post-issue-comment.js) (in utils folder)
 
#### Support File Folders: 
- [.github/workflows](https://github.com/hackforla/website/blob/gh-pages/.github/workflows)
- [github-actions/trigger-schedule/list-inactive-members](https://github.com/hackforla/website/tree/gh-pages/github-actions/trigger-schedule/list-inactive-members)  
- [github-actions/utils](https://github.com/hackforla/website/tree/gh-pages/github-actions/utils)


### Process:
- The [schedule-monthly.yml](https://github.com/hackforla/website/blob/gh-pages/.github/workflows/schedule-monthly.yml) workflow is triggered on a cron schedule at 11:00 UTC / 4:00 am PDT on the first day of every month, except January and August. (January and August are omitted because the 'website' team takes off December and July. The time limits discussed below are adjusted to account for this expected inactivity during the break.)
- This workflow consists of the job "Trim_Contributors" and three main steps:
  - "Get Contributors" calls [get-contributors-data.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/trigger-schedule/list-inactive-members/get-contributors-data.js):
    - The function `fetchContributors()` queries GitHub for data about all user contributions to the 'hackforla/website' repo:  
      - If a user made any contribution  due to 1. commits, 2. comments, or 3. issue assignments.
      - All user contributions within the last month (first run, `allContributorsSinceOneMonthAgo`) and within the last two months (second run, `allContributorsSinceTwoMonthsAgo`) are recorded.
      - Note that members of the 'website-admin' team as well as three Hack for LA bot accounts are considered 'permanent contributors' and are automatically included in these two lists regardless of contributions. 
      - This function also records any open issue whose assignee does not show any activity within the last two months, and whether the issue is a "Pre-work Checklist"- `inactiveWithOpenIssue`. 
  - "Trim Inactive Members" calls [trim-inactive-members.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/trigger-schedule/list-inactive-members/trim-inactive-members.js):
    - The function `readPreviousNotifyList()` retrieves the list of 'notified members' from the previous month `previouslyNotified`.
    - The function `getTeamMembers()` records all current 'website-write' team members `currentTeamMembers`.
    - The function `removeInactiveMembers()` iterates through the list of current team members and checks whether the member is listed on the `allContributorsSinceTwoMonthsAgo` list. If the team member does not show any activity in the last two months, the function then checks:
      - whether the team member is listed on the 'website' team; if not the person is added. (We want to make sure that the member is on the 'website' team before they are removed from 'website-write' team),
      - whether the team member is on the list of `inactiveWithOpenIssue` and the issue is not a "Pre-work Checklist", if so their name and issue are added to `cannotRemoveYet`. (We don't want to remove the person with an open assignment)
      - whether or not the team member was notified of their inactivity in the last month, i.e. whether their name is on the `previouslyNotified` list. If not, they will not be removed in the current month. (We want to give people a notification prior to removing them from the 'website-write' team) 
      - Else, the team member will be removed from both the 'website-write' and 'website-merge' teams if applicable.
      - If the member that was just removed has an open "Pre-work Checklist", this issue is closed by the bot vis `closePrework()`.
    - The function `getTeamMembers()` then runs a second time to update the list of `updatedTeamMembers`.
    - The function `notifyInactiveMembers()` iterates through the list of updated team members and checks whether the member is listed on the `allContributorsSinceOneMonthAgo` list. If the member does not show any activity within the last month, the function checks:
      - whether the member cloned HfLA's repo within last month. If so, then the member might be new and are still setting up, and they will not be notified of inactivity yet.
      - otherwise, the member is added to the list of members to be notified.
  - "Update Inactive Members JSON" uses `stefanzweifel/git-auto-commit-action@v5.0.1` to commit the record of `inactive-members.json` to the repo.
- The post workflow file [wr-schedule-monthly.yml](https://github.com/hackforla/website/blob/gh-pages/.github/workflows/wr-schedule-monthly.yml) runs if `schedule-monthly.yml` is successful and includes:
  - The job "Create-New-Issue" and related Step call [create-new-issue.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/trigger-schedule/list-inactive-members/create-new-issue.js):
    - The function `createIssue()` writes the lists of removed members and members to be notified to a template [inactive-members.md](https://github.com/hackforla/website/blob/gh-pages/github-actions/trigger-schedule/list-inactive-members/inactive-members.md) 
    - The function `postComment()` posts a comment to the Monday Dev Meeting Agenda issue [#2607](https://github.com/hackforla/website/issues/2607#issuecomment-2029144743), informing that the workflow has run, linking to the issue that was created, and if applicable listing members with open issues (and issue number) [post-issue-comment.js module](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/post-issue-comment.js) module
  - The job "Close-New-Issue" gathers the pertinent data and then as a last step closes the just-generated "Review Inactive Team Members" issue. 

Data auto-generated by Google Sheets worksheets (maintained in the "hackforla-bot@hackforla.org" account) complement this workflow:
- [Current 'website-write' team](https://docs.google.com/spreadsheets/d/11u71eT-rZTKvVP8Yj_1rKxf2V45GCaFz4AXA7tS_asM/edit#gid=1432079772)  
- [Current Google Drive (Website) Registrants](https://docs.google.com/spreadsheets/d/11u71eT-rZTKvVP8Yj_1rKxf2V45GCaFz4AXA7tS_asM/edit#gid=653104171) 

## Test Procedure
Important note: line numbers and specific code references may have changed slightly since the time of this Wiki- verify with the actual files.
- You will need to have a functioning test environment on your local repo. Refer to [Hack for LA's GitHub Actions](https://github.com/hackforla/website/issues/6537#issuecomment-2041147335), especially Tips 6, 7, & 8.
- IMPORTANT: In addition to the 'files changed' in the PR, there are additional changes that you should make to help with testing. If you do not make these edits, you might delete or mis-edit Hack for LA data, delete team members, generate false or junk notifications, and/or create junk issues on HfLA's live Project Board. 
- In `schedule-monthly.yml`, you will need to activate two personal tokens (see [Hack for LA's GitHub Actions](https://github.com/hackforla/website/issues/6537#issuecomment-2041147335))
  - `HACKFORLA_BOT_PA_TOKEN` scopes: admin:org_hook, public_repo
  - `HACKFORLA_ADMIN_TOKEN` scopes: admin:org_hook, repo, write:org
  - In the same file, around line 12, replace 'hackforla' with your personal repo.
  - Finally, add the line `workflow_dispatch:` under `on:` (in line with `schedule:`)
- In `get-contributors-data.js`
  - Near line 84 replace with `owner: 'hackforla',`
  - Near line 85 replace with `repo: 'website',` (unless your repo is 'website')
- In `trim-inactive-members.js`:
  - IMPORTANT:  Disable the following by replacing:  
    ```
    await github.request('DELETE /orgs/{org}/teams/{team_slug}/memberships/{username}', {
              org: context.repo.owner,
              team_slug: team,
              username: username,
            });
        }
    ```  
    with:  
    ```
            console.log('Would be removed: ' + username);
            // await github.request('DELETE /orgs/{org}/teams/{team_slug}/memberships/{username}', {
            //   org: context.repo.owner,
            //   team_slug: team,
            //   username: username,
            // });
          }
    ```  

  - IMPORTANT: Disable the entirety of the `closePrework()` function, starting around ln 110:
    ```  
    async function closePrework(member, issueNum){ 
      // Close the assignee's "Pre-work Checklist" and add comment
      // await github.request('PATCH /repos/{owner}/{repo}/issues/{issue_number}', {
      //   owner: org,
      //   repo: repo,
      //   issue_number: issueNum,
      //   state: 'closed'
      // });
      console.log(`Would be closing "Skills Issue" issue number  ${issueNum} for ${member}`);
      // // Add comment to issue
      // await github.request('POST /repos/{owner}/{repo}/issues/{issue_number}/comments', {
      //   owner: org,
      //   repo: repo,
      //   issue_number: issueNum,
      //   body: 'The Hack for LA Bot has closed this issue due to member inactivity.'
      // });
    }
    ```  
- In `create-new-issue.js`:
  - For `const AGENDA_ISSUE_NUM = ` replace with an **existing** issue number in your repo 
  - The variables for `const owner` and `const repo` are ok as is- do not replace.
  - IMPORTANT: replace the following lines exactly as shown:
    ```  
    let removedList = removeList.map(x => "@ " + x).join("\n");    // important to add space
    let notifiedList = notifyList.map(x => "@ " + x).join("\n");   // important to add space
    ```  
  - Towards the end of the files, comment out: `// let milestone = parseInt(issueObject['milestone']);`
  - And next: `// milestone,`
- In `/utils/get-team-members.js`:
  - Ln 16, replace with `org: 'hackforla',`
- In `/utils/add-team-member.js`:
  - Ln 11, replace with `org: 'hackforla',`
  - Ln 18, replace with `org: 'hackforla',`
- In `/utils/get-timeline.js`:
  - Ln 16, replace with `owner: 'hackforla',`
  - Ln 17, replace with `repo: 'website',`
- In `/utils/post-issue-comment.js`:
  - Ln 9, replace with `owner: ' <your name> ',` **_NOT_** 'hackforla'
  - Ln 10, replace with `repo: 'website',` or however you named your repo
  

## Testing Resources
- [Hack for LA's GitHub Actions (interim update)](https://github.com/hackforla/website/issues/6537#issuecomment-2041147335)
- [How to Test GitHub Actions](https://drive.google.com/drive/u/0/folders/1MPY9CKcfKKN7hpDCG46ARrRzRSM8d8OA)
- [Additional Notes for GitHub Actions](https://docs.google.com/document/d/1frtvr5twBa_3yRGCG0divhlOMW8dPJxT/edit)