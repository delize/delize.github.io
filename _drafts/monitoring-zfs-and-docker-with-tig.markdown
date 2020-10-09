---
title: Monitoring ZFS and Docker with TIG
excerpt: Long term monitoring with Telegraf, InfluxDB, and Grafana for Docker, ZFS, and System Performance. This will go into graph reporting and metrics. 
link: https://andrewdoering.org/blog/2020/10/10/monitoring-zfs-and-docker-with-tig
slug: monitoring-zfs-and-docker-with-tig
author: Andrew Doering
comments: true
date: 2020-10-10 12:00:00 -0700
last_modified_at: 
layout: post
tags:
- list
- all
- tags
- here
img_bg: /assets/blog/2020/10/
img_bg_alt: Placeholder Alt text
img_blog: /assets/blog/2020/10/
img_alt: Placeholder Alt Text
permalink: /blog/:year/:month/:day/:title/
---



# Introduction

I have a homemade NAS system at home, that incorporates ESXi, Ubuntu, ZFS, and Docker for applications/services I want to run. One of the biggest issues I have is monitoring long term performance on the device itself. Because I am not running VSphere and am solely runnng ESXi itself, it makes it difficult to grab metrics from the hypervisor, so I am left to grab metrics from the guest. 

I wanted a way to have historical data from the system in case something went _horribly_ (not really, but maybe) wrong for troubleshooting purposes, being sent to another server. 

A bit of background on the system in place here, and yes - I am a data hoarder and a fan of data storage systems.


__ESXi Hypervisor Specs:__

* CPU: AMD Ryzen 7 1700 Eight-Core Processor
* Memory: 64 GB ECC Memory
* Storage:
  * NVME: 1x Samsung 970 Pro 1TB, 1x WD SN750 1TB
  * SSD: 2x Intel DC S3510 120 GB 
* GPU: Nvidia GT 710
* Motherboard: ASRock X470 Master SLI/AC

__NAS Guest Specs:__

* OS: Ubuntu 20.04 LTS
* Memory: 48 GB
* Root Drive: 40 GB
* ZFS Pool: 32 TB (WD RED 6TB Drives - SATA_WDC_WD60EFRX-68L_WD)
* Cache: 1TB
* HBA: LSI SAS2008 PCI-Express Fusion-MPT SAS-2 Falcon rev. 03 ([Flashed to IT mode](https://forums.serverbuilds.net/t/guide-updating-your-lsi-sas-controller-with-a-uefi-motherboard/131))

You might think 48 GB of memory is overkill for a system that mainly deals with storage and a few applications. Unfortunately, not at the current moment - but I do agree, it is overkill. I also use rclone as an online storage system with Google Drive as a backend. Meaning this is my normal:

```
    nas:~$ free -mh
                total        used        free      shared  buff/cache   available
    Mem:           47Gi        31Gi       1.1Gi       9.0Mi        14Gi        15Gi
    Swap:         3.8Gi       3.0Gi       909Mi
```
There is definitely some room for improvement, don't get me wrong, but at the moment, without fine tuning it further and it operating smoothly for a few months this is where it is at. 

So with all that said and done, the things I wanted to monitor came out to be:

* Memory Usage
* CPU Usage
* File System Usage
* Docker 
* ZFS Pool
* Network Traffic
* 

# Getting Started

## Getting InfluxDB and Grafana running in Docker

Thankfully, Grafana and InfluxDB already provide decent out of the box docker images, so no modification is needed for a small scale deployment and what I am trying to accomplish.

I started out doing this for my docker-compose file

```
    version: '3.3'
    services:
        grafana:
            ports:
                - '3000:3000'
            container_name: grafana
            environment:
                - 'GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource'
            volumes:
                - '/media/docker/grafana/plugins:/var/lib/grafana/plugins'
                - '/media/docker/grafana/logs:/var/log/grafana'
                - '/media/docker/grafana/config:/etc/grafana/'
                - '/media/docker/grafana/data:/var/lib/grafana'
            restart: always
            image: grafana/grafana
            networks:
                - lame
        influxdb:
            container_name: influxdb
            ports:
                - '8086:8086'
                - '2003:2003'
            environment:
                - INFLUXDB_HTTP_AUTH_ENABLED=true
                - INFLUXDB_ADMIN_USER=myuser
                - INFLUXDB_ADMIN_PASSWORD=mysecret
            volumes:
                - '/media/docker/influxdb:/var/lib/influxdb'
            restart: always
            image: influxdb
            networks:
                - lame
    networks:
        lame:
            name: lame
            driver: bridge

```

I am aware Docker volume stores are supposedly the more supported/official way to go, but, I can't get away from accessing files directly off the host filesystem - it's just convenient to me.

