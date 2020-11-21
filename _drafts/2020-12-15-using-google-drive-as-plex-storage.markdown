---
title: Using Google Drive as storage for Plex
excerpt: How to use Google Drive Storage to have an on the fly encrypted storage system to stream data from Google Drive to Plex.
link: https://andrewdoering.org/blog/2020/12/15/using-google-drive-for-plex
author: Andrew Doering
published: true #Remove this after finishing the document
comments: true
date: 2020-11-01 01:00:00 -0700
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

Plex is a great tool, however data/nas systems can take up a lot of space and electricity for rotating disks. To keep costs down, automate ripping and organization of legal media constantly coming in, and 

## Requirements

### Where to obtain media

* Library
* Stores (such as Best Buy, Media Markt, etc)
* Digitally (iTunes, etc)

### Automation tools

* [radarr](https://radarr.video/)
* [sonarr](https://sonarr.tv/)
* [lidarr](https://lidarr.audio/)
* [Bazarr](https://www.bazarr.media/)
* [traktarr](https://github.com/l3uddz/traktarr)
* [readarr](https://github.com/Readarr/Readarr) (once stable)
* [lazylibrarian](https://gitlab.com/LazyLibrarian/LazyLibrarian)

### Servers/Players

* [calibre](https://calibre-ebook.com/)
* [Plex](http://plex.tv/), [emby](http://emby.media/), [kodi](https://kodi.tv/) (Your choice/favorite)

### Other
* [rclone](https://rclone.org/downloads/)
* [mergerfs](https://github.com/trapexit/mergerfs)
* Google Workspace Enterprise account
* Various scripts

## Rclone Configuration

