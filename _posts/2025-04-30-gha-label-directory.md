---
layout: post
title: GHA Label Directory
date: 2025-04-30 16:00 -0700
description: GitHub Actions Automation to Monitor Labels in Repo
image: 
  path: ../assets/img/site_images/label_directory_gha/label_directory_0r.png
  alt: Update Label Directory Workflow
category: [Automations]
tags: [github actions, workflows, google apps script, yml, yaml, javascript]
---

<figure>
  <img src="../assets/img/site_images/label_directory_gha/base_template_flow_only.png" alt="" title='Flow diagram of "Update Label Directory" workflow' style="max-width:90%; margin:auto; box-shadow: 4px 4px 8px rgba(192,192,192,0.5);">
  <figcaption style="font: italic small sans-serif; text-align:center">Flow diagram of "Update Label Directory" workflow</figcaption>
</figure>

# Update Label Directory<br><sub>Workflow</sub>

## Summary
The `update-label-directory.yml` workflow documents and manages the use of labels in the `hackforla/website` repository ('Repo labels') and ensures that the `label-directory.json` file ('JSON labels') and the Google Sheets 'Source of Truth' ('SOT labels') are kept in sync whenever a label is created, edited, or deleted in the `hackforla/website` repository. Specific details and functionalities are described below. 

### Workflow Diagram

<details><summary>Workflow Diagram</summary>


<img src="../assets/img/site_images/label_directory_gha/base_template.png" alt="" title='Full diagram of "Update Label Directory" workflow' style="max-width:90%; margin:auto; box-shadow: 4px 4px 8px rgba(192,192,192,0.5);">
<figcaption style="font: italic small sans-serif; text-align:center">Full diagram of "Update Label Directory" workflow</figcaption>

</details> 

### Trigger
- Workflow is triggered whenever a label is created, edited, or deleted from the `hackforla/website` repository.

### Labels Used by Workflow
- all labels are used indirectly

### Tokens Used by Workflow
- `HACKFORLA_BOT_PA_TOKEN` 
  - Scopes: public_repo, admin:org_hook

### Workflow Files

- [update-label-directory.yml](https://github.com/hackforla/website/blob/gh-pages/.github/workflows/update-label-directory.yml)
  - [update-label-directory.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/update-label-directory.js) (in utils folder)
  - [label-directory.json](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/data/label-directory.json) (in utils/data folder)

  Complementary file (not part of workflow):
  - [retrieve-label-directory.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/retrieve-label-directory.js) (in utils folder)
 
#### Support File Folders  
- [.github/workflows](https://github.com/hackforla/website/blob/gh-pages/.github/workflows)
- [github-actions/utils](https://github.com/hackforla/website/tree/gh-pages/github-actions/utils)  
- [github-actions/utils/data](https://github.com/hackforla/website/tree/gh-pages/github-actions/utils/data)


### Background
This workflow and the supporting functions and documents surrounding it were created for these reasons:
1. The [Repo Labels](https://github.com/hackforla/website/issues/labels): 
    - The labels used for issue-making, Project Board tracking, etc. for the `hackforla/website` repository are maintained in the repo itself and are referred to as the "Repo Labels". The "Repo Labels" are the current, actual labels that the repo 'knows about'. 
    - Prior to the "Update Label Directory" workflow, our codebase often referenced label names (labelName): for example, workflows such as "Schedule Friday 0700" include functionality to add or remove specific labels from issues, and the new issue templates specify preliminary labels to attach when an issue is created. If we need to make an edit to the labelName in the "Repo Labels", then we would also need to search for and ensure that we edit the labelName in each place it occurs, otherwise the workflow may have crashed or we may not have had the correct default labels applied to new issues.
    - To avoid this situation we decided to create a JSON file of key:value pairs so that our workflows, new issue templates, and all other code could reference a labelKey rather than a labelName. With the JSON, we could change the labelName is many times as we wanted and not worry about changing the codebase, as long as the JSON file kept track.
2. The [JSON Labels](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/_data/label-directory.json):
    - This workflow updates and uses a `label-directory.json` file, aka the "JSON Labels". The "JSON Labels" match labelKeys with labelNames, allowing the codebase to refer to a labelKey rather than a labelName. Accessory functions ensure that the labelKeys and the labelNames are coordinated.
    - When another workflow or template refers to a labelKey, the [retrieve-label-directory.js](https://github.com/hackforla/website/blob/gh-pages/github-actions/utils/retrieve-label-directory.js) utility retrieves the up-to-date labelName.
3. The [SOT Labels](https://docs.google.com/spreadsheets/d/1Y1-ba1ITSs6L6Av0mtX0Qy-Oq-tsKP_rwR-AJfuDYCk/edit#gid=0):  << Check link!
   - This workflow also sends data to a Google Apps Script, which in turn updates a Google Sheets worksheet named `"Source of Truth Labels" Directory`.  
   
TO BE CONTINUED

### Process
- The [update-label-directory.yml](https://github.com/hackforla/website/blob/gh-pages/.github/workflows/update-label-directory.yml) workflow is triggered when a label is created, edited, or deleted in the `hackforla/website` labels.


TO BE CONTINUED



## Test Procedure
- You will need to have a functioning test environment on your local repo. Refer to [Hack for LA's GitHub Actions](https://github.com/hackforla/website/issues/6537#issuecomment-2041147335), especially Tips 6, 7, & 8.
- In addition to the 'files changed' in the PR, there are additional changes that you should make to help with testing. 
- The `create-label-directory.yml` workflow requires a personal access token. Make sure that you have set up the token with the following scopes (see Tip 7): 
  - `HACKFORLA_BOT_PA_TOKEN` scopes: admin:org_hook, public_repo 
- In the same file, around line 13, replace 'hackforla' with your personal repo's specs.
- In the same file, around line 40 check that the website link is active.
- Note that although having all of HfLAs label is not necessary, Tip 8 explains a simple way to copy all of HfLA's labels to your own repo from the GitHub CLI.
- No other changes are needed for running the workflow. 
- Suggested tests:
  - Go to your personal repo's label page at: `https://github.com/<owner>/<repo>/issues/labels` where 'owner' and 'repo' refer to your setup. Note- you do not need to match any labels, or even have labels yet. Do not attempt to edit the labels in the `label-directory.json` file. These match HfLA only.
  - Begin by selecting "New label" in the upper right. Enter the name, and optionally the description and color.
    - Select "Create label" and confirm that the new label is added.
    - Repeat with two other labels. 
    - _(FUTURE: add script to automatically create labels)_
    - Next, open the `label-directory.json` and confirm that the new labels are added here is well (probably the last entries).
    - You can also review the "Action" logs to make sure there were no errors.
  - Next, select one of the labels you just created and change the color only. The "Action" log will run but there should be no changes.
  - Now, select on of your labels and change the name. The label should be changed in your repo and in the JSON file.
  - Finally, select on of your labels and delete it. The label should be removed from your repo, however in the JSON file it should be renamed "Your label" + " (deleted)", and the labelID should be reset to 9999999999.
 - Refer to the "Source of Truth" document (at the link for the Google Sheets). 
   - All of the changes you made should be detailed on the sheet with the tab titled "Worksheet: GitHub Label Updates Log", and edited, created, and deleted labels should be shown at the top of the "'SOT Labels' Directory" tab.

## Testing Resources
- [Hack for LA's GitHub Actions (interim update)](https://github.com/hackforla/website/issues/6537#issuecomment-2041147335)