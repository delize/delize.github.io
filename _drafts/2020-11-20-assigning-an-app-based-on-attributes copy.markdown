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

We historically had made a change from Github Enterprise to Github Cloud. One of the downsides of this is that this would require external Github accounts that were not really within "control", or included management of accounts. We needed to setup tracking of this information. We also needed a way to automate groups, which up [until recently](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-organizations-and-teams/synchronizing-a-team-with-an-identity-provider-group) was not possible. 

## Create an Okta mastered attribute 


## Create Okta group rule

For a basic implementation of assigning the Okta application to the user. This is assuming that you do not have team groups configured/setup, and only have a single group assigned to the Github applications.

* Okta Expression Language: `(user.com_github_username != null or user.com_github_username == "" )`
* Okta Group: Your Github application group
