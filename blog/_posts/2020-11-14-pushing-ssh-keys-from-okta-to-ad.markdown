---
title: Pushing SSH keys from Okta to AD
excerpt: When using SSSd with Linux, pushing SSH Keys from Okta to Active Directory, allowing for flexibility to push ssh keys to other services/systems. This also allows for SSH keys to be stored in Active Directory..
link: https://andrewdoering.org/blog/2020/11/14/pushing-ssh-keys-from-okta-to-ad 
author: Andrew Doering
published: true
comments: true
date: 2020-11-15 01:00:00 -0700
last_modified_at: 
layout: post
tags:
- ssh
- ssh keys
- okta
- active directory
categories:
- Identity Access Management
img_bg: /assets/blog/2020/11/push-ssh-keys/img_bg.jpg
img_bg_alt: Photo taken from unsplash by @bradfordnicolas, for use with SSH Key blog post by Andrew Doering.
img_blog: /assets/blog/2020/11/push-ssh-keys/img_blog.jpg
img_alt: Photo taken from unsplash by @bradfordnicolas, for use with SSH Key blog post by Andrew Doering.
permalink: /blog/:year/:month/:day/:title/
---
- [Introduction](#introduction)
- [Configuring SSSD on Linux to work with Active Directory](#configuring-sssd-on-linux-to-work-with-active-directory)
- [Modifying the Active Directory Schema](#modifying-the-active-directory-schema)
  - [Enable Schema Updates](#enable-schema-updates)
  - [Adding the attribute in AD schema](#adding-the-attribute-in-ad-schema)
  - [Adding a new class for the attribute](#adding-a-new-class-for-the-attribute)
  - [Associate the attribute and class to the user property](#associate-the-attribute-and-class-to-the-user-property)
- [Creating an attribute in Okta's User Directory Schema](#creating-an-attribute-in-oktas-user-directory-schema)
- [Adding an ssh key to Okta profile](#adding-an-ssh-key-to-okta-profile)
- [Add the new AD attribute to the Directories Profile in Okta](#add-the-new-ad-attribute-to-the-directories-profile-in-okta)
- [Pushing Attributes from Okta to Application](#pushing-attributes-from-okta-to-application)

## Introduction

I really would love to implement Okta Advanced Sever Access (ASA) to our server infrastructure in our environment. ASA has three downsides:

* Expensive ($50,000 minimum) depending on the amount of servers being deployed to 
* Requires extensive management approval
* Requires extensive security review from our team

Okta's LDAP interface also [does not support Linux/PAM officially](https://help.okta.com/en/prod/Content/Topics/Directory/LDAP-interface-limitations.htm), which appears to be (more or less) an artificial limit to push/advertise the ASA product.

So we need to find workarounds for this. This blog post covers details on how to do to 2 things:

* Expand the Active Directory schema to store and use SSH keys over LDAP.
* Allow for SSH Keys to be stored in Okta and pushed to Active Directory by the End User.

## Configuring SSSD on Linux to work with Active Directory

We want to configure SSSD on Linux to hook up into Active Directory (a very basic configuration file, edit it to your needs):

```text
    {% raw %}
    [sssd]
    domains = LDAP
    services = nss, pam
    config_file_version = 2

    [nss]
    filter_groups = root
    filter_users = root

    [pam]

    [domain/LDAP]
    id_provider = ldap
    ldap_uri = ldap://ldap.example.com
    ldap_search_base = dc=example,dc=com

    auth_provider = krb5
    krb5_server = kerberos.example.com
    krb5_realm = EXAMPLE.COM

    cache_credentials = true
    min_id = 10000
    max_id = 20000
    enumerate = False
    {% endraw %}
```

I would suggest reading through [Red Hat's documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/sssd-integration-intro#sssd-ad-proc) for validating the configuration. I won't go into SSSD more in-depth here. 

## Modifying the Active Directory Schema

We will need to modify the Active Directory schema to create an attribute called sshPublicKey.

### Enable Schema Updates

1. Open up an administrative command prompt session
2. Open up an administrative registry editor session by running `regedit`
3. Browse to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NTDS\Parameters`
![Registry Path](/assets/blog/2020/11/push-ssh-keys/en-schema-update1.png)
4. Add a new DWORD key that is called `Schema Update Allowed` with a value of `1`
![Schema Update Allowed](/assets/blog/2020/11/push-ssh-keys/en-schema-update2.png)
5. In the administrative session of the command prompt, run `regsvr32 schmmgmt.dll` to enable Schema Management MMC Plug-in and accept the prompt.
![Register module](/assets/blog/2020/11/push-ssh-keys/en-schema-update3.png)

### Adding the attribute in AD schema

I would highly recommend putting the project number/name in the description below, as well as what I have written.

1. Once the mmc window has been opened, add the schema module to the running mmc window. 
![Open MMC window](/assets/blog/2020/11/push-ssh-keys/add-attr-schema.png)
2. Expand the listing on the left hand side.
3. Right Click on `Attributes`, and then select `Create New Attribute`.
![Add new attribute](/assets/blog/2020/11/push-ssh-keys/add-attr-schema2.png)
4. Accept the warning that pops up and continue on.
5. For `Common Name`, enter `sshPublicKeys` in the field.
6. For `LDAP Display Name`, enter `sshPublicKeys` in the field.
7. For `Description`, enter `For Public SSH Key storage for sssd`.
8. For `Unique X500 Object ID` enter the following OID, `1.3.6.1.4.1.24552.1.1.1.13`.
9.  For `Syntax`, select `IA5-String`.
10. Check the `Multi-Valued` box.
11. Leave both `Minimum` and `Maximum` blank.
![Register module](/assets/blog/2020/11/push-ssh-keys/add-attr-schema3.png)
12. Click `Ok`.


### Adding a new class for the attribute

Once we have added the attribute into Active Directory, we need to add a class for the attribute to be associated with it. I would highly recommend putting the project number/name in the description as well.

1. Right-click on `Classes`, then click `Create class`.
2. If presented, click `Continue` on the warning.
3. For `Common Name`, enter `ldapPublicKey` in the text field.
4. For `LDAP Display Name`, enter `ldapPublicKey` in the text field.
5. For `Unique X500 Object ID`, enter `1.3.6.1.4.1.24552.500.1.1.2.0`.
6. For `Description`, enter `For use with sshPublicKeys attribute`. 
7. For `Parent Class` enter `top`.
8. For `Class Type`, select `Auxiliary`.
  ![Register module](/assets/blog/2020/11/push-ssh-keys/add-class-2.png)
9.  Click `Next` 
10. Under `Optional`, select `Add`, and scroll to find `sshPublicKeys`, click `Ok`.
  ![Register module](/assets/blog/2020/11/push-ssh-keys/add-class-3.png)
11. Click `Finish` and leave the Schema MMC window open.

### Associate the attribute and class to the user property

1. With the schema window still open on `Classes`, find the `user` class 
  ![Schema window](/assets/blog/2020/11/push-ssh-keys/associate-class-1.png)
2. Right-click select on `properties`.
  ![Properties value](/assets/blog/2020/11/push-ssh-keys/associate-class-2.png)
3. Select the `Relationship` tab, and click `Add Class...`
  ![Relationship Tab](/assets/blog/2020/11/push-ssh-keys/associate-class-3.png)
4. Select `ldapPublicKey` and click `Ok`.
  ![Selecting ldapPublicKey class](/assets/blog/2020/11/push-ssh-keys/associate-class-4.png)
5. Select the `Attributes` tab and click `Add` under `Optional`.
  ![Associate attribute tab](/assets/blog/2020/11/push-ssh-keys/associate-attr-1.png)
6. Select `sshPublicKey` and click `Ok`.
  ![Associate attribute tab](/assets/blog/2020/11/push-ssh-keys/associate-attr-2.png)
7. Click `Apply` and then click `Ok`
8. Proceed to close the MMC window.

## Creating an attribute in Okta's User Directory Schema

We will begin to create the attribute under Okta's Universal Directory.

1. Go to [`https://yourdomain.okta.com/admin/universaldirectory`](https://yourdomain.okta.com/admin/universaldirectory)
2. Select `Profile` under `User (default)`
  ![Okta Universal Directory](/assets/blog/2020/11/push-ssh-keys/okta-step1.png)

3. Select `Add Attribute`
  ![Okta Profile](/assets/blog/2020/11/push-ssh-keys/okta-step2.png)

4. Once the `Add Attribute` window has appeared, fill out the prompts with the following options:
  ![Okta step 3](/assets/blog/2020/11/push-ssh-keys/okta-step3.png)

   * Display Name: `SSH Public Key`
   * Variable Name: `sshPublicKey`
   * Description: Add what you would like here, an example is below in the photograph
   * Once done, select `Save`

5. Once the attribute is created, scroll down to the bottom, and find the newly created Okta attribute and click the pencil icon to edit the attribute.
  ![Okta step 4](/assets/blog/2020/11/push-ssh-keys/okta-step4.png)

6. You will want to change the attributes to match to the picture below.
  ![Okta step 5](/assets/blog/2020/11/push-ssh-keys/okta-step5.png)

  The reason for this is to allow user's to modify the contents of the attribute value, and to not have a profile master potentially override or own the attribute contents.

## Adding an ssh key to Okta profile

1. Go to [`https://yourdomain.okta.com/enduser/settings`](https://yourdomain.okta.com/enduser/settings)
2. Select `Edit Profile`
  ![Edit Profile](/assets/blog/2020/11/push-ssh-keys/okta-profile-add-key1.png)
3. If prompted enter your pasword, if not proceed to the next step.
4. Click `Edit`
  ![Edit](/assets/blog/2020/11/push-ssh-keys/okta-profile-add-key2.png)
5. Within the `[]`, add your SSH Key wrapped in `"`. An example of this would be shown below.
  ```
  ["ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEAoi9tCoNOkZgrOpEm2aoVY3uWNUq3MvNSHrgC9r2ELyVpODAz7eCFswszhtYB2o1FPqarJxJq3QQjflhEAZ16o7oaMD8kWzTdGMBjy9vynCr9dMQTuWoEBlAFNsjK1xdaJyWM2sEFV7p6c85yxeeDei1wBPc1AA4X9H2uS4ZTjaNm/Zobe7j7q8lc/e2Sb0tvY0auLv1sRRScxgZFQ5X/uMK0VtcubXxWxh6JceOb4BZRmHDCXOmX3z3wevtuDw6udafyZ6sjowSFH+PD+p7V97m9S81mQAuXfXzgmd/LrRlxHuzx0DpHKbi623lDvWWNb9QJwlLKbfMEP/DGPiEdsw== example-key"]
  ```
  ![Profile with ssh key](/assets/blog/2020/11/push-ssh-keys/okta-profile-add-key3.png)
6. Select `Save`

From here, if you need to add new keys, simply put them in a comma delimited format. So multiple keys would look like `["key 1", "key 2", "key 3"]`.

## Add the new AD attribute to the Directories Profile in Okta

One crucial step in allowing us to push the content down is to add the attribute from Active Directory so that Okta is aware of it.

1. Go to [`https://yourdomain.okta.com/admin/universaldirectory`](https://yourdomain.okta.com/admin/universaldirectory) and select `Directories`, then select your domain.
  ![Okta Universal Directory - Profile Editor](/assets/blog/2020/11/push-ssh-keys/okta-to-ad-step1.png)
2. Click `Profile`, once the page finishes loading, select `Add Attribute`, and then search for `sshPublicKey`
  ![Okta Universal Directory - Profile Editor](/assets/blog/2020/11/push-ssh-keys/add-ssh-attr-to-okta.png)
3. Click `Save`

## Pushing Attributes from Okta to Application

Before doing this, announce to your users will use a change. This could be a breaking change if SSH Keys are not saved into the user's Okta profile. We will use Active Directory as the example application here, but this could be used in other ways.

1. Go to [`https://yourdomain.okta.com/admin/universaldirectory`](https://yourdomain.okta.com/admin/universaldirectory) and select `Directories`, then select your domain.
  ![Okta Universal Directory - Profile Editor](/assets/blog/2020/11/push-ssh-keys/okta-to-ad-step1.png)
2. Click `Mappings`, and then select `Configure User Mappings`
  ![Okta Universal Directory - Configure Mappings](/assets/blog/2020/11/push-ssh-keys/okta-to-ad-step2.png)
3. Select the `Okta User to domain.com`
  ![Okta to User](/assets/blog/2020/11/push-ssh-keys/okta-to-ad-step3.png)
4. Scroll down to your `sshPublicKey` attribute on the right column and search for `sshPublicKey` on the left column. 
  ![sshPublicKey Push](/assets/blog/2020/11/push-ssh-keys/okta-to-ad-step4.png)
5. Validate with a user that has saved SSH Keys to their Okta profile, before clicking save. Once validated, select `Save mapping`.

From here, your user's SSH keys should be in a multi value format pushed down to Active Directory. Note, there are two values here, as I added the SSH key twice to the Okta profile to show the multi-valued aspect.
![Finished Result](/assets/blog/2020/11/push-ssh-keys/ssh-keys-added-to-ad.png)

From here, you could use tools like Ansible, SSSD, and others, to pull the content/value out of Active Directory for use with other services and systems. This could also be done directly to the LDAP Interface on Okta, however, that is not officially supported.