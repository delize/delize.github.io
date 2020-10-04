---
title: Authorizing MongoDB Atlas users against Okta's LDAP Groups
excerpt: Previously MongoDB systems earlier than 4.4 versions, did not properly support doing lookups in Okta's LDAP interface properly, but recently that has changed. Come find out how!
link: https://andrewdoering.org/blog/2020/09/23/authorizing-mongodb-atlas-users-against-oktas-ldap-groups
slug: authorizing-mongodb-atlas-users-against-oktas-ldap-groups
author: Andrew Doering
comments: true
date: 2020-09-22 06:00:00 -0700
last_modified_at:
layout: post
tags:
- Okta
- LDAP Interface
- MongoDB
- Atlas
- MongoDB Atlas
- Cloud
- office
- okta
- authorization
- authentication
img_bg: /assets/blog/2020/09/mongodb_logo.png
img_bg_alt: MongoDB Logo taken from mongodb.com
img_blog: /assets/blog/2020/09/stay-on-the-path.jpg
img_alt: Image about MongoDB Authorization Issues with Okta LDAP Interface. Intro image.
permalink: /blog/:year/:month/:day/:title/
---

- [Introduction](#introduction)
- [Issue](#issue)
- [Solution](#solution)

# Introduction

As could be understood by my recent blog postings, we have been migrating to an Okta LDAP entry point as our primary LDAP service. Working towards a Cloud Centric first expectation. As could be imagined, there have been some issues popping up while moving infrastructure and making necessary changes. This describes one of these issues. One of these issues was the way that MongoDB Atlas handled LDAP look ups during the authorization process. 

# Issue

When running MongoDB Versions equal to or lower than, 3.6.19, 4.0.20, 4.2.9 and using the Mongo `Query Template` for Authorization on LDAP Interface :

`{% raw %}ou=groups,dc=<okta-instance-id>,dc=okta,dc=com??sub?(&(objectClass=groupofUniqueNames)(uniqueMember={USER})){% endraw %}`

We would receive failed messages in the shell for MongoDB Atlas. While on the Okta side, we saw several rate limit messages.


MongoDB did group lookups in an inefficient way due to excluding the `DN` value and/or specifying a specific `CN` value from the lookup and attempting to broadly search for a single user within the group that the lookup is being performed on. Binding on every attempt to make a connection to Okta's LDAP Interface. Effectively hammering Okta's LDAP server - which has API limitations [outlined here](https://developer.okta.com/docs/reference/rate-limits/). There was an [issue](https://jira.mongodb.org/browse/SERVER-43233) present on MongoDB's public facing issue tracker (however, not realizing this before hand - we would never have known to look through 47968+ potential issues.)

We reached out to Okta to request an increase to the API limit in the mean time while continuing to test around the issue which they performed on the following:

`{% raw %}/api/v1/users/{id}/groups | READ / WRITE | EXACT_MATCH | 1000 / minute{% endraw %}`

However, even with the API increase, we still ended up running into events in the Okta System Log resulting in group lookups not being completed due to rate limiting, and on the MongoDB/Authorization side - failed authorization results.

We ultimately raised a ticket internally with MongoDB Support, to ask if back porting the fix introduced into 4.4.* would be possible, however there was not an ETA at the present time.

# Solution

After several weeks of waiting for the issue to be back ported. It finally has! Beginning with MongoDB Atlas Versions 3.6.20, 4.0.21, and 4.2.10 you can use Okta's LDAP Interface Okta mastered Groups for Authorization.

The comments written directly by the PoC in charge of the incident filed:


        Please note that in order to utilize the Okta-optimized codepath, you MUST request the DN attribute in the authorization query explicitly.

        Here's the example configuration (swap the `yourdomain` below to your okta tenant domain, as well as be aware of the `your_service_account` ):


        LDAP server: yourdomain.ldap.okta.com
        Server Port: 636
        Bind Username: uid=your_service_account@domain.tld,dc=yourdomain,dc=okta,dc=com
        User To DN Mapping: {% raw %}[ { "match": "(.+)", "substitution": "uid={0},ou=users,dc=yourdomain,dc=okta,dc=com" } ]{% endraw %}
        Query Template: {% raw %}ou=groups,dc=yourdomain,dc=okta,dc=com?dn?one?(&(objectClass=groupofUniqueNames)(uniqueMember={USER})){% endraw %}


Once configured, you should be able to use Okta's LDAP Interface in MongoDB Atlas and have successful Authorization results!
