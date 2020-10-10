---
title: Placeholder Title
excerpt: Placeholder Description
link: https://andrewdoering.org/blog/
slug: 
author: Andrew Doering
published: false #Remove this after finishing the document
comments: true
date: 2020-09-20 18:20:49 -0700
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
img_blog: /assets/blog/2020/10/ #Must be 800 x 650 pixels
img_alt: Placeholder Alt Text
permalink: /blog/:year/:month/:day/:title/
---



- [Existing Infrastructure](#existing-infrastructure)
- [Investigating next steps](#investigating-next-steps)
- [Planning our Workflows and Processes](#planning-our-workflows-and-processes)
- [Phase 1 - Setting up BambooHR and Okta Integration](#phase-1---setting-up-bamboohr-and-okta-integration)
  - [Requirements:](#requirements)
  - [Sandbox Testing](#sandbox-testing)
  - [Production Implementation](#production-implementation)
- [Phase 2 - Import all Active Directory Attributes](#phase-2---import-all-active-directory-attributes)
- [Phase 3 - Migrate membership and application assignments from Active Directory groups and data to Okta groups and data](#phase-3---migrate-membership-and-application-assignments-from-active-directory-groups-and-data-to-okta-groups-and-data)
- [Phase 4 - Switch the Profile Master from Active Directory to BambooHR](#phase-4---switch-the-profile-master-from-active-directory-to-bamboohr)
- [Phase 5 - Turn off Profile Master from Active Directory](#phase-5---turn-off-profile-master-from-active-directory)
- [Phase 6 - Remove delegated authentication](#phase-6---remove-delegated-authentication)
  - [End Users](#end-users)
  - [Administrator Dashboard](#administrator-dashboard)
- [Phase 7 - Push membership, attributes, values, and groups from Okta into Active Directory](#phase-7---push-membership-attributes-values-and-groups-from-okta-into-active-directory)
- [Phase 8 - Decode Base64 encoded values from Okta](#phase-8---decode-base64-encoded-values-from-okta)
- [Phase 9 - Migrate Meraki/Wi-Fi authentication from AD to EAP-TTLS with Okta](#phase-9---migrate-merakiwi-fi-authentication-from-ad-to-eap-ttls-with-okta)
- [Summary](#summary)
- [Questions?](#questions)

