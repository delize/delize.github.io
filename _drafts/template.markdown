---
title: Placeholder Title
excerpt: Transitioning from legacy Active Directory to a cloud centric HRIS, Okta, and Push System. This blog post will go over all workflow process of transitioning, this does not go into low level details on how to transition due to the environment that this took place in. 
link: https://andrewdoering.org/blog/2020/09/20/transitioning-to-okta-from-active-directory-new-directory-service-infrastructure1/
slug: transitioning-to-okta-from-active-directory-new-directory-service-infrastructure1
author: Andrew Doering
published: false #Remove this after finishing the document
comments: true
date: 2020-09-20 18:20:49 PT
layout: post
tags:
- Active Directory
- bamboohr
- hris
- infrastructure
- meraki
- office
- okta
blogsingle-backgroundimage: https://images.wallpapersden.com/image/download/huawei-4k-stock-abstract_66336_1920x1080.jpg
img_bg: /assets/blog/2020/10/
img_bg_alt: Placeholder Alt text
img_blog: /assets/blog/2020/10/
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


At $currentcompany, we have traditionally been using Active Directory to master our Okta groups and services. However, managing Active Directory (correctly) is a pain, and doesn't scale well when you need to set them up in different offices. What I am going to go over here, is what our project plan was, what it improved, what tools we used and our diagram architecture, and where it still needs improvement.

This is being written ~3 to 5 months after changes have been performed, so there may be some spotty information or less than specific information due to the time that has occurred between performing the changes and writing this guide.

## Existing Infrastructure

Previously at $currentcompany, we had 6 Active Directory Domain Controllers, two in a single office in the AMER, two in an office in EMEA, and two in a physical Data Center environment. Initially, the EMEA office did not exist, as we expanded into this area in the beginning of 2019. I inherited the instances in AMER and the Physical Data Center in 2016. There was always some issues going through the two locations, usually dealing with replication or DNS environments. Even after providing [this](https://support.microsoft.com/en-us/help/179442/how-to-configure-a-firewall-for-domains-and-trusts) document to our networking team to open up ports and services in the firewall, if I ever ran diagnostics on the AD servers we would still see errors.  

When we deployed the EMEA office location, we experienced more problems, this time, mainly due to the major delay in synchronization and geographical distance and the lack of reliability on the line in the EMEA office. Again, no matter the amount of research and testing, we did - we always experienced issues when running diagnostics across our cluster of Domain Controllers. Nothing we did really resolved the AD issues. 


The issues we experienced were:

  * 24 hour difference between password synchronizations when AD was mastering Okta, and users had to change their password in Okta
  * Since users did not have VPN access, users could not change their local laptop password to be in sync with the AD instance
  * Attributes were not updating properly with in-house tools randomly, and no one was updating the in-house tools requiring IT team versus the owners of the tool to try and update them
  * $8,000 servers being purchased and being under utilized for a single instance of each: AD server, Bind9, and isc-dhcp-server
  * Onboarding (Employee Creation) was being done manually versus automated, inducing a lot of errors for new IT employees setting up users
  * General headaches (Microsoft Licensing, Annoyances, Lack of built in robust change monitoring)
  * Lack or modernization of Active Directory On-Premise 

Personally, a big part of this (in my opinion) was due to the lack of communication between our servers and what was classified as our "Primary Domain Controller", which was not directly reachable by others in the network due to "security restrictions" around our Data Center environment.

I had also been trying to make these changes for several years, but, due to a lack of "ownership" and not having specific permissions on several different systems, it made it much more difficult to get approval until more recently. 

## Investigating next steps


While a possibility we did look at, was deploying an Active Directory Server into Google Cloud Platform, or Amazon Web Services, ultimately, we wanted to move away from Active Directory entirely within the organization and move to a more cloud centric/first environment. 

As we already were using Okta as our SSO/Identity Provider for SAML access, this seemed like the most appropriate choice. Okta provided:

  * [Cloud LDAP Interface](https://help.okta.com/en/prod/Content/Topics/Directory/LDAP-interface-main.htm) with robust security measures
  * The ability to use [Enhanced Group Push](https://help.okta.com/en/prod/Content/Topics/Directory/ad-agent-cofigure-group-push-AD-OUs.htm), to go from Okta to Active Directory (where/when necessary, Early Access)
  * [HRIS provisioning system for user accounts](https://saml-doc.okta.com/Provisioning_Docs/BambooHR_Provisioning.html)
  * APIs for programatic changes and modifications
  * [Terraform Provider](https://registry.terraform.io/providers/oktadeveloper/okta/latest/docs)
  * Seamless integration of [Meraki RADIUS with Okta](https://help.okta.com/en/prod/Content/Topics/integrations/cisco-meraki-radius-intg.htm) (Early Access)


When looking at others, the options were:


  * [JumpCloud](https://jumpcloud.com/)
    * Compatibility with [Radius](https://support.jumpcloud.com/support/s/article/configuring-a-cisco-meraki-wap-to-jumpclouds-radius-as-a-service1-2019-08-21-10-36-47)
    * [Non-official support](https://github.com/hashicorp/terraform/issues/16049) of [terraform](https://github.com/CognotektGmbH/terraform-provider-jumpcloud)
    * Provides SAML authentication
  * [Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-whatis)
  * [FoxPass](https://www.foxpass.com/)

For options, JumpCloud honestly stood out as the most robust product we could potentially use (that did not involve Microsoft related products), but, having to pay for two services JumpCloud and Okta (we were not ready to migrate all of our SAML applications out of Okta). It would have been a harder sell to our Finance team and more work in the long run.  We didn't want to continue on with Microsoft Products unless required for Windows machines or Office 365 Licensing.

## Planning our Workflows and Processes


We wanted several requirements from this new system namely:


  * Minimal to Zero Touch Onboarding Process (ATS > HRIS > IdP/IAM)
  * Automation internally for group memberships 
  * Go from "role" based memberships to department/team based memberships
  * Migrate from "mastered AD groups" in Okta, to "native Okta groups"
  * Help us set being the process around Device Trust
  * Migrate from Active Directory Radius to Cloud Based Radius
  * Allow Infrastructure team, and other services that previously used AD to integrate and use Okta to authenticate
  * Prevent unauthorized changes to group memberships from team members, while still allowing them to manage processes/products that they own
  * Implement logging and change alerting based on actions performed in Okta
  * Cloud-first centric workflows/processes, and cloud-first mentality
  * Future planning for terraform'ed Okta

As you can see, this is quite an extensive list, even without everything being completely planned out and the goals changing multiple times, and a list of final requirements was never fully defined due to a lack of time commitment/communication from other teams when asking for requirements . More stuff popped up during this transition and there were delays caused by co-workers - but this was the majority of what we were trying to fix and improve.

While we wanted to automate the Application Tracking System (ATS) <> Human Resources Information Systems (HRIS) system, there was some concern from HR about doing this. While I don't personally understand their concerns (they never actually listed them),  and as IT doesn't have access to the ATS generally - we left the integration piece for now to come back to it at a later time. You can find the instructions here on how to integrate [Greenhouse](https://support.greenhouse.io/hc/en-us/articles/201177624-BambooHR) and [BambooHR](https://help.bamboohr.com/hc/en-us/articles/216831767-Greenhouse) (the systems we use/used at $currentcompany).


## Phase 1 - Setting up BambooHR and Okta Integration

I want to start out with saying that the [documentation](https://saml-doc.okta.com/Provisioning_Docs/BambooHR_Provisioning.html) that Okta has is excellent.


We had already configured and setup Okta as an IdP with BambooHR for SSO/SAML before we were doing these changes, however we were not using BambooHR as a profile master/provisioning feature. We wanted to be very careful with this process, since changing our profile master in production would be very risky, as well as overwriting any attributes/values that belonged to the employee


### Requirements:

  * BambooHR Service Account with the appropriate permission level to see attributes that you want imported
  * Okta Service Account with the appropriate permission level to connect the OIDC integration 

Our Service Account in Okta looks like this:

![](https://andrewdoering.org/blog/wp-content/uploads/2020/08/Screen-Shot-2020-08-07-at-8.57.06-PM-1024x656.png)You will notice that we have a User Type called "Service Account", that basically just minimizes any potential attributes so that the user cannot be used with various services. Whatever you want to call the service account will work.

Our Service Account in BambooHR looks like this:

![](https://andrewdoering.org/blog/wp-content/uploads/2020/08/Screen-Shot-2020-08-07-at-9.05.40-PM-1024x588.png)My recommendation here, is to use the exact same email/uid as the Okta Service Account. Mainly to simplify things.

We discussed in depth with our HR team on what information we wanted to import into Okta so we could potentially utilize that information into our services/systems. At a bare minimum, we wanted the following from the HRIS system in Okta:


|         Attribute Name        |      Attribute Key      |
|:-----------------------------:|:-----------------------:|
|            Username           |         userName        |
|           Work Email          |        workEmail        |
|           Best Email          |        bestEmail        |
|           First Name          |        firstName        |
|           Last Name           |         lastName        |
|            Address            |         address1        |
|              City             |           city          |
|            Country            |         country         |
|           Department          |        department       |
|            Division           |         division        |
|        Employee Number        |      employeeNumber     |
|          Display Name         |       displayName       |
|           Job Title           |         jobTitle        |
|            Location           |         location        |
|          Middle Name          |        middleName       |
|          Mobile Phone         |       mobilePhone       |
|            Nickname           |         nickname        |
|             State             |          state          |
|           Supervisor          |        supervisor       |
|         Supervisor ID         |       supervisorId      |
|           Work Phone          |        workPhone        |
|   Work Phone Plus Extension   |  workPhonePlusExtension |
|            Zip Code           |         zipcode         |
|         Address Line 2        |         address2        |
|          BambooHR ID          |        bamboohrId       |
|             Laptop            |          Laptop         |
|            Manager            |         Manager1        |
|           Superhero           |        Superhero        |
|           Birth Date          |       dateOfBirth       |
|    Employment Status: Date    |    employeeStatusDate   |
|    Employment Status – FTE    | employmentHistoryStatus |
|           Hire Date           |         hireDate        |
|           Home Email          |        homeEmail        |
|           Home Phone          |        homePhone        |
|          LinkedIn URL         |         linkedIn        |
|      Job Informatin: Date     |          p4047          |
|        Termination Type       |          p4343          |
|       Termination Reason      |          p4344          |
|         Preferred Name        |      preferredName      |
|        Termination Date       |     terminationDate     |
|          Twitter Feed         |       twitterFeed       |
|         Work Extension        |    workPhoneExtension   |
|             Gender            |          gender         |
| Security Training – Completed |          p4326          |
|  Security Training – Expires  |       p4326_u002E1      |
|  Security Training – Due Date |       p4326_u002E2      |
|          Reporting To         |           p91           |




With the above fields from BambooHR, we can do the following:

  * If we used BambooHR's Security Training modules/tasks, we could restrict all access to applications until the security training was completed. Alternatively, if this was a configurable field, we could utilize the API from our current security training vendor, and have it auto populate/mark the field as completed to then allow the employee to access their applications.
  * We can have the employees select their preferred name. There is however an issue with this. By default, BambooHR provides a single attribute field called "Preferred name" - but this does not specify First, Last, or both. We have several employees with bi- and tri-nomial  names (Example: two first names, two last names) or several people that have non-conjugated names (Example: Smith, John Lee / John Lee Smith or Lee Smith, John / John Lee Smith). When you have these issues with these names, it is impossible to come up with a common naming scheme. This also creates general problems with application provisioning, as some applications that we used were using First + Last Name fields, or using the Display Name field to populate this information. To make things more challenging, our HR team needs/wanted to have their full legal name in the HRIS system with no exceptions.  
  
**Resolution to naming issues:**

What we came up with with our HR team was the ability to incorporate a "Preferred First Name" and "Preferred Last Name", that way HR can keep their full legal name in the "First Name" and "Last Name" fields, while employees can self-service their own names in the Preferred name fields. Our HR team also didn't like this because the risk of employees being malicious was too high. Ultimately, I believe that you should treat the employees like adults and monitor name changes, I don't think this would cause any issues in the longer term, if it does - you take individual action with that person.  You can read our post mortem when we completed this phase below.   

  * We are aware of the employee's start date, and termination date for automation workflows, as well as promotion dates, department/teams, etc. This allows us to automate the entire onboarding process from beginning to end, including their team memberships when transitioning between departments or teams. As well as on/off board them where necessary.
  * Automate some of our survey data, applications to a much greater extent, automation tools we code ourselves, and also use Okta Workflows more efficiently. 


Before implementing this, InfoSec had an issue with us pulling specific attributes, and allowing Application Administrators, User Administrators, Group Administrators seeing those attributes. Thankfully, Okta has an [Early Access feature called Sensitive Attributes](https://help.okta.com/en/prod/Content/Topics/users-groups-profiles/usgp-hide-sensitive-attributes.htm). What this done is effectively not allow administrators below the Super Admin level, to be able to see the attribute value. I would HIGHLY recommend turning this on, it's super useful and so far (our team) has not found any downside to this. It also removes any risks from the end user of Okta seeing sensitive data as well. 


### Sandbox Testing


From here, we setup the integration to occur in our Okta Sandbox (OktaS) & BambooHR Sandbox (BHRS) environment initially, we imported all users from the BHRS into OktaS, this went fine without any issues. We then started to add users from into AD initially to implement them into Okta, and then added them to BHRS to have pre-existing information into OktaS. We did not see any major issues during the initial sandbox testing phase. However the scope of what were testing was limited compared to our production environments.


### Production Implementation

We discussed with both our InfoSec team as well as our HR team on appropriate time frames and ranges to have accounts onboarded before hand, as well as when they should be offboarded. Our Okta Configuration looks like the following:


![](https://andrewdoering.org/blog/wp-content/uploads/2020/08/Screen-Shot-2020-08-08-at-12.43.28-AM-1024x552.png)You can see that we are using Departments, and importing those Departments as Groups into Okta. While we had some debate, we initially elected for 3 days prior to the new employees start date to create their account. However, this meant employees starting on a Monday (or Tuesday) would have their account created too late (as we had not switched off of AD yet).  We elected for 5 days instead.


![](https://andrewdoering.org/blog/wp-content/uploads/2020/08/Screen-Shot-2020-08-08-at-12.45.52-AM-1024x454.png)We wanted to make sure that any updates that occurred to employees would occur multiple times throughout the day, our HR team was concerned about making these changes too frequently, however for a company the size of ~500 people, I don't see much of an issue moving this down from 6 hours to 1 - 3 hours. Our username format is meant to always be email address across the entire company.


![](https://andrewdoering.org/blog/wp-content/uploads/2020/08/Screen-Shot-2020-08-08-at-12.46.04-AM-1024x504.png)
We discussed and wanted to enable all features here - mainly to make sure that minimal fuss and touch would be applied to any accounts going forward.

![](https://andrewdoering.org/blog/wp-content/uploads/2020/08/Screen-Shot-2020-08-08-at-12.46.14-AM-1024x671.png)Due to employees potentially leaving and coming back into the company, we wanted to reactivate suspended Okta Users and reactivate deactivated Okta users.  The safeguards here are honestly very dependent on your needs. We did not modify the original settings here and 20% is the general safeguard for Okta applications.   
However, if you are using BambooHR as the sole profile master, and you are deactivating accounts if something occurs at the application level (the employee gets removed from the App, BambooHR gets deactivated, etc) 20% of your employees will be deactivated causing you a massive headache when this occurs. My recommendation would be to set the percentage of your employee base to 20 people or less.


Now you should be ready to run an import from BambooHR into Okta, you will likely have to confirm some users after the import runs. Since BambooHR was appended to the Profile Master hierarchy, you will notice that all users still have AD listed in the "Mastered by:" field. 







What I would suggest at this point, is creating Okta groups based on the Department Groups that we specified would be created in the Provisioning Section. The requirements for these groups will obviously differ for every company, internally we use XXXX, XYYY, XYZY numerical formatting for our groups. Meaning that those that are their team groups are not included in high level department groups. So if a teams numeric number is 1610, 1620, 1630 the only way to manage all these employees at once would be to create a 16XX group. This way you are inclusive to all teams. Then set up Group Rules to mirror the memberships from the BambooHR mastered Groups into Okta mastered groups, and to converge all team groups up into the high level department group. Since we can't use non-Okta mastered groups for certain features of Okta, this helps us work around that.







After this is complete, and you have finished any validation, you should be ready to move on to Phase 2.







## Phase 2 - Import all Active Directory Attributes







When I state import all attributes, I mean ALL. Make sure you import all the data you need. We were using several custom attributes within our Active Directory schema - we needed these to be a part of Okta's default Profile Editor / User Profile schema. 







To do this, make sure that you create attributes in the User Profile Type first, before you try and import the attributes from Active Directory. As an example:





![](https://andrewdoering.org/blog/wp-content/uploads/2020/08/Screen-Shot-2020-08-07-at-10.56.16-PM-1024x945.png)





(Unfortunately, we have already disabled Active Directory Profile Mastering - so I can't provide photos here) 







At this point, you should be able to map the attribute from Active Directory into Okta, as I said, I would recommend migrating in ALL attributes that you might even think are not necessary. 







## Phase 3 - Migrate membership and application assignments from Active Directory groups and data to Okta groups and data  







This would have been a fairly arduous and manual task - however, I made this a lot easier (even though the code looks like, well honestly, shit). Use [this python script from Github](https://github.com/delize/ad_to_okta/blob/master/ad-to-okta-migration.py). Unfortunately, my mindset was just getting and testing the API when working with it, not writing production level code. It is currently being reworked to be more clear. While you will need to remove the comment out from the last 8 lines - what this does is:







  * Create Okta groups from existing Active Directory groups in Okta
  * Create Okta Group Rules based on existing Active Directory Group Names to link with Okta Group Names
  * Activate all Group Rules to copy group membership from AD to Okta
  * (Optionally) Deactivate all Group Rules so that group membership no longer is updated on changes to groups
  * Append and assign newly created Okta groups, to applications where AD groups are assigned 
  * (Optionally) Remove the user from the Active Directory Application
  * (Optionally) Send a Password reset email to the employee's Work and Home emails






We thought these were the most important tasks that should be performed during our migration (maybe not all migrations, but at least ours). Looking back at this now, I noticed we did not think about the "Enhanced Push Groups" functions for applications, and something we should have written into our script - and I am happy to accept PRs to the code above (once it is improved). This might have been due to me not being able to find the API call to create, modify, or delete the push groups function.







While I cannot show you in detail what the groups will look like (there will be too much redacting). The group names will be the exact same name of the AD group, and a Group Description of `AD2OKTA-API - ` followed by the Domain path. Group members should be mirrored from the original group.







We wanted to allow for a period of time for memberships to update from AD. However, we didn't want this enabled permanently - as we had a general problem of employees updating group memberships without letting people know or getting approval (this was due a Director+ employee requesting Domain Admin to his direct reports hierarchy). We gave a weeks time before re running the script to disable all Group Rules. 







Application assignments will be appended with the new groups at the bottom, after the existing group assignments. They will have the exact same configuration as the previously assigned groups for the application if it was assigned with an Active Directory group. So for example in the `Atlassian Cloud` application in Okta (which encompasses Jira and Confluence) when looking at the API for "List Assigned Groups":  
  







    
 
    {
            "id": "0F0F0F0F0F0F0F0F0F0F",
            "lastUpdated": "2019-10-02T23:45:35.000Z",
            "priority": 1,
            "profile": {
                "preferredLanguage": "English",
                "timezone": null,
                "organization": "currentcompany",
                "department": "IT"
            },
            "_links": {
                "app": {
                    "href": "https://currentcompany.okta.com/api/v1/apps/01010101010101010101"
                },
                "self": {
                    "href": "https://currentcompany.okta.com/api/v1/apps/01010101010101010101/groups/0F0F0F0F0F0F0F0F0F0F"
                },
                "group": {
                    "href": "https://currentcompany.okta.com/api/v1/groups/0F0F0F0F0F0F0F0F0F0F"
                }
            }
        },







The updated entry will appear like:






    

    {
            "id": "0A0A0A0A0A0A0A0A0A0A",
            "lastUpdated": "2020-04-02T23:45:35.000Z",
            "priority": 36,
            "profile": {
                "preferredLanguage": "English",
                "timezone": null,
                "organization": "currentcompany",
                "department": "IT"
            },
            "_links": {
                "app": {
                    "href": "https://currentcompany.okta.com/api/v1/apps/01010101010101010101"
                },
                "self": {
                    "href": "https://currentcompany.okta.com/api/v1/apps/01010101010101010101/groups/0A0A0A0A0A0A0A0A0A0A"
                },
                "group": {
                    "href": "https://currentcompany.okta.com/api/v1/groups/0A0A0A0A0A0A0A0A0A0A"
                }
            }
        },






We can see that the priority has changed but the profile or configured options stay the same, and the new group id has been applied.







Or as another example, for Office 365, if this is the AD group that is assigned (as shown from the API and edited)






    

    {
           "id": "FFFFFFFFFFFFFFFFFFFF",
           "lastUpdated": "2020-04-13T21:38:26.000Z",
           "priority": 1,
           "profile": {
               "licenses": [
                   "Azure Active Directory Premium P1 - Azure Active Directory Premium P1",
                   "Azure Active Directory Premium P1 - Azure Multi-Factory Authentication",
                   "Office 365 Business - O365 Business"
               ],
               "roles": null
           },
           "_links": {
               "app": {
                   "href": "https://currentcompany.okta.com/api/v1/apps/0b0b0b0b0b0b0b0b0b0b"
               },
               "self": {
                   "href": "https://currentcompany.okta.com/api/v1/apps/0b0b0b0b0b0b0b0b0b0b/groups/FFFFFFFFFFFFFFFFFFFF"
               },
               "group": {
                   "href": "https://currentcompany.okta.com/api/v1/groups/FFFFFFFFFFFFFFFFFFFF"
               }
           }
       },







This will be the new group assignment:






    

    {
           "id": "00000000000000000000",
           "lastUpdated": "2020-04-13T21:38:26.000Z",
           "priority": 4,
           "profile": {
               "licenses": [
                   "Azure Active Directory Premium P1 - Azure Active Directory Premium P1",
                   "Azure Active Directory Premium P1 - Azure Multi-Factory Authentication",
                   "Office 365 Business - O365 Business"
               ],
               "roles": null
           },
           "_links": {
               "app": {
                   "href": "https://currentcompany.okta.com/api/v1/apps/0b0b0b0b0b0b0b0b0b0b"
               },
               "self": {
                   "href": "https://currentcompany.okta.com/api/v1/apps/0b0b0b0b0b0b0b0b0b0b/groups/00000000000000000000"
               },
               "group": {
                   "href": "https://currentcompany.okta.com/api/v1/groups/00000000000000000000"
               }
           }
       },







Notice the updated priority and the new group / id references. The "profiles" dict stays the same, copying the exact same information over.







While we provide the option to remove the user from Active Directory entirely, I ultimately decided against doing it this way - as we have not completely deactivated Active Directory from our environment (as our deadline to deprecated AD is November of this year). However with everything done above, we can move on to the following phases.







The same goes for the password reset emails. Since we were eventually going to turn off delegated authentication, it didn't make much sense from an experience perspective - there was no reason to have them reset their password twice. We did not run this portion of the utility in production, but did test in our OktaS.







## Phase 4 - Switch the Profile Master from Active Directory to BambooHR







We initiated this phase by informing our employees that were making changes that would streamline our onboarding process by changing the "Source of Truth" of where our data is coming from, but this wouldn't affect the majority of user's information or cause any problems. 







We navigated to [this page](https://yourcompany-admin.okta.com/admin/profile-masters), and moved BambooHR from position 2 to position 1. We then went to the BambooHR Application we had in our Okta instance and run a full import.







We did not modify the Profile Master section for at least a few weeks before continuing on to phase 5 of removing Active Directory as a Profile Master.







As noted in the issues section, Names that have extended sections, or more than a single first and single last name presented issues. This was the ONLY issue that presented itself and you can read our full documentation and post-mortem on the changes that we sent to HR below:


> ### Name Mapping in BambooHR and Okta
> BambooHR Defaults and Custom Name Fields
> By default, BambooHR comes with the following fields to capture a person’s name:
> 
> * First Name  
> * Middle Name  
> * Last Name  
> * Preferred Name
> 
> In most cases, the above is sufficient. The “Preferred Name” can be set to anything (for instance, more than just a single word), however, as of May 5th, 2020, it is typically used to capture the preferred first name, such as a shortened name (i.e. Alexander → Alex).
> 
> There are employees with extended, combined, dual or shortened names who usually would like to request a reasonable “simplified” or “shortened” version of their full legal names. For example, consider a hypothetical person with the following full name: “Alexandre Juarez Juan-Castillo Alfonso”, where the legal first name is officially specified as “Alexandre Juarez” and legal last name is set as “Juan-Castillo Alfonso”. Suppose now that commonly, this person goes by “Alex Castillo”, with “Alex” being the preferred first name and “Castillo” being the preferred last name. It is impossible in this case to push the preferred name outside of the HR system with only the default fields available in BambooHR. At the same time, the legal requirement of keeping the full legal name on file in the HR system must be met.
> 
> In order to accommodate both sides, two additional custom fields can be setup in BambooHR:
> 
> * Preferred First Name  
> * Preferred Last Name
> 
> These will distinguish between what is legally required to be captured by the HR system (using the default fields of BambooHR) and the requested version to accommodate the requester’s commonly used short name (using the additional custom fields setup in BambooHR). To be fully clear and explicit, the above employee would be entered as follows inn BambooHR:
> 
> * First Name: Alexandre Juarez  
> * Middle Name:  
> * Last Name: Juan-Castillo Alfonso  
> * Preferred Name:
> 
> * Preferred First Name: Alex  
> * Preferred Last Name: Castillo
> 
> Notice that the original “Preferred Name” field is redundant. As it is a BambooHR default field, it cannot be changed/removed, however, it can be hidden and not used generally. The new custom “Preferred First/Last Name” fields, however, can be either hidden or visible to the general employee. It is suggested that they are set to be visible and editable, subject to verification and approval of the people operations team.
> 
> ### Name Sync with Okta and Services Provisioned by Okta
> Okta will pull the names from BambooHR in the logical manner described below. Note that the name in Okta and the majority of services and apps in Okta will be reflected depending which of these fields will be set in BambooHR.
> 
> * If “Preferred First” and “Preferred Last” name fields are both specified, they will be treated as official first and last names on Okta and all its services. For the above example, the employee will see Alex Castillo as their name outside of BambooHR.  
> * If “Preferred First” name field is empty and “Preferred Last” name field is specified, Okta will pull BambooHR’s “First Name” field as the first name and “Preferred Last” name field as the last name. In the above example, the employee will see Alexandre Juarez Castillo as their name outside of BambooHR.  
> * If “Preferred First” name field is specified and “Preferred Last” name field is empty, Okta will pull “Preferred First” name field as the first name and BambooHR’s “Last Name” field as the last name. In the above example, the employee will see Alex Juan-Castillo Alfonso as their name outside of BambooHR.  
> * If “Preferred First” and “Preferred Last” name fields are both unspecified, Okta will pull BambooHR’s “First Name” and “Last Name” fields as first and last names, respectively. For the above example, the employee will see Alexandre Juarez Juan-Castillo Alfonso as their name everywhere.
>
> ### The Case of the “displayName” Attribute
> Both BambooHR and Okta have a user attribute called displayName. The two do not necessarily have to be the same. In case of Okta, certain apps and services are provisioned with this attribute for the names as opposed to the separate first and last names. The IT team will create logic on the Okta side so that Okta’s displayName attribute matches exactly in the manner described in the above section. In case of BambooHR, the exact functionality of this attribute is described in detail on BambooHR admin help documentation called Display Name. IT recommends that this value is set to {First Name} {Last Name} pair on BambooHR to align with legal name requirements of the HR system.
> 
> 







## Phase 5 - Turn off Profile Master from Active Directory

First off, inform your product owners that any changes made in Active Directory will no longer update Okta with 2 to 4 weeks notice. Even today, and after three announcements and 4 months after our change - we still have employees making changes in Active Directory. You can't solve everything, but, still make the announcement. If you have done imports correctly (by importing everything) in Phase 2, you should be good to go and have nothing to worry about here. That is, assuming that you don't have any applications that import data into AD that is needed to push into Okta. 

You will also need to disable scheduled imports in Active Directory Provisioning Tab To Okta.  
  
Since here at $currentcompany, we mainly have cloud only services and have a cloud first mindset - we don't have to worry about this.

## Phase 6 - Remove delegated authentication

This was the most concerning portion of our transition. This is a disruptive change. While it was suggested that it could be possible to Update Users with a password hash, the Okta API only allows for creation and not update of ONLY the password value (at least not without also pushing the rest of the profile in the same PUT request), at least as far as I could research with the public API. In the long term, I also want to make sure that employees are starting with a "clean state" here.

Before doing this, I would recommend announcing a few things to end users and making a few changes and performing a few tasks on the Admin side.

### End Users

  * Have them update or verify their Security Question - if they can't remember their security question you should assist them using this API

### Administrator Dashboard

  * Update the Self Service time frame and the length of emails to be 24 to 48 hours (or the time frame that you see fit) - we did not do this and set the Self Service time frame to an hour - would not recommend. 
  * Create a new password policy around your companies policies and any new Okta groups that have been created
  * Create an API Token to use Postman as a backup in case all Admin users get locked out of the Administrator Dashboard
  * Have the following Postman Collections prepared to use
    * [Find User](https://developer.okta.com/docs/reference/api/users/#find-users) - `{% raw %}{{url}}{% endraw %}/api/v1/users?q={% raw %}{{SEARCHSTRING}}{% endraw %}`
    * [Unlock User](https://developer.okta.com/docs/reference/api/users/#unlock-user) - `{% raw %}{{url}}{% endraw %}/api/v1/users/{% raw %}{{userId}}{% endraw %}/lifecycle/unlock`
    * [Reset Password ](https://developer.okta.com/docs/reference/api/users/#reset-password)- `{% raw %}{{url}}{% endraw %}/api/v1/users/{% raw %}{{userId}}{% endraw %}/lifecycle/reset_password?sendEmail=true`
    * [Set Recovery Question](https://developer.okta.com/docs/reference/api/users/#set-recovery-question-answer) - `{% raw %}{{url}}{% endraw %}/api/v1/users/{% raw %}{{userId}}{% endraw %}` 


[//]: # (Hello)



<!-- end of the list -->
  


            {
              "credentials": {
                "recovery_question": {
                  "question": "What is a Kid's Cartoon and listens to directions?",
                  "answer": "Dora the Explorer"
                }
              }
            }


Note that this needs to be in the body of the PUT request. Annoyingly, if you use the word "The" in your question, you can't use the word or any variation of "The" in your answer. 


We gave employees a months notice, with an announcement every week on both Slack and Email. When we made the change, we setup a two day SLA response expectation based on timezone and support availability.  
  
We were making the changes on a Tuesday at 12 PM Pacific Time. This obviously has some issues, as the time period in Europe would be 9PM GMT at the earliest. We also only have Support Teams in London, Austin, and San Francisco. Meaning that our timezone availability is limited in the APAC region.

We allowed for a 5 hour time frame for immediate response and live chat via a Slack Channel beginning at 11:30 AM where people could come in with issues (the most common of these being Security Question, Account State Issues, Password Reset Emails being expired/Resending Password Reset). This 5 hour window also allowed us to get the employees we have located in the earliest APAC region (Japan and Australia).  We don't have many employees in other areas of APAC so it was less of an issue. The majority of our employees even in EMEA performed the password resets when they were sent an email.   
  
Then when our support employee started work in the EMEA region, his first 5 hours were dealing with issues pertaining to the Authentication changes as well. 

Overall, I am happy with how this was executed - and there was minimal issues from end users because of the long-term advanced notice. While we didn't have many complaints, the complaints that did come in were mainly from our most vocal user base because they couldn't remember their security question or that their account was in a locked out state because their account did not have a password.

## Phase 7 - Push membership, attributes, values, and groups from Okta into Active Directory

Things to be aware of before making this change:

  * Any service/software that uses Common Name values (as these could have possibly changed based on the Phase 4)
  * Any memberships or users that were manipulated in Active Directory after the Scheduled Imports and Profile Mastering was disabled
  * If Active Directory is listed as a profile master, you will not be able to perform any of the steps below


With this, you will want to utilize [Enhanced Group Push](https://help.okta.com/en/prod/Content/Topics/Directory/ad-agent-cofigure-group-push-AD-OUs.htm) with Active Directory to link the Okta groups that were created from the transitioning phase, to the existing Active Directory Groups where expected. Primarily, this will only be for groups that the company wants to blanket management.

The steps below to provision users from Okta into Active Directory are copied directly from [Okta's site](https://support.okta.com/help/s/article/How-to-sync-users-from-Okta-to-Active-Directory?language=en_US). Note that these must be done for each Active Directory Domain you want to provision users into.

The following details how you can use Okta to create users in AD:

  1. In the Okta Admin Console, navigate to **Admin > Directory > Groups**
  2. Create a new group in Okta (You also can use an existing Okta Group).
  3. Add Okta users that you want to be pushed to AD to the group created above.
  4. Open the group in Okta and click on **Manage Directories** button.
  5. Select the target Active Directory instance and then click **Next**.
    * NOTE: Your AD integration must have this OU selected in the **"Import and Provisioning" **section in order for Okta to create the AD users, as seen below:

![User-added image](https://support.okta.com/help/servlet/rtaImage?eid=ka01Y0000010yec&feoid=00N0Z00000GpsjR&refid=0EM1Y000000xYEh)

  1. Select the target OU where the users should be created in
  2. Click on **Confirm Changes**.

All users in the group will be pushed and synced to your Active Directory. If you add new users to the group, Okta will also push them automatically.

**Notes:**

  * The **Create Users** option must be enabled under the Active Directory settings in Okta, in order to push and create new users from Okta in Active Directory:

![User-added image](https://support.okta.com/help/servlet/rtaImage?eid=ka01Y0000010yec&feoid=00N0Z00000GpsjR&refid=0EM1Y000000Ethy)

  * The service account used by the Okta AD agent needs to either be a domain admin, or have permissions to make changes (creating users, update etc.) to your Active Directory. Otherwise, you will receive errors while trying to sync users from Okta to AD.
  * For detailed info regarding Active Directory integration with Okta, please refer to [Install and configure the Okta Active Directory (AD) agent](https://help.okta.com/en/prod/Content/Topics/Directory/ad-agent-install.htm) documentation.

While the steps above, I would recommend using your department based groups from the Phase 1 rollout, that way you know that the entire department/team will be have access to the AD Environment if needed. If a smaller subset is required you can revert to the team or department groups from the Phase 1 integration. We also decided to push attributes and other information so that employees had a single place to update and manage these details. You can see an example of our configuration below:

![](https://andrewdoering.org/blog/wp-content/uploads/2020/08/image-836x1024.png)





For mappings from Okta to AD, these are highly dependent on your needs and what you would like to push. While I am not able to share our mappings, for both `distinguishedName` and `objectGUID` these are listed as unmapped and not configured. 







From here, I would recommend going through the managed Directories/Groups being used multiple times, just to validate that you have not left any details out.







From this point, you should be able to start Phase 8 if it is required, otherwise Phase 9.







## Phase 8 - Decode Base64 encoded values from Okta







In case you find any [attributes encoded in Base64](https://support.okta.com/help/s/question/0D50Z00008OJKeX/custom-ad-attribute-pushing-into-okta-as-a-the-base64-encoding-of-the-value?language=en_US) values from your AD import (even though they shouldn't be) in Phase 2, you can use [this python tool](https://github.com/delize/Okta_decode_b64) to decode to specify the attribute you want decoded along with your. Note that as of now (I plan to update this in the future to include a dry run flag or doit flag), you will need to change the python code to specify the option to actually run the functions against the host. The script was written for a specific purpose, and may not fit all needs - but it helped decode our attributes that had been in a bork'ed state for employees again. 







I am curious as to why not only restarting agents or performing full imports didn't correct this. You would think that over the span of time that we did this, Okta's AD Agent would have recognized and corrected the Base 64 attribute issue, but apparently this is still a problem in 2020. Unfortunately, I don't know enough about how Okta internally works to make a further diagnosis on why it was still base64 encoded. It could have been that attributes initially recognized from an older time frame cause this, but then upgrading the agent and performing a full import - should have resolved that, so yeah, a little clueless as to the why, but the end result is that they are no longer encoded.







## Phase 9 - Migrate Meraki/Wi-Fi authentication from AD to EAP-TTLS with Okta







With Corona Virus causing all of our employees to Shelter In Place/Work From Home, this has been a lesser degree of involvement but given us a lot more time to work on it properly (and unfortunately, not yet finished). If we had to do this with employees in the office at the time, this would have been moved up in the list and would have been a prerequisite before proceeding with Phase 2, and executed the change within a few days of performing Phase 6.







For this, again with a cloud first mentality - we wanted to deploy the Radius Agent using Docker. Unfortunately, Okta doesn't have an official docker image of their Radius Agent, and I was unable to find any personal docker images. We have not built it ourselves yet (due to our recent announcement and closure of $acquiringcompany acquisition of $currentcompany).  In the mean time, we deployed a GCP VM with minimal requirements to host the RADIUS Agent. You will need to setup a firewall rule within GCP to enable outside access into the RADIUS Agent. My recommendation would be to use a more open approach to the Radius Agent initially before locking down the allowed IP ranges that communicate with the inbound port. You can then apply the radius tag to the servers that will host the Radius Agent. An example of this can be seen below:





![](https://andrewdoering.org/blog/wp-content/uploads/2020/08/image-1-638x1024.png)





From this point, you should be able to follow Okta's[ instructions on the Meraki Interoperability Feature](https://help.okta.com/en/prod/Content/Topics/integrations/cisco-meraki-radius-intg.htm#Part1), this is a fairly straight forward guide that does not need a lot of description on. The hardest part from our testing was not over engineering this and just keeping it simple. We use Meraki MR5X series devices, and did not experience any initial issues on a single AP. Since we are not in the office yet, this might have some issues when we scale it up - but we have not been able to test this adequately yet.







## Summary







We now have a lot more flexibility in our services and there are some more room for improvements.







For instance, we can utilize [Okta2Anything](https://github.com/pmcdowell-okta/okta2anything) to create a proxy to allow our DB systems and other services, if Okta LDAP Interface can't be used or you need more/managed reliability to the instance. 







If  you still need Active Directory, you can create "on the fly"/ad-hoc Active Directory domains with Terraform, to spin up random instances of Active Directory. Note, that the Active Directory Domain HAS to have a unique name for you to spin up users in an "on the fly" manner. If the Domain Name is the same, Okta will treat it as a single domain, even if the AD Domain Controllers are not in a cluster or joined together. The idea behind this is that you can scope specific users for certain requirements, limiting the scope of the user base and entry points. This utilizes the Enhanced Group Push service from Okta.







Automate Administrators with Okta Groups, unfortunately, this is restricted where groups that have group rules applied to them - can't be automatically used to grant administrator access. The main idea was to grant our InfoSec team Read Only access across the board when they were first hired. 







Things we could have done better:







  * Validate Common Name/Display Name changes before pushing them from Okta into AD
  * Be aware of the name controversy from BambooHR to Okta that caused some annoyance to a small subset of individuals (less than 15)






We have greatly removed our reliance on Active Directory at this point, and we no longer have to worry about or maintain it. As the only tools that our office infrastructure needs for integration is Okta. While we won't tear down Active Directory immediately (mainly due to SIP/WFH environments), this puts us in a better position to deprecate it. 







This also means, that any infrastructure that requires Active Directory, we can push users into and our DevOps / SRE teams can now manage the AD instance in the way they see fit, with a base group membership that can be managed when/where needed. This prevents owners not being aware of changes being made to the system without change management in place.







Our DevOps/SRE team has some concerns due to Okta not having Service Level Agreements without [paying $100,000 USD for 99.99% uptime](https://www.okta.com/blog/2020/07/99-point-99-percent-uptime-for-all/) (this was the initial response we received). While a valid concern, when looking at [multiple](https://status.okta.com/) [statistics](https://status.okta.com/#history) from Okta on their availability, I would not be overly worried. We have also begun testing Okta's endpoints with our own Product to validate the SLA times for our own needs.







Admittedly, the one thing that really annoys me, is that Okta does not provide a company hosted/non-self managed Okta RADIUS Interface. This seems like it would be a no-brainer to provide to your customers. It honestly boggles my mind.







## Questions?







If you have any questions, feel free to leave a comment below (just note, I don't always pay attention to those) or come by the  [#okta Slack Channel](https://macadmins.slack.com/archives/C0LFP9CP6) on [MacAdmin](https://www.macadmins.org/). There is a super helpful group of people located there that can assist you with this or any other Okta related issues.







EDIT: I was asked to remove all references to [$currentcompany](http://thousandeyes.com/) and [$acquiringcompany](https://www.cisco.com/) due to "reason". Hopefully this suffices, as there was no endorsements in this to begin with, but complied. 



