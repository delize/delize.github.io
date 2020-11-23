---
title: Tracking Github Accounts and automatically assigning the application
excerpt: 
link: https://andrewdoering.org/blog/2020/11/20/assigning-applications-on-attributes
author: Andrew Doering
published: true #Remove this after finishing the document
comments: true
date: 2020-11-20 01:00:00 -0700
last_modified_at: 
layout: post
tags:
- list
- all
- tags
- here
categories:
- random
img_bg: /assets/blog/2020/10/
img_bg_alt: Placeholder Alt text
img_blog: /assets/blog/2020/10/ # Image must be 800 x 650
img_alt: Placeholder Alt Text
permalink: /blog/:year/:month/:day/:title/
---

## Introduction

We historically had made a change from Github Enterprise to Github Cloud. One of the downsides of this is that this would require external Github accounts that were not really within "control", or included management of accounts. We needed to setup tracking of this information. We also needed a way to automate groups, which up [until recently](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-organizations-and-teams/synchronizing-a-team-with-an-identity-provider-group) was not possible. However, the route we did go was able to be done in a self-service aspect (with a few limitations).

## Create an Okta mastered attribute 

So very similar to the blog post on pushing [SSH keys from Okta to AD](https://andrewdoering.org/blog/2020/11/15/pushing-ssh-keys-from-okta-to-ad/), we need to create an attribute to host Github Account information.

1. Go to [`https://yourdomain.okta.com/admin/universaldirectory`](https://yourdomain.okta.com/admin/universaldirectory)
2. Select `Profile` under `User (default)`
  ![Okta Universal Directory](/assets/blog/2020/11/push-ssh-keys/okta-step1.png)

3. Select `Add Attribute`
  ![Okta Profile](/assets/blog/2020/11/push-ssh-keys/okta-step2.png)

4. Once the `Add Attribute` window has appeared, fill out the prompts with the following options:
    * Display Name: `SSH Public Key`
   * Variable Name: `com_github_accountname` or `com_github_username`
   * Description: Add what you would like here, an example is below in the photograph
   * Once done, select `Save`

5. Once the attribute is created, scroll down to the bottom, and find the newly created Okta attribute and click the pencil icon to edit the attribute.
  ![Okta step 4](/assets/blog/2020/11/assign-github-on-attribute/github-attribute-1.png)

6. You will want to change the attributes for `User Permission` and `Master Priority` to match to the picture below.
  ![Okta step 5](/assets/blog/2020/11/assign-github-on-attribute/github-attribute-2.png)

7. While optional, you may also want to change the `Attribute Length` to `1` to `39`. Github has a maximum value of 39 characters, which can be seen [here](https://github.com/shinnn/github-username-regex) and [here](https://gist.github.com/tonybruess/9405134), and [here](https://github.com/join) (only when typing something more than 39 characters). This way there is validation of the account name length in place and someone doesn't put the incorrect account in potentially.

## Create Okta group & rule

### Group

Create a group that will be assigned to the Okta application. My recommendation would be to call it `iam_github` or something to that extent, just to keep things very simple. You can go ahead and manually add a few people there for now, however, you ultimately do not want to manually assign users to this group, so that the group rule can take effect.

### Group Rule

For a basic implementation of assigning the Okta application to the user. This is assuming that you do not have team groups configured/setup, and only have a single group assigned to the Github applications. If you want to also do validation (as in, you require a prefix, or suffix attached to the account), you can modify the expression language below to validate that an account matches those requirements before assigning the application.


* Okta Expression Language: `(user.com_github_username != null or user.com_github_username == "" )`
* Okta Group: Your Github application group

Once created, mark it as active.


## Assign the group to the application

Once the group & rule is finished being created, assign the group to the application.

# So what happens now?

If an employee has not filled out the attribute in their Okta profile, they will not be assigned the application. When an employee does fill out the application, if you have added the syntax check (such as a suffix of prefix), it will assign the application assuming it meets any checks.

Once added to Github, without doing the previously mentioned assigning users to [teams](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-organizations-and-teams/synchronizing-a-team-with-an-identity-provider-group). Everything going forward would be self-service/self-requested when joining teams within the org. And from there, managers of each team can get approval notifications and add team members to the team once the SSO connection has been established.

You can further expand on this by creating multiple groups and checks within the rule, for instance, creating a group rule that checks against department attribute + okta username, to assign a different group and assign them the appropriate team with the URL above. However, you are still limited to five groups per team. So there needs to be careful consideration on how you apply those groups. 

The other option is to do this using Okta workflows, this would do things directly against the API, and may not be limited to the five team limitation as described in the documentation.