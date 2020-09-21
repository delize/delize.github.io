---
author: Andrew Doering
comments: true
date: 2019-12-11 04:24:00 PT
layout: post
link: https://andrewdoering.org/blog/2019/12/10/macos-using-nudge-to-deploy-major-and-minor-os-updates/
slug: macos-using-nudge-to-deploy-major-and-minor-os-updates
title: macOS - Using nudge to deploy major (and minor) OS updates
wordpress_id: 548
comments: true
tags:
- Catalina
- Erik Gomez
- nudge
- OS Updates

img_bg: https://images.wallpapersden.com/image/download/huawei-4k-stock-abstract_66336_1920x1080.jpg
img_bg_alt: Placeholder Alt text
img_blog: /assets/img/2019-12-11-post/nudge800x650.jpg
img_alt: Image about using nudge on macOS for updates. Intro image.
---






Having users update that do not have administrator permissions in macOS is a pain. Thankfully there is a cool (not new, but not well known) tool out called nudge (by Erik Gomez). You find the tool [here on Github](https://github.com/erikng/nudge).







I will be going over how we are implementing this at ThousandEyes.







## Prerequisites







  * GCP Bucket (because it is free!)
  * JSON template
  * Software Deployment Tool (JAMF or Munki, we use Munki)
  * Catalina Installer (we need to grab the icons from somewhere)






## Let's get started!







### Fork the Github Code to your own repo







In case something happens to the source, always be sure to fork the repo to your own. You can always pull updates and push into your fork.







### Examining the template file and the resources folder







Erik was very considerate enough to already provide the template file in the fork, however, there is some modifications to make here.







If we take a look at the fork we will notice a few specific things. Mainly that there is a nice "resources" folder located [here](https://github.com/erikng/nudge/tree/3901ca478b32a323924f7b443826c4ead61c3a60/payload/Library/Application%20Support/nudge/Resources). This means that we can indepently modify both the company logo, as well as the update_ss.png content after deployment without recompiling the entire application. In addition to this, if you look below:






    
    <code>{
        "preferences": {
            "button_title_text": "Ready to start the update?",
            "button_sub_titletext": "Click on the button below.",
            "cut_off_date": "2018-12-31-00:00",
            "cut_off_date_warning": 3,
            "days_between_notifications": 0,
            "logo_path": "/path/to/company_logo.png",
            "main_subtitle_text": "A friendly reminder from your local IT team",
            "main_title_text": "macOS Update",
            "minimum_os_version": "10.14.0",
            "more_info_url": "https://google.com",
            "no_timer": false,
            "paragraph1_text": "A fully up-to-date device is required to ensure that IT can your accurately protect your computer.",
            "paragraph2_text": "If you do not update your computer, you may lose access to some items necessary for your day-to-day tasks.",
            "paragraph3_text": "To begin the update, simply click on the button below and follow the provided steps.",
            "paragraph_title_text": "A security update is required on your machine.",
            "path_to_app": "/Applications/Install macOS Mojave.app",
            "screenshot_path": "/path/to/update_ss.png",
            "timer_day_1": 600,
            "timer_day_3": 7200,
            "timer_elapsed": 10,
            "timer_final": 60,
            "timer_initial": 14400,
            "random_delay": false,
            "update_minor": false,
            "update_minor_days": 14
        },
        "software_updates": [{
            "name": "091-22861",
            "force_install_date": "2018-12-31-00:00"
        }]
    }</code>







We are also able to specify custom paths for the company logo and the screenshot path. So we don't necessarily need to constantly replace/overwrite the files listed in here and recompile the application every time a major OS is released.   
  
We can simply modify the config json file, time after time, with changes to:







  * `screenshot_path` - the image that shows in the middle of the nudge UI
  * `logo_path` - the company logo (hopefully this doesn't change too often)
  * `more_info_url` - the documentation page that you should provide employees about the OS update
  * `cut_off_date` - the deadline you want employees to upgrade by
  * `minimum_os_version` - the minimum MAJOR OS Version that you would like to apply






However, within this template we need to make a few more changes.







Everything from `"software_updates": [{` to `}]` needs to be removed. Also be sure to remove the comma(`,`) next to the `}`.  This is due to the code changing, and this becoming deprecated in a previous version of the software. We also need to remove the comma since we no longer have a second array in the configuration file.







Now, in addition to the above, we need to add several things depending on your configuration (I assume you are using a management software to roll out the OS upgrade). Platform independent changes are going to be adding a minimum sub os version (`minimum_os_sub_build_version`) and a value (in our case we used `19B88` - based on the sub build version from [here](https://developer.apple.com/news/releases/?id=10292019c)).







#### JAMF specific configuration







I just want to be clear, we don't use JAMF at ThousandEyes, so this configuration may be wrong - I am basing the information here on the readme.md file, as well as general knowledge of what I can find in the JAMF's documentation.







If you would like users to launch the UI page in the Jamf Self Service tool, for them to perform the upgrade at a conveient time:  
`"local_url_to_upgrade": " jamfselfservice://content?entity=policy&id=<id>&action=view `







If you would like users to automatically initiate the upgrade as soon as they hit the button:  
`"local_url_to_upgrade": " jamfselfservice://content?entity=policy&id=<id>&action=execute "`







Now with both of the methods above, you would need to replace the `<id>` with the actual ID of the policy. Since we don't run JAMF at ThousandEyes, not sure if this is a numeric value or a string. 







#### Munki specific configuration







We do however, use Munki at ThousandEyes. So I can give you a more specific/detailed explanation here. Unfortunately, what we are doing here is very straight forward. As ThousandEyes 







#### ThousandEyes Example Template






    
    <code>{
        "preferences": {
            "button_title_text": "Ready to start the update?",
            "button_sub_titletext": "Click on the button below.",
            "cut_off_date": "2020-1-15-00:00",
            "cut_off_date_warning": 20,
            "days_between_notifications": 0,
            "logo_path": "/Library/Application Support/nudge/Resources/company_logo.png",
            "main_subtitle_text": "A friendly reminder from your local IT team",
            "main_title_text": "macOS Update",
            "minimum_os_version": "10.15.1",
            "minimum_os_sub_build_version": "19B88",
            "more_info_url": "https://knowledgebase.documentation.com",
            "no_timer": false,
            "paragraph1_text": "A fully up-to-date device is required to ensure that IT can your accurately protect your computer.",
            "paragraph2_text": "If you do not update your computer, you may lose access to some items necessary for your day-to-day tasks.",
            "paragraph3_text": "To begin the update, simply click on the button below and follow the provided steps.",
            "paragraph_title_text": "A security update is required on your machine.",
            "path_to_app": "/Applications/Install macOS Catalina.app",
            "screenshot_path": "/Library/Application Support/nudge/Resources/update_stg.png",
            "timer_day_1": 600,
            "timer_day_3": 3600,
            "timer_elapsed": 10,
            "timer_final": 300,
            "timer_initial": 14400,
            "random_delay": false,
            "update_minor": true,
            "update_minor_days": 14,
            "local_url_for_upgrade": "munki://detail-Install_macOS_Catalina"
        }
      }
    </code>







I left the defaults for some of the content, as to be honest, I don't know if I could come up with any better wording. However, I did change the timer numbers - I want to maximize the amount of popups but without the too much annoyance. If we leave too much a gap, users will just ignore it. 







Omitting our own knowledgebase article url, include yours. Ours is quite slim, as this tool has made it far easier to upgrade. 







While `path_to_upgrade` should/would be ignored since we are using `local_url_for_upgrade`, we left it in as a backup in case. 







`local_url_for_upgrade` is pointing directly to our Catalina installation. If you don't modify the contents when using `munkiimport` this should make perfect sense as to why we used "Install_macOS_Catalina" (it's the default on importing). Within munki, we setup our pre-requisites to require our AntiVirus software prior to installation of Catalina. The reason for this is that in the past Kernel Panics have been a problem. Because of this, we prefer to setup the latest version of the AV as a pre-req.







`minimum_os_sub_build_version` if this is not included you end up running into this [bug](https://github.com/erikng/nudge/issues/24). This key is required (as of now). 







Now let's move on to the LaunchAgent configuration. 







### nudge LaunchAgent







[https://github.com/erikng/nudge/blob/3901ca478b32a323924f7b443826c4ead61c3a60/payload/Library/LaunchAgents/com.erikng.nudge.plist](https://github.com/erikng/nudge/blob/3901ca478b32a323924f7b443826c4ead61c3a60/payload/Library/LaunchAgents/com.erikng.nudge.plist)






    
    <code><?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
    	<key>Label</key>
    	<string>com.erikng.nudge</string>
    	<key>LimitLoadToSessionType</key>
    	<array>
    		<string>Aqua</string>
    	</array>
    	<key>ProgramArguments</key>
    	<array>
    		<string>/Library/Application Support/nudge/Resources/nudge</string>
    		<string>--jsonurl=https://fake.domain.com/path/to/config.json</string>
    	</array>
    	<key>RunAtLoad</key>
    	<true/>
    	<key>StandardOutPath</key>
    	<string>/Library/Application Support/nudge/Logs/nudge.log</string>
    	<key>StandardErrorPath</key>
    	<string>/Library/Application Support/nudge/Logs/nudge.log</string>
    	<key>StartCalendarInterval</key>
    	<array>
    		<dict>
    			<key>Minute</key>
    			<integer>0</integer>
    		</dict>
    		<dict>
    			<key>Minute</key>
    			<integer>30</integer>
    		</dict>
    	</array>
    </dict>
    </plist></code>







In addition to this, there are some personal recommendations I would make for this based on feedback we had from our employees (they were pissed on our initial roll out of [UMAD](https://github.com/erikng/umad)). While I agree that on the 0 and 30 (every half hour) is a good plan, unfortunately, our company is small enough that I will get complaints and an ear full. Because of this, 







Now, if you remember that config.json file from earlier, and you notice that default option here is `--jsonurl=` you might see where this is headed. Making an updatable and flexible preferences file to load into nudge (while also allowing for flexible image locations as well).







### Using GCP







Google Cloud Platform has a [free tier](https://cloud.google.com/free/) for 5GB storage buckets. This _may_ not work well for you if you have over 10000 employees all hitting the service at the same time over a month or more of time. As the limitations of GCP Free tier are outlined below (as of 2019/12/02)







  * 5 GB-months of regional storage (US regions only)
  * 5,000 Class A Operations per month
  * 50,000 Class B Operations per month
  * 1 GB network egress from North America to all region destinations (excluding China and Australia) per month






The definition of the "classes" can be found [here](https://cloud.google.com/storage/pricing#operations-pricing). However for our needs, this works fine. The service is cheap enough, the operating costs are minimal, and it just works for our service. 







S3 would also be an alternative here as well, we just did not use it at ThousandEyes internally for IT.







We will be granting access based on permissions here: https://cloud.google.com/storage/docs/access-control/lists#scopes  
Since there is nothing private or personal here, and google defaults to HTTPS sites with no authentication it works great for our usecase.







To do this, create a storage bucket, call it something relevant to you, we called ours `nudge-os-release-notification` and then place your config.json file in the bucket. Once you create this bucket, while you could and should have this authenticated, the easiest way for you to begin testing is to set the member `allUsers` to the `Storage Object Viewer` role, this creates a public bucket, where anyone could download the contents of the files within the bucket. Not great for production, but perfectly fine for one-off testing.







### STG & PROD







What happens if we want to test deployment of a minor OS Update. We have a stg and production version of this application, where we apply the production version to all machines, and then apply stg to specific ones. This is not revolutionary in anyway.







## Ways to improve this







While we have only implemented an initial version of this software:







  * Instead of placing the file (either the logo, or the download icon) on the machine, why not allow them to be displayed on demand (similar to the json file, grab them from the internet and then keep them in a cache location). Then remove after use (or upgrade). This allows us to be flexible enough with the files, while still not having to redistribute the original nudge package (or single file packages) for every OS Upgrade revision.






## Major thanks to Erik







I want to give a major thanks to Erik, ThousandEyes IT has been using several of the tools he has developed over the past few years and it has made our job much easier, allowing us to develop and work on other things that pop up within the company.



