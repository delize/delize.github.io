---
author: Andrew Doering
comments: true
date: 2017-06-23 17:27:53+00:00
lastmodified: 2019-05-30 17:27:53+00:00
layout: post
link: https://andrewdoering.org/blog/2017/06/23/g-suite-enterprise-ca-list-for-smime-encryption/
slug: g-suite-enterprise-ca-list-for-smime-encryption
title: G Suite Enterprise - CA List for S/MIME Encryption
wordpress_id: 404
tags:
- G Suite
- S/MIME
- DigiCert
---

My company just recently enabled G Suite Enterprise on our Google Accounts in an attempt to provide a more seamless process for email encryption. We ran into a bit of an issue though, our certificate chain wasn't trusted. Throwing out this message when attempting to upload anything.




[![](https://andrewdoering.org/blog/wp-content/uploads/2017/06/2017-06-20_21-28-15.png)](https://andrewdoering.org/blog/wp-content/uploads/2017/06/2017-06-20_21-28-15.png)




This proved very odd to us, as the company uses Digicert as a provider, which is a very popular certificate authority. We got in touch with our Google Account Manager and Solutions Engineer/Sales Engineer to help get a point of reference for what was happening.




The Solutions Engineer had us go through the following steps (to be fair most of which we already tried):






  * Add root CA, intermediary CA, certificate chain in one export and  
attempt to upload.


  * Upload just certificate.


  * Provide a sample set  
certificate to give to Google




After going through this process and reaching the final step, he finally presented me with a list of CA's they pull from.  
  
I have to note that this is in no way a complete list. Knowing Google, I am sure they don't use a single source of information. However, this is is a great way to at least see any potential risks in terms of what you are going to be able to use prior to purchasing G Suite Enterprise.




[G Suite Enterprise - CA List for S/MIME](https://hg.mozilla.org/mozilla-central/raw-file/tip/security/nss/lib/ckfw/builtins/certdata.txt)




At the time of writing (2017/06/20) they supported the following Digicert certificates:



    
    Subject: C=US, O=DigiCert Inc, OU=www.digicert.com, CN=DigiCert Assured ID Root G2
    Subject: C=US, O=DigiCert Inc, OU=www.digicert.com, CN=DigiCert Assured ID Root G3
    Subject: C=US, O=DigiCert Inc, OU=www.digicert.com, CN=DigiCert Global Root G2
    Subject: C=US, O=DigiCert Inc, OU=www.digicert.com, CN=DigiCert Global Root G3
    Subject: C=US, O=DigiCert Inc, OU=www.digicert.com, CN=DigiCert High Assurance EV Root CA
    Subject: C=US, O=DigiCert Inc, OU=www.digicert.com, CN=DigiCert Trusted Root G4
    




This again, is not comprehensive. As you can judge by the file in question and the list, even though the "Digicert Assured ID Root CA" certificate is listed, it is not in Google's trusted store.




I would love to be able to pull the full list of certificates from Google, or be able to validate their current list in some way. I am still asking our Google SE to provide the full list, but, for now, there is at least some new information out there. As I was not able to find a tiny relation to this link when searching for issues that could help my cause when we first flipped the switch.




**Update 7/17**  
As of today, this still hasn't been accomplished, even through they have told me multiple times that the new certs would be imported. Which is definitely a bummer, though, I am not all that surprised. The reasoning that was given was:




`I asked about what the hold up has been and there was an internal fire drill that caused some delays in the binary releases.`




No idea what this means, so, thats a bummer, and while our SE has been helpful in keeping tabs on it, this information is somewhat vague.






**Update 2019/05/30**  
During the past year, Google has made some rather large feature sets into this, and it definitely works much better than it did during the time of writing. As mentioned below, if you create the DigiCert Full Chain CA along with your personal cert, you should have the ability to allow for DigiCert to be uploaded for use with S/MIME within G Suite.  
  
Though in the words of our InfoSec staff, it still isn't S/MIME.





















