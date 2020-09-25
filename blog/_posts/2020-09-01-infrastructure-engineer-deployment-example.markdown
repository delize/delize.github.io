---
title: Infrastructure Engineer Deployment - Example
excerpt: This is an infrastructure engineer assignment outline, that was created for an interview challenge.
link: https://andrewdoering.org/blog/2020/09/01/infrastructure-engineer-deployment-example
slug: infrastructure-engineer-deployment-example
author: Andrew Doering
comments: true
date: 2020-09-01 18:20:49 -0700
layout: post
tags:
- Office Deployment
- Infrastructure Engineer 
- Interview Example
- Homework Assignment
img_bg: /assets/blog/2020/09/eg-infra-engg/infra-engg-bg.jpg
img_bg_alt: Background for Infrastructure Engineer Article
img_blog: /assets/blog/2020/09/eg-infra-engg/infra-engg-icon.jpg
img_alt: Icon Image, Infrastructure Engineer Article
permalink: /blog/:year/:month/:day/:title/
---


# Introduction

I created the following document as part of a challenge for an interview for an Infrastructure Engineer position located in another country, and while I ultimately didn't get the position due to extenuating circumstances (namely due to the current Pandemic and Visa/Work Permits), I spent a lot of time on this document, and would like to publish it for others - maybe it could give them some ideas going forward. This is intended as high level work, not as a detailed document or 100% accurate in terms of deployment.

This is nearly verbatim to the document that was sent in for the homework assignment, with some minor wording and documentation edits.


- [Introduction](#introduction)
- [Assignment / Request](#assignment--request)
  - [Assumptions](#assumptions)
- [Deployment](#deployment)
  - [Options for Services](#options-for-services)
  - [Hardware](#hardware)
  - [Services infrastructure deployment](#services-infrastructure-deployment)
- [Network Logistics](#network-logistics)
  - [Subnetting](#subnetting)
    - [Corp](#corp)
    - [Cloud](#cloud)
  - [Corporate Network](#corporate-network)
    - [Initial Deployment](#initial-deployment)
    - [Future Improvements](#future-improvements)
    - [Firewall](#firewall)
    - [QoS](#qos)
    - [Guest Network](#guest-network)
    - [Cloud Network](#cloud-network)
    - [VPN](#vpn)
- [Cloud Infrastructure](#cloud-infrastructure)
  - [Asset Management (Cloud) and IPAM (Cloud)](#asset-management-cloud-and-ipam-cloud)
    - [Asset Management](#asset-management)
    - [IPAM](#ipam)
    - [Deployment Steps](#deployment-steps)
      - [Asset Management](#asset-management-1)
  - [CI/CD System](#cicd-system)
  - [Identity Management Infrastructure](#identity-management-infrastructure)
  - [Task Management](#task-management)
    - [Jira Cloud vs Jira Server](#jira-cloud-vs-jira-server)
- [Future Plans](#future-plans)
- [Summarization](#summarization)
- [Appendix:](#appendix)
  - [Supporting Files](#supporting-files)
  - [List of Infrastructure Services to Deploy](#list-of-infrastructure-services-to-deploy)
  - [Example docker-compose for SnipeIT hosted in GCP](#example-docker-compose-for-snipeit-hosted-in-gcp)
  - [GCloud Compute Scripts for an Active Directory Deployment](#gcloud-compute-scripts-for-an-active-directory-deployment)
  - [Networking and Software Equipment](#networking-and-software-equipment)



# Assignment / Request
Our team would like for you to create a plan for deploying a network infrastructure plan for a new company.
At the moment, the company does not have any existing infrastructure so it's up to you to create one! 

The requirements and technical limitations are as follows:
* There are three offices, one office will be the headquarters and the other two will be satellite offices. The headquarters will have both R&D and support staff (ie. finances) the remote offices will consist of just R&D.
* The work from home employees will need to connect via a VPN.
* The headquarters will have some infrastructure as well as one of the satellite offices. The other satellite office will not.
* To reduce operational costs; the R&D department would like to deploy some infrastructure in the cloud.

The plan should include; but not necessarily be limited to; the following topics:
* Hardware and software to use (the company is keen on virtual infrastructure or cloud where possible), and the reasons why.
* Requirements for subnets, routing, security, connectivity, etc.
* Abstract configuration for the software (in broad terms - we don’t need every field detailed!).
* A brief sketch of any processes the company may wish to consider deploying in the future to aid in network and data management.
* Some scripting examples of compute resource deployment in the cloud.

We’re not looking for a huge report here; this is primarily an exercise in research and reasoning, along with a demonstration of knowledge of infrastructure requirements.


## Assumptions
* As employee size is not defined, I will assume 200 current employees scaling to 500 employees. Update 2020/06/22, Recruiter mentioned that employee size was ~300.
* As office infrastructure is not defined or categorized as corporate, platform, or both, I assume corporate infrastructure will be deployed in offices only, as hosting elements of platform infrastructure in an office environment is not a best practice. Update 2020/06/22, Recruiter confirmed.
* Cloud infrastructure will also have parts of corporate infrastructure for scalability purposes.
* Non essential network services (dhcp, dns, etc) should be located in the cloud to allow for accessibility and modularity to offices and remote employees. However, the ability to deploy into office infrastructures should be available
* Services that operate in the cloud have volatility. Office and cloud infrastructure should be able to handle volatile availability.
* I am unaware if End User Device infrastructure is a part of this configuration. I have included my past experience with it as a way to complete the infrastructure trifecta (corp, euc, platform). However I will not go into detail here outside of listing the services needed.
* Laptop Infrastructure (macOS and Windows) images are considered satisfactory from the OEM and no “golden image” is required.
* Infrastructure Rack is provided by building management


# Deployment

These are options to deploy the new infrastructure. These are based on two factors:  
* personal experience with each service  
* the flexibility provided by each one


## Options for Services


For a list of infrastructure options for deployment, and to reduce space in this document, please see [the appendix](#list-of-infrastructure-services-to-deploy) for an extended list of Cloud and SaaS based services.

## Hardware

Please see the [Appendix](#networking-and-software-equipment) for a complete list of hardware to buy per office. I really appreciate the Meraki Stack. However, I have not deployed their switches in a production environment; I have only deployed their Access Points. I would need to research more about the benefits and cons of the MS lineup; however, I would highly look into that platform before deploying the switches. As long as the switches can support teaming/bonding, Spanning Tree Protocol, at a minimum, the network configuration should be suitable.


## Services infrastructure deployment


The sections below are based on the options above and my recommendations for deployment of the company's new infrastructure for each office and remote location. Cloud infrastructure will be deployed in GCP. Diagrams will be provided where I find it adds relevance or workflow assistance. Since we won’t be repeatedly building this for a platform infrastructure, we will be using Kubernetes where possible. If we were deploying for the product or platform, we could use the App Engine platform within GCP. This is not a comprehensive list of all configurations, but, it is extensive enough for a homework lab assignment.

# Network Logistics

## Subnetting

### Corp

| Office 	| Level 	|        Use        	| IP Prefix 	| Floor ID 	| VLAN ID 	| Client Address 	| CIDR Mask 	| IP Network      	| Office Domain      	|   	| Notes                                                     	|
|:------:	|:-----:	|:-----------------:	|:---------:	|:--------:	|:-------:	|----------------	|:---------:	|-----------------	|--------------------	|---	|-----------------------------------------------------------	|
|   HQ   	|  1st  	|        DMZ        	|     10    	|    201   	|    5    	| X              	|    /24    	| 10.201.5.X/24   	| osl1.o.company.com 	|   	|                                                           	|
|   HQ   	|  1st  	|      General      	|     10    	|    201   	|   100   	| X              	|    /24    	| 10.201.100.X/24 	| osl1.o.company.com 	|   	| Will use closest three letter IATA code for domain        	|
|   HQ   	|  1st  	|    Engineering    	|     10    	|    201   	|   110   	| X              	|    /24    	| 10.201.110.X/24 	| osl1.o.company.com 	|   	|                                                           	|
|   HQ   	|  1st  	|     Operations    	|     10    	|    201   	|    99   	| X              	|    /24    	| 10.201.99.X/24  	| osl1.o.company.com 	|   	|                                                           	|
|   HQ   	|  1st  	| VoIP/Conferencing 	|     10    	|    201   	|    50   	| X              	|    /24    	| 10.201.50.X/24  	| osl1.o.company.com 	|   	|                                                           	|
|   HQ   	|  1st  	|      Printers     	|     10    	|    201   	|    60   	| X              	|    /24    	| 10.201.60.X/24  	| osl1.o.company.com 	|   	|                                                           	|
|   HQ   	|  1st  	|      Servers      	|     10    	|    201   	|    70   	| X              	|    /24    	| 10.201.70.X/24  	| osl1.o.company.com 	|   	| DHCP, Windows, etc (Services being run from esxi servers) 	|
|   HQ   	|  1st  	|        mgmt       	|     10    	|    201   	|    80   	| X              	|    /24    	| 10.201.80.X/24  	| osl1.o.company.com 	|   	| Access Points, ESXi servers                               	|
|   HQ   	|  1st  	|       GUEST       	|     10    	|    201   	|   192   	| X              	|    /24    	| 10.201.192.X/24 	| osl1.o.company.com 	|   	| Guest ONLY VLAN                                           	|
|   BO1  	|  1st  	|      General      	|     10    	|    202   	|   100   	| X              	|    /24    	| 10.202.100.X/24 	| hrw1.o.company.com 	|   	|                                                           	|
|   BO1  	|  1st  	|    Engineering    	|     10    	|    202   	|   110   	| X              	|    /24    	| 10.202.110.X/24 	| hrw1.o.company.com 	|   	|                                                           	|
|   BO2  	|  1st  	|    Engineering    	|     10    	|    203   	|   110   	| X              	|    /24    	| 10.203.110.X/24 	| osl2.o.company.com 	|   	|                                                           	|
|        	|       	|                   	|           	|          	|         	|                	|           	|                 	|                    	|   	|                                                           	|
|        	|       	|                   	|           	|          	|         	|                	|           	|                 	|                    	|   	|                                                           	|




This is a simple configuration and can be more complex if needed. This also leaves plenty of room to grow as the department grows. It also allows you to specify VLAN IDs to be based on department groups. This does create a problem where you may run into more than 50 combined office locations or floors. If this is the case, removing the N+1 function of the floors in FloorID and changing to Building ID could resolve any capacity issues. If this is still a problem, you could expand on the VLAN ID as a VLAN+Floor ID. Though this could ultimately cause confusion and something I don’t recommend.


### Cloud


| Use Case               	| IP Start   	| IP End         	| CIDR 	| Notes|
|------------------------	|------------	|----------------	|-------------------	|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| Platform               	| 10.0.0.0   	| 10.99.255.254  	|   16 	| This is a total of 6553400 IPs for the platform, from here engineers can then divide the network up as needed for their own services. This would be split out individually, but for the sake of time, I have lumped the whole range together here, and denoted that it has a 16 for each subnet. 	|
| Staging-Platform       	| 10.100.0.0 	| 10.100.255.254 	|   16 	| Within this range, there should be a sufficient amount of IP Addresses for a Staging Network as well.                                                                                                                                                                                            	|
| Staging-Platform       	| 10.101.0.0 	| 10.101.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Staging-Platform       	| 10.102.0.0 	| 10.102.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Staging-Platform       	| 10.103.0.0 	| 10.103.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Reserved               	| 10.104.0.0 	| 10.127.255.254 	|   16 	| Reserved for future uses for the engineering and product                                                                                                                                                                                                                                         	|
| Infrastructure         	| 10.128.0.0 	| 10.128.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Infrastructure         	| 10.129.0.0 	| 10.129.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Infrastructure         	| 10.130.0.0 	| 10.130.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Infrastructure         	| 10.131.0.0 	| 10.131.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Infrastructure         	| 10.132.0.0 	| 10.132.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Infrastructure         	| 10.133.0.0 	| 10.133.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Infrastructure         	| 10.134.0.0 	| 10.134.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Infrastructure         	| 10.135.0.0 	| 10.135.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Infrastructure         	| 10.136.0.0 	| 10.136.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Infrastructure         	| 10.137.0.0 	| 10.137.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Staging-Infrastructure 	| 10.138.0.0 	| 10.138.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|
| Staging-Infrastructure 	| 10.139.0.0 	| 10.139.255.254 	|   16 	|                                                                                                                                                                                                                                                                                                  	|


## Corporate Network

### Initial Deployment

**[Physical Topology Diagram Link](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-Office-Physical_Topology.png)**:
![Physical Topology Diagram Link](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-Office-Physical_Topology.png)

**[Logical Topology Diagram Link](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-Office-Logical_Topology_(Simplified).png)**
![Logical Topology Diagram Link](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-Office-Logical_Topology_(Simplified).png)


As core/distribution switches have much higher failure rates compared to access switches (due to the amount of services running on the core switches), we would be routing traffic via layer 2 mechanisms rather than layer 2 and taking advantage of layer 3 services in the firewall to deal with routing. We would utilize vlans broadcast at specific ports to be able to achieve this. This results in a less expensive network overall.

Ideally, through the use of the IAM tools we will be using later on in the document, we can utilize a full cloud stack for RADIUS for end user wifi authentication. The reasoning behind cloud RADIUS is that if your ISP(s) are down, you will not have much use of the on premise system since you won’t be able to access most of the remote services anyways due to the down(ed) ISP(s). With the RADIUS instances locally, you would be able to authenticate locally to the network and access any services though. While the most secure method is EAP-TLS, it is highly inconvenient. Due to that, my recommendation would be to use EAP-TTLS.

Keeping the DHCP and DNS configurations (zones, static addresses, etc files if required) into a revision control tool (such as Github) would be a good first step if required. We can later automate and make improvements on the deployment of DHCP and DNS configuration files. While using embedded software in the Firewall would be possible, it’s difficult for people to look at the services if they don’t have access to the Firewalls. However for simplicity sake, using the firewall’s DHCP server should be sufficient if there is not a requirement where the firewall would not be able to perform.

Based on the number of employees per office, 500 Mbps lines as a minimum would be required, and 1 Gbps lines would be highly recommended. A backup line with a separate ISP would be required in case the first line goes down or switch goes down, but the speed does not need to be the same as the primary line - 100 to 250 Mbps should be fine. Especially with people in the office streaming music, high quality video, and other things.

### Future Improvements

Deployment of SDN/SD-Wan using VeloCloud, Meraki, or Juniper Sky, to streamline the deployment, management,inter-office communication, and/or the cloud infrastructure data center connection.   Based on the build process, this could be accomplished two ways:

1. Physical device that sits in front of the firewall
2. Virtualized SD-WAN appliance that exists in the hypervisor

If going with step two, the best decision to converge all services where they are most compatible, so choosing ESXi/VSphere now for the Hypervisor with a velocloud virtual device makes sense so that you could have finer control over the networking plane.

In addition to the above, if we want to keep the dhcp server and dns server configuration (and address leases) in a change management and revision service (IE: Github Cloud), we could eventually hook the repository into a CI/CD system to automate the deployment of DHCP and DNS packages to each corporate office through a package repository system. However longer term, the plan should be to move off managed DHCP/DNS internally to minimize complexity to the infrastructure. In this way, your managed network stack boils down to just Firewall, Switch, IDS/IPS, and Access Points. 

Before automation tools and the like are installed, you would want to implement logging, monitoring, and alerting to be aware of issues that are brought up across the network. The tools mentioned in LAMES are good for that, specifically when it comes to host monitoring, and using APIs and other sources. These can generate several graphs to determine baselines. 

To implement network monitoring, not considering my current employer, a good place to consult are Gartner reports as a source of information. Gartner has also sunsetted their Magic Quadrant NPM reports for [Market Guides](https://www.gartner.com/en/documents/3981838/market-guide-for-network-performance-monitoring-and-diag), and more companies are moving into the [Digital Experience Monitoring](https://www.gartner.com/en/documents/3956998/market-guide-for-digital-experience-monitoring) space. However, from an internal network perspective and focusing on open source tools, using a combination of ElasticSearch, Logstash, Kibana, Grafana, and open source forwarders (osquery, telegraf, etc.), you can accumulate decent network monitoring tools. I previously had experience with Nagios, but it had deficiencies compared to the set of tools above. In addition, using Network Performance Monitoring tools is becoming increasingly hard due to increased security around packets - overall a good thing. 

Lastly, short of keeping and manually exporting the configuration set of the switch or firewall, a tool that can automate the backups of the switch and firewall directly into a datastore would be required. 


### Firewall

I find that firewalls that set a “drop all rule” rather than a “drop specific/known malicious services” can be too restrictive. I would say that continually monitoring the firewall for the first few weeks of a new deployment would be a better part of the deployment process. That said, the initial process should be to get a list of known services to implement and run (for instance, Active Directory on premise, Video Conferencing, SIP, etc) and then be sure to specify port rules closely. If the rules are too vague, reach out to the vendor to collect that information. In addition, make sure to follow guidelines based on [NIST](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=901083) and alternatives.

Since we are using Palo Alto firewalls and they have the ability to firewall services (IE: Applications) as well as specific ports, we can be very dynamic when it comes to firewall allowance. A (very rudimentary) example can be found below, as well:

**Ingress (inbound)**
1. VPN Protocols
2. Block all traffic

**Egress (outside)**
1. HTTP
2. HTTPS
3. DNS
4. IMAP
5. POP
6. SMTP
7. ssh
8. SIP
9. Zoom / WebEx / PEXIP
10. Other required services
11. Block all

**Inside (intra-vlan)**
1. ping
2. Telnet / netcat
3. Multicast
4. ssh
5. Block other traffic


### QoS
Splitting out the single lane of QoS into multiple lanes, and transmitting Real Time Packets (Video, Audio, Multimedia streaming, etc) and non real time packets (http, messaging services, imap/pop, smtp, etc) is beneficial. There are multiple ways to configure QoS in Palo Alto devices. It would be helpful. As the Pexip platform supports [DSCP marking](https://docs.pexip.com/admin/infinity_features.htm), in addition so does [Zoom](https://support.zoom.us/hc/en-us/articles/207368756-QoS-DSCP-Marking), and [WebEx](https://help.webex.com/en-us/WBX83779/What-are-the-System-Requirements-for-Cisco-Webex-Video-Platform) [does as well](https://help.webex.com/en-us/WBX83779/What-are-the-System-Requirements-for-Cisco-Webex-Video-Platform) we can also [configure Palo Alto](https://docs.paloaltonetworks.com/pan-os/9-0/pan-os-admin/quality-of-service/enforce-qos-based-on-dscp-classification.html) to recognize the markings as well. 

### Guest Network

The guest network would be firewalled off from the rest of the network, running under 192.168.1.0/24 subnet. Traffic would only be confined to this network, and would not be able to traverse outside the 192.168.1.0/24 subnet.


### Cloud Network

The de facto way to connect branch offices to the cloud, is using IPSec tunnels. The IPSec tunnel can be configured in the same subnet space as 172.168.0.0/18 space described in the next space, after the Operations subnet space. 
Future Improvements

The other consideration could be a zero-trust/beyondcorp model. The idea behind this is using certificates and other device state facts, that can be incorporated into the use of trusted devices on untrusted networks, without the use of VPN. This would make remote employees’ lives much easier when it comes to management of infrastructure, access to infrastructure, and flexibility on access to services. The idea behind this is making tons of contextually aware identity proxies that allow access to services around the network. This is similar to Google’s Cloud Identity Access systems.

The same method of Office Networks, incorporating SD-WAN here, could be beneficial to help ease the pain of manually creating IPSec tunnels between the environments. 


### VPN

The VPN should be hosted in the cloud, and run off a 172.16.0.0/18 Subnet for maximum compatibility to both the corporate and cloud network. 

| Profile           	| IP                           	| Default Route 	| Notes                                                                                                                                                                                                                                                                                            	|
|-------------------	|------------------------------	|---------------	|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| HQ-Route          	| 172.16.1.0/24                	| Yes           	| Default Routes are only used for the purpose of client access, or more secure access to services. Generally, Default Routes should not be necessary for clients connecting to the VPN. However if Customer requires specific IP addresses for access to services, this would be the access used. 	|
| HQ                	| 172.16.2.0/24                	| No            	|                                                                                                                                                                                                                                                                                                  	|
| BranchOffice      	| 172.16.3.0/24                	| No            	|                                                                                                                                                                                                                                                                                                  	|
| General-Route     	| 172.16.4.0-172.16.7.254/22   	| Yes           	|                                                                                                                                                                                                                                                                                                  	|
| General           	| 172.16.8.0-172.16.11.254/22  	| No            	|                                                                                                                                                                                                                                                                                                  	|
| Engineering-Route 	| 172.16.12.0-172.16.15.254/22 	| Yes           	| Has access to Platform network in Cloud                                                                                                                                                                                                                                                          	|
| Engineering       	| 172.16.16.0-172.16.19.254/22 	| No            	| Has access to Platform network in Cloud                                                                                                                                                                                                                                                          	|
| Operations-Route  	| 172.16.20.0-172.16.23.254/22 	| Yes           	| Has access to MGMT VLAN, Server VLAN, Cloud MGMT VLAN, and other special or restricted VLANs                                                                                                                                                                                                     	|
| Operations        	| 172.16.24.0-172.16.27.254/22 	| No            	| Has access to MGMT VLAN, Server VLAN, Cloud MGMT VLAN, and other special or restricted VLANs                                                                                                                                                                                                     	|




From an entry point or access perspective, the VPN should be able to access required services inbound as necessary. For instance, the engineering profiles should route to cloud infrastructure (platform, and other infrastructure that has been deployed) but not connect to the management sections of the network (201 vlan). The operations infrastructure should have access to the required amount of VLANs to perform their base job (MGMT, and other related services). 

We could also limit this to a much lower mask initially, but keep the space reserved for the upper mask. As it is unlikely to have more than 1000 clients connected at any one point in time. However, once the company becomes larger, this would be a good starting point.


# Cloud Infrastructure

## Asset Management (Cloud) and IPAM (Cloud)

### Asset Management


The plan here is to hook up to the reseller’s database (if possible) and pull details from orders to feed into SnipeIT. An example of this can be done with CDW with their B2B unit. They drop csv files of purchases into a storage bucket for you to pull from. Alternatively, this could exist when working directly with Dell, Lenovo/IBM, or others. Since the service can also be used with end user computing hardware as well, create scripts to interact with reseller’s websites to automatically load data.

It also allows for the use of tags to easily separate the EUC vs infrastructure based hardware, and has a license integration. When setting up a client to automatically add/create assets, it can also be completely automated - by then checking out a device automatically to the end user based on the API data from the MDM. This would then become the Source of Truth for all assets. There are also several tools that exist that allow you to create inventory and asset information from other services (JAMF, PowerShell, AD, etc). 

Since this requires a database backend, and will host a web frontend over docker/kubernetes, we need to setup a CloudSQL DB for Postgres, and use HELM or deploy SnipeIT directly into Kubernetes. Optionally, we can then do one of two things. As SnipeIT supports a [limited capability](https://github.com/snipe/snipe-it/pull/5142) of SAML/oAuth2, and [progress](https://github.com/snipe/snipe-it/pull/8036) is actively being made to improve it, we would want to hook it up to both LDAP (in this case preferably Okta’s LDAP interface, but, if required, an AD instance inside GCP) and then configure SAML Authentication to the front end to utilize the backend email address configuration.

### IPAM

Very much in the same aspect of the Asset Management tool, Netbox also supports tags, the ability to set configurations based on the API, [SAML authentication](https://github.com/netbox-community/netbox/issues/2328). Alternatively if the need to break away or keep a separate Data Center Inventory Management tool exists from the above Asset Management tool, netbox is also used as a DCIM tool.

Much in the same way with tags support, we would use them to differentiate between sets of configuration, and is quite a versatile tool when it comes to IP Address Management. It’s quite comprehensive.

While the use of dynamic updates to the API would be a customized tool, [some tools](https://github.com/netbox-community/go-netbox) already exist that allow you to upload client data into Netbox. However extensive testing would need to be used to validate their viability into the infrastructure.

[Diagram](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-Asset_Management_IPAM_Deployment.png)
![Diagram](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-Asset_Management_IPAM_Deployment.png)



### Deployment Steps
This can be initially deployed into a server model method, but if we go the serverless method (offloading database servers into GCP, and running the instances through kubernetes) we deal with less overhead of management of servers. The configuration is stated exactly as is described, and can be brought up and scaled up without much effort.

#### Asset Management

[Docker Compose file](/assets/blog/2020/09/eg-infra-engg/example-assetmanagement-docker-compose.yaml) [- Appendix](#example-docker-compose-for-snipeit-hosted-in-gcp)


## CI/CD System

Traditionally, I have used Jenkins and self hosting for a CI/CD system - and this has worked well. You do have to worry about the infrastructure that is a part of the CI/CD system. If Github Cloud meets the business requirements, I would suggest using it over Jenkins. Since Github Cloud Actions uses MacStadium for macOS runners, it makes running services through it much easier and you don’t have to worry about virtualization of macOS hardware. On the Windows and Linux side they utilize AzureAD. This way we can simplify and provide less complexity into the infrastructure. The only issue becomes cost to benefit ratios. For comparison, if we find that Github Enterprise meets the requirements of the business better due to security, going the Jenkins route or CircleCI route would be better. On the Jenkins side, there are multiple guides for GCP deployments if that is the route to go. However while managing Github Enterprise was quite easy from past experience, if deployed incorrectly (which I inherited). It can create enormous latency due to a past team deploying the Github Enterprise system to the west coast of the US while having members in Central and Western Europe. 


## Identity Management Infrastructure

If we are deploying infrastructure into GCP, the best route to use is Google’s Cloud Identity. However we need to populate that data into GCP somehow, and that is where our SoT Identity-AAS (IDaaS) provider comes in. I have personal experience with Okta, and think it has more dynamic uses compared to solely using G Suite as an Identity Management tool for all services. I have used that as the backbone to the rest of the infrastructure. From Okta, you can push users into Active Directory. So far, I have found that it works well. The key thing here is that because you can push groups from Okta to any service in a more flexible way than using Active Directory, and you bypass the limitations of AzureAD (for instance, you can’t push AzureAD users into Active Directory on brand new deployments) you end up with a more flexible and modular system. In addition, you can stand up any number of Active Directory environments with the same data from Okta, making it easy to create “staging” or development AD servers for testing when needed with the exact same data that would be considered production.

[Diagram](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-AD_in_GCP.png)
![Diagram](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-AD_in_GCP.png)


Google Cloud Shell Commands  
[Script/Commands](/assets/blog/2020/09/eg-infra-engg/IAM_AD-Infrastructure_GCP_Shell_Commands.txt)

We are deploying a limited range of machines into this space, so we would want to split the subnet from a 10.X.X.0/24 to a 14 host per subnet /28 CIDR mask. If it is done this way, we can also save a subnet and utilize it for multiple regions and zones which would be ideal. If we don’t want to do that, we could just use 10.X.X.0/24 and 10.X.Y.0/24 as alternative subnets for the AD configuration. However we would waste a ton of IP space.

The reason for deploying a limited range of machines into this space is because we do not need dozens of Windows Domain Controllers. As client devices would be running against other services that would mimic the traditional behavior of binding, we only need to use the Domain Controllers to provide a backup to the LDAP interface being served by Okta. While Google’s LDAP does provide certificate based, there are options to work around the [certificate deployments](https://support.google.com/cloudidentity/answer/9048541).

We want to provision this into two different regions (or at least in two separate regions that are closest to each office and the overall services we are providing), to provide maximum coverage over all. We would then incorporate the use of [VPC Peering](https://cloud.google.com/vpc/docs/vpc-peering) to allow those two to communicate between each other. However, to keep things simple for this document, we will deploy into the same region.

Even after performing all of the commands used above, to use the identity aware proxy, we would need to deploy it first. After doing research, it does not appear as if the command line tool is quite ready to deploy Identity Access Proxy services, as it is still in [beta](https://cloud.google.com/sdk/gcloud/reference/beta/iap/web/set-iam-policy). So I would use the GUI to deploy it for convenience.

## Task Management

### Jira Cloud vs Jira Server


The two major difference between deploying these two are:
1. Price Ratio per User
2. Managed by Atlassian vs Managed by Companies Teams

For companies less than 300 people, it makes sense to deploy Jira Cloud, and add on the corresponding services that are required - especially due to the cost involved in employee salary and maintenance fees. However, once the company hits 500 people or more, it makes more financial sense to pay for Server (and ideally more sense for Data Center due to SAML authentication) and self-host it in the cloud so that it is available to all employees where needed.

Ideally, we want to put the infrastructure behind a load balancer and host multiple front ends so that there isn’t too much load on a single node, I would start out with two and then create more as needed. You would use GCP’s [autoscaler](https://cloud.google.com/compute/docs/autoscaler) functions for this to work, so that as load increases the hosts increase as well. We would be deploying the Cloud Load Balancer with an exposed port 443 and terminate the connection internally on port 8080 (the default port for Jira).

[Diagram](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-Jira_Confluence_Deployment.png )
![Diagram](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-Jira_Confluence_Deployment.png )

# Future Plans
Deploy this using Docker and/or Kubernetes. I don’t believe that this is an officially supported method by Atlassian as I can’t find any documentation on their site about this. But, Atlassian has been creating docker images for both [Jira](https://hub.docker.com/r/atlassian/jira-software) and [Confluence](https://hub.docker.com/r/atlassian/confluence-server) which are listed in Docker Hub. While deploying, we would utilize CloudSQL on GCP for the backend of both services, and Cloud Storage as a way to host uploads, images, and other data. Similar to the compute version, we would still be using Cloud Storage and CloudSQL as the backends, so the deployment or upgrade path to roll out kubernetes should be simple. Again, this would follow the serverless based infrastructure.

[Diagram](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-Kubernetes_Jira_Confluence_Deployment.png)
![Diagram](/assets/blog/2020/09/eg-infra-engg/Company_XYZ-Kubernetes_Jira_Confluence_Deployment.png)

# Summarization
Besides deploying all of the above services, I would start by getting a good foundational service for each requirement of the business. Then, I would grow it and expand upon it. With the above, we have the basics to start doing a lot of “grow your own” processes. Starting on the networking side, the Palo Alto Network Firewalls have built in URL filtering when a license is applied, they have VPN software access for deployments, and can perform several other features as needed by the security team. We can then start to export log data into Kibana/ElasticSearch for analysis

The reason why I have only shown infrastructure being deployed into the cloud, is due to it minimizing any potential security risks to each office. While it might be convenient to deploy infrastructure into the office for initial use, it creates risks when performing assessments and an inconvenience to end-users trying to access those services. In the cases of infrastructure deployed in the office, it should be minimal or what is necessary to achieve the business requirement for the office. From a network standpoint, this should be an IDS/IPS, DHCP, and DNS, networking equipment. From a virtualization standpoint, the infrastructure exists internally to allow local developer and testing infrastructure in each office location as well. 

All in all, SaaS vs Cloud services would be based on a few factors:
1. Cost per seat ratio
2. Business benefit
3. Administration costs
4. Time Costs
Once those factors have been determined, you can then make adequate decisions on where or how to deploy each service. Benefiting both the business and the end user.

With the range of software and services outlined in this document, there should be a complete infrastructure stack for a productive corporate infrastructure.



# Appendix:

## Supporting Files
[Collection of all files](/assets/blog/2020/09/eg-infra-engg/Archive.zip)

## List of Infrastructure Services to Deploy
* Asset Management
  * SnipeIT

* Authentication, Authorization, Accounting, Identity
  * SaaS based HRIS service to integrate with Identity Management tool
    * BambooHR
    * Workday
  * SaaS Identity Management & LDAP (Azure or Okta) 
    * Okta (Recommended)
    * Azure
  * Push SaaS data into Cloud hosted LDAP, if required (Active Directory, AWS managed AD, GCP cloud directory)
* Continuous Integration/Continuous Delivery (CI/CD)
  * Jenkins
  * Github Workflows
  * ArgoCD 
* Containers & Orchestration
  * Docker - Containerization
  * Kubernetes - Orchestration
  * Twistlock - Security/Vulnerability Reporting
* Git / Revision Control
  * Github Cloud (Preferred)
  * Github Enterprise
  * Gitlab Cloud
* Hypervisor
  * ESXi/VSphere
  * Xen
  * proxmox
* LAMES (Logging, Alerting, Monitoring Excellence & Security)
  * Infrastructure Monitoring
  * Grafana
  * Prometheus
  * Elasticsearch, Logstash, Kibana
  * AlertManager
  * Osquery
* Network Performance Monitoring
  * SaaS/Cloud Network Monitoring
    * ThousandEyes
* Network Management
  * SaaS or Cloud based management portal (IE: Software Defined Networking)
    * Meraki
    * Unifi
    * Juniper Sky
    * Palo Alto 
    * Velocloud
  * SaaS/Cloud hosted RADIUS based authentication/802.1x
    * Implement EAP-TTLS - https://tools.ietf.org/html/rfc5281#section-11.2.5
  * Security
    * IDS
      *   Vectra
      *   Cisco Firepower
* IPAM/DCIM
  * Nextbox in Cloud Environment
* Productivity software
  * Email & Document Editing
    * G Suite (primary)
    * O365 (secondary, where required for finance)
  * Instant Communication
    * Slack (Recommended)
    * Microsoft Teams
    * RocketChat
  * Telecommunications/conferencing
    * Phone System
      SaaS based where possible/Cloud self-hosted where not
      * SaaS options:
        * Zoom Phone
        * RingCentral
        * Twilio
        * CloudTalk
      * Cloud self-hosted options:
        * Asterisk
        * OpenSIP
        * FreePBX
    * Video conferencing
        * Zoom
        * WebEx
        * Pexip*
* Task Management
  * Jira (Cloud - SaaS)
  * Jira Server (Cloud - Self Hosted)
  * Asana
* EUC Management
  * Full Disk Encryption
    * Windows - Bitlocker (Using AES- XTS 256) & Crypt-Server for escrow
    * macOS - Crypt2 & Crypt-Server for escrow
  * macOS Management
    * DEP and MDM tool that is compatible with “installapplication” MDM specification/command
      * Fleetsmith
      * Mosyle
      * Simplemdm
      * Micromdm
      * JAMF
      * Workspace One
    * Installapplications - bootstrapping, configuration stored on cloud storage
    * DEPNotify - Visual display to the end user for bootstrapping process
    * Munki - bootstrapping, software management, patch management
    * Nudge - patch management/minimum OS version enforcement
    * Privileges - User escalation to administrator on a per department basis
    * GCP Cloud Storage Backend
  * Time Series Reporting
    * Sal - Compatible with both macOS and Windows, as well as Linux and Chromebooks
  * Windows Management
    * MDM
      * Workspace One
      * Intune
    * Autopilot or OOBE experience - Automates client experience 
    * Configuration Management
      * Chef/
      * Ansible -
    * Package Management
      * chocolatey
      * winget
      * nuget


## Example docker-compose for SnipeIT hosted in GCP

      Docker-compose.yml
      ---
      apiVersion: apps/v1
      kind: StatefulSet
      metadata:
        namespace: snipe
        name: snipe
        labels:
          app: snipe
      spec:
        serviceName: snipe
        volumeClaimTemplates:
        - metadata:
            name: snipe-disk
          spec:
            accessModes: ["ReadWriteOnce"]
            resources:
              requests:
                storage: 100Gi
        replicas: 1
        selector:
          matchLabels:
            app: snipe
        template:
          metadata:
            labels:
              app: snipe
          spec:
            containers:
            - name: snipe-it
              image: imagename
              env:
              - name: APP_KEY
                valueFrom:
                  secretKeyRef:
                    name: snipe-app-key
                    key: APP_KEY
              - name: MAIL_ENV_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: snipe-mail-key
                    key: MAIL_ENV_PASSWORD
              - name: MAIL_ENV_USERNAME
                value: example@example.com
              - name: APP_DEBUG
                value: "false"
              - name: MAIL_PORT_587_TCP_ADDR
                value: smtp.gmail.com
              - name: MAIL_PORT_587_TCP_PORT
                value: "587"
              - name: MAIL_ENV_FROM_ADDR
                value: example@example.com
              - name: MAIL_ENV_FROM_NAME
                value: Example
              - name: MAIL_ENV_ENCRYPTION
                value: tls
              - name: APP_URL
                value: snipe.example.com:443
              - name: MYSQL_PORT_3306_TCP_PORT
                value: "3306"
              - name: MYSQL_PORT_3306_TCP_ADDR
                valueFrom:
                  secretKeyRef:
                    name: snipe-database
                    key: host
              - name: MYSQL_DATABASE
                valueFrom:
                  secretKeyRef:
                    name: snipe-database
                    key: database
              - name: MYSQL_USER
                valueFrom:
                  secretKeyRef:
                    name: snipe-database
                    key: username
              - name: MYSQL_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: snipe-database
                    key: password
              ports:
              - containerPort: 80
              resources:
                requests:
                  cpu: 150m
                  memory: 512Mi
                limits:
                  cpu: 150m
                  memory: 512Mi
              volumeMounts:
              - mountPath: /var/lib/snipeit
                name: snipe-disk
              - mountPath: /var/www/html/storage/app/backups
                name: backup-disk
            - name: snipe-it-backup-sidecar
              image: imagename2
              env:
              - name: GCP_CREDENTIALS_LOCATION
                value: /tmp/gcpCredentials/creds.json
              resources:
                requests:
                  cpu: 50m
                  memory: 128Mi
                limits:
                  cpu: 50m
                  memory: 128Mi
              volumeMounts:
              - mountPath: /var/lib/snipeit
                name: snipe-disk
              - mountPath: /shared
                name: backup-disk
              - name: gcp-credentials
                mountPath: "/tmp/gcpCredentials/"
                readOnly: true
            volumes:
            - name: gcp-credentials
              secret:
                secretName: snipe-backup-sidecar
            - name: backup-disk
              emptyDir: {}

      
## GCloud Compute Scripts for an Active Directory Deployment
      export region=us-east1
      export zone_1=${region}-b
      export zone_2=${region}-c
      export vpc_name=winaddcnet
      export project_id=company-xyz-ldap

      gcloud config set compute/region ${region}
      gcloud config set project ${project_id}

      gcloud compute networks create ${vpc_name}  \
          --description "VPC network to deploy Active Directory" \
          --subnet-mode custom

      gcloud compute networks subnets create private-ad-zone-1 \
        --network ${vpc_name} \
        --range 10.250.250.0/28

      gcloud compute networks subnets create private-ad-zone-2 \
          --network ${vpc_name} \
          --range 10.250.250.16/28

      gcloud compute firewall-rules create allow-internal-ports-private-ad \
          --network ${vpc_name} \
          --allow tcp:88,135,389,445,464,636,3268,3269,49152-65535,udp:88,123,389,464,icmp \
          --source-ranges  10.0.0.0/8,172.16.0.0/12,192.168.0.0/16 \
          --target-tags=ad

      gcloud compute firewall-rules create allow-internal-ports-private-dns \
          --network ${vpc_name} \
          --allow tcp:53,udp:53,icmp \
          --source-ranges  10.0.0.0/8,172.16.0.0/12,192.168.0.0/16 \
          --target-tags=ad

      gcloud compute firewall-rules create allow-rdp \
          --network ${vpc_name} \
          --allow tcp:3389 \
          --source-ranges 35.235.240.0/20 \
          --target-tags=rdp

      #source ranges should be adjusted for each service that needs to reach out to the ingress ldap port.
      gcloud compute firewall-rules create allow-ingress-ldaps \
          --network ${vpc_name} \
          --allow tcp:636 \
          --source-ranges 1.2.3.4/32 \
          --target-tags=ldaps-ingress


      gcloud compute instances create ad-dc1 --machine-type n1-standard-2 \
          --boot-disk-type pd-ssd \
          --boot-disk-size 50GB \
          --image-family windows-2019 --image-project windows-cloud \
          --network ${vpc_name} \
          --zone ${zone_1} \
          --subnet private-ad-zone-1 \
          --private-network-ip=10.250.250.1

      gcloud compute reset-windows-password ad-dc1 --zone ${zone_1} --quiet

      gcloud compute instances create ad-dc2 --machine-type n1-standard-2 \
          --boot-disk-size 50GB \
          --boot-disk-type pd-ssd \
          --image-family windows-2019 --image-project windows-cloud \
          --can-ip-forward \
          --network ${vpc_name} \
          --zone ${zone_2} \
          --subnet private-ad-zone-2 \
          --private-network-ip=10.250.250.17

      gcloud compute reset-windows-password ad-dc2 --zone ${zone_2} --quiet


      #Configuring the Windows Domain Controller is separate from the GCP Configuration, so won't configure it in this guide.
      #Below will allow you to rdp from your local machine into the GCP proxy/tunnel.
      gcloud beta compute start-iap-tunnel ad-dc1 3389 \
          --local-host-port=localhost:53389 \
          --zone=${zone_1} \
          --project=${project-id}

      gcloud beta compute start-iap-tunnel ad-dc2 3389 \
          --local-host-port=localhost:63389 \
          --zone=${zone_2} \
          --project=${project-id}

## Networking and Software Equipment

| Make               	| Item/SKU                                                                 	| SKU                 	|  QTY  	|  Cost  	| Required?                 	|   	| Note                                                                                                                                                              	|   SUM   	|   Total Sum   	|
|--------------------	|--------------------------------------------------------------------------	|---------------------	|:-----:	|:------:	|---------------------------	|---	|-------------------------------------------------------------------------------------------------------------------------------------------------------------------	|:-------:	|:-------------:	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|    $63,746    	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	| Sectional Sum 	|
| Firewall           	|                                                                          	|                     	|       	|        	|                           	|   	| Firewall                                                                                                                                                          	|         	|    $19,814    	|
| Palo Alto Networks 	| Firewall PA-850                                                          	| PA-850              	|  2.00 	| $6,000 	| Yes                       	|   	|                                                                                                                                                                   	| $12,000 	|               	|
| Palo Alto Networks 	| Palo PANDb URL Filtering for PA-850                                      	| PAN-PA-850-URL4-HA2 	|  2.00 	| $1,500 	| No, recommended           	|   	|                                                                                                                                                                   	|  $3,000 	|               	|
| Palo Alto Networks 	| Palo Premium Support Program - extended service agreement 2 - 1 year     	| PAN-SVC-PREM-850    	|  2.00 	|  $907  	| No, recommended           	|   	|                                                                                                                                                                   	|  $1,814 	|               	|
| Palo Alto Networks 	| Palo Alto GlobalProtect Gateway for PA-850 - subscription license        	| PAN-PA-850-GP-HA2   	|  2.00 	| $1,500 	|                           	|   	|                                                                                                                                                                   	|  $3,000 	|               	|
| Palo Alto Networks 	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Switches           	|                                                                          	|                     	|       	|        	|                           	|   	| Switches                                                                                                                                                          	|         	|    $22,960    	|
| Juniper Networks   	| EX3400-48P                                                               	| EX3400-48P          	|  2.00 	| $4,000 	| Yes                       	|   	|                                                                                                                                                                   	|  $8,000 	|               	|
| Juniper Networks   	| EX-SFP-10GE-DAC-1M /                                                     	| 740-030076          	|  4.00 	|   $20  	| Yes                       	|   	|                                                                                                                                                                   	|   $80   	|               	|
| Juniper Networks   	| Juniper Care Next-Day - extended service agreement - 3 years - shipment  	| SVC-ND-EX3300-48P-3 	|  2.00 	| $1,290 	| No, recommended           	|   	|                                                                                                                                                                   	|  $2,580 	|               	|
| Juniper Networks   	| Juniper Care Core - technical support - 1 year                           	| SVC-COR-EX34-48     	|  2.00 	|  $150  	| No, recommended           	|   	|                                                                                                                                                                   	|   $300  	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Meraki             	| MS-390                                                                   	| MS390-48UX2         	|  2.00 	| $6,000 	|                           	|   	|                                                                                                                                                                   	| $12,000 	|               	|
| Meraki             	| 4 x 10G Uplink Module                                                    	| MA-MOD-4X10G        	|  2.00 	|  $750  	|                           	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Access Points      	|                                                                          	|                     	|       	|        	|                           	|   	| Access Points                                                                                                                                                     	|         	|     $8,130    	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	| Amount of WAPs will be dependent on Office Layout, Client QTY, and several other factors. Purchasing 6 as a minimum, but obviously the number will be a lot more. 	|         	|               	|
| Meraki             	| Cisco Meraki MR55                                                        	| MR55-HW             	|  6.00 	| $1,250 	|                           	|   	|                                                                                                                                                                   	|  $7,500 	|               	|
| Meraki             	| Cisco Meraki Enterprise Cloud Controller - subscription license (1 year) 	| LIC-ENT-1YR         	|  6.00 	|  $105  	|                           	|   	|                                                                                                                                                                   	|   $630  	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Security           	|                                                                          	|                     	|       	|        	|                           	|   	| Security                                                                                                                                                          	|         	|               	|
| Vecta              	| Brain/Controller                                                         	|                     	|  1.00 	|        	| Yes, for Cloud Deployment 	|   	|                                                                                                                                                                   	|         	|               	|
| Vectra             	| Sensor                                                                   	|                     	|  1.00 	|        	| Yes, for offices          	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Cables             	|                                                                          	|                     	|       	|        	|                           	|   	| Cables                                                                                                                                                            	|         	|     $2,307    	|
| Tripp Lite         	| Tripp Lite Cat6 Gigabit Snagless Molded Patch Cable (RJ45 M/M) Blue, 7'  	| N201-007-BL         	| 30.00 	|   $6   	| Yes                       	|   	|                                                                                                                                                                   	|   $180  	|               	|
| Tripp Lite         	| Tripp Lite 1000ft Cat6 Gigabit Bulk Cable Solid CMP Plenum PVC Gray TAA  	| N224-01K-GY         	|  4.00 	|  $362  	|                           	|   	|                                                                                                                                                                   	|  $1,448 	|               	|
| Tripp Lite         	| Tripp Lite Cat6 Gigabit Snagless Molded Patch Cable (RJ45 M/M) Blue, 10' 	| N201-010-BL         	| 20.00 	|   $7   	|                           	|   	|                                                                                                                                                                   	|   $140  	|               	|
| Tripp Lite         	| Tripp Lite Cat6 Gigabit Snagless Molded Patch Cable (RJ45 M/M) Blue, 15' 	| N201-015-BL         	| 20.00 	|   $9   	|                           	|   	|                                                                                                                                                                   	|   $180  	|               	|
|                    	| Tripp Lite Cat6 Gigabit Snagless Molded Patch Cable (RJ45 M/M) Blue, 5'  	| N201-005-BL         	| 60.00 	|   $6   	|                           	|   	|                                                                                                                                                                   	|   $359  	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Server             	|                                                                          	|                     	|       	|        	|                           	|   	| Server                                                                                                                                                            	|         	|    $10,000    	|
| Dell               	| R440                                                                     	| N/A                 	|  2.00 	| $5,000 	| Yes                       	|   	| Approximate value, as these will be custom spec'd, and does not have a specific SKU model.                                                                        	| $10,000 	|               	|
| Palo Alto Networks 	| Firewall PA-850                                                          	| PA-850              	|  2.00 	| $6,000 	| Yes                       	|   	|                                                                                                                                                                   	| $12,000 	|               	|
| Palo Alto Networks 	| Palo PANDb URL Filtering for PA-850                                      	| PAN-PA-850-URL4-HA2 	|  2.00 	| $1,500 	| No, recommended           	|   	|                                                                                                                                                                   	|  $3,000 	|               	|
| Palo Alto Networks 	| Palo Premium Support Program - extended service agreement 2 - 1 year     	| PAN-SVC-PREM-850    	|  2.00 	|  $907  	| No, recommended           	|   	|                                                                                                                                                                   	|  $1,814 	|               	|
| Palo Alto Networks 	| Palo Alto GlobalProtect Gateway for PA-850 - subscription license        	| PAN-PA-850-GP-HA2   	|  2.00 	| $1,500 	|                           	|   	|                                                                                                                                                                   	|  $3,000 	|               	|
| Palo Alto Networks 	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Switches           	|                                                                          	|                     	|       	|        	|                           	|   	| Switches                                                                                                                                                          	|         	|    $22,960    	|
| Juniper Networks   	| EX3400-48P                                                               	| EX3400-48P          	|  2.00 	| $4,000 	| Yes                       	|   	|                                                                                                                                                                   	|  $8,000 	|               	|
| Juniper Networks   	| EX-SFP-10GE-DAC-1M /                                                     	| 740-030076          	|  4.00 	|   $20  	| Yes                       	|   	|                                                                                                                                                                   	|   $80   	|               	|
| Juniper Networks   	| Juniper Care Next-Day - extended service agreement - 3 years - shipment  	| SVC-ND-EX3300-48P-3 	|  2.00 	| $1,290 	| No, recommended           	|   	|                                                                                                                                                                   	|  $2,580 	|               	|
| Juniper Networks   	| Juniper Care Core - technical support - 1 year                           	| SVC-COR-EX34-48     	|  2.00 	|  $150  	| No, recommended           	|   	|                                                                                                                                                                   	|   $300  	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Meraki             	| MS-390                                                                   	| MS390-48UX2         	|  2.00 	| $6,000 	|                           	|   	|                                                                                                                                                                   	| $12,000 	|               	|
| Meraki             	| 4 x 10G Uplink Module                                                    	| MA-MOD-4X10G        	|  2.00 	|  $750  	|                           	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Access Points      	|                                                                          	|                     	|       	|        	|                           	|   	| Access Points                                                                                                                                                     	|         	|     $8,130    	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	| Amount of WAPs will be dependent on Office Layout, Client QTY, and several other factors. Purchasing 6 as a minimum, but obviously the number will be a lot more. 	|         	|               	|
| Meraki             	| Cisco Meraki MR55                                                        	| MR55-HW             	|  6.00 	| $1,250 	|                           	|   	|                                                                                                                                                                   	|  $7,500 	|               	|
| Meraki             	| Cisco Meraki Enterprise Cloud Controller - subscription license (1 year) 	| LIC-ENT-1YR         	|  6.00 	|  $105  	|                           	|   	|                                                                                                                                                                   	|   $630  	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Security           	|                                                                          	|                     	|       	|        	|                           	|   	| Security                                                                                                                                                          	|         	|               	|
| Vecta              	| Brain/Controller                                                         	|                     	|  1.00 	|        	| Yes, for Cloud Deployment 	|   	|                                                                                                                                                                   	|         	|               	|
| Vectra             	| Sensor                                                                   	|                     	|  1.00 	|        	| Yes, for offices          	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Cables             	|                                                                          	|                     	|       	|        	|                           	|   	| Cables                                                                                                                                                            	|         	|     $2,307    	|
| Tripp Lite         	| Tripp Lite Cat6 Gigabit Snagless Molded Patch Cable (RJ45 M/M) Blue, 7'  	| N201-007-BL         	| 30.00 	|   $6   	| Yes                       	|   	|                                                                                                                                                                   	|   $180  	|               	|
| Tripp Lite         	| Tripp Lite 1000ft Cat6 Gigabit Bulk Cable Solid CMP Plenum PVC Gray TAA  	| N224-01K-GY         	|  4.00 	|  $362  	|                           	|   	|                                                                                                                                                                   	|  $1,448 	|               	|
| Tripp Lite         	| Tripp Lite Cat6 Gigabit Snagless Molded Patch Cable (RJ45 M/M) Blue, 10' 	| N201-010-BL         	| 20.00 	|   $7   	|                           	|   	|                                                                                                                                                                   	|   $140  	|               	|
| Tripp Lite         	| Tripp Lite Cat6 Gigabit Snagless Molded Patch Cable (RJ45 M/M) Blue, 15' 	| N201-015-BL         	| 20.00 	|   $9   	|                           	|   	|                                                                                                                                                                   	|   $180  	|               	|
| Tripp Lite         	| Tripp Lite Cat6 Gigabit Snagless Molded Patch Cable (RJ45 M/M) Blue, 5'  	| N201-005-BL         	| 60.00 	|   $6   	|                           	|   	|                                                                                                                                                                   	|   $359  	|               	|
|                    	|                                                                          	|                     	|       	|        	|                           	|   	|                                                                                                                                                                   	|         	|               	|
| Server             	|                                                                          	|                     	|       	|        	|                           	|   	| Server                                                                                                                                                            	|         	|    $10,000    	|
| Dell               	| R440                                                                     	| N/A                 	|  2.00 	| $5,000 	| Yes                       	|   	| Approximate value, as these will be custom spec'd, and does not have a specific SKU model.                                                                        	| $10,000 	|               	|