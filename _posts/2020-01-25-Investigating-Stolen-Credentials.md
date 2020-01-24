---
title: 'Weaponized Defense: Investigating potentially stolen creds'
date: 2020-01-25 20:45
categories:
  - blog
tags:
  - graylog
  - Windows
  - Logging
  - Incident Response
published: false
---

# Weaponized Defense
## Can you spot stolen credentials in your network?

You wake up.  You grab your coffee, sit down, quickly scan your favourite headline feed and catch a glimpse of a corporate business partner splashed all over the headlines.  Their **everything** was jacked straight off their precious servers, or from a mishap, ransomware, open S3 bucket, whatever.  Either way, you know you have a LOT of accounts from your domain on that platform.

Some call it a nightmare from hell.  The rest of us call it Tuesday. 

Regardless of your experience in this scenario, you know there are a lot of Kyle@yourdomain.com and Karen@yourcorp.net with their suspiciously strong passwords of `Password!#` all over that single-factor hot mess.  Nobody ever suspects the [octothorpe](https://www.lexico.com/en/definition/octothorp) at the end of a password.  Nobody.  It's definitely not in the list of 1 billion hashes per sec my 3-year laptop can rip through hashcat. 

Let's see if anyone is already perusing the inboxes of those precious Kyles and Karens shall we?  

## The required Logs

I'm going to use an On Premises Exchange and Graylog for this example.  Obviously, the concept is the same for any web-based login page.  Here is what you should be logging for any authentication that faces the internet at a *bare minimum* to have any investigative or alerting value: 

*  Authentication Pass / Fail
*  At least 2 identifiers (and the more the merrier):
    * Username
    * Source IP
    * User Agent String

### The toolbox for this setup

* Graylog
    * GeoIP lookup configured
* Exchange
    * [IIS Logging on](https://docs.microsoft.com/en-us/iis/manage/provisioning-and-managing-iis/configure-logging-in-iis)
        * With [X-Forwarded-For in your logs](https://support.kemptechnologies.com/hc/en-us/articles/360002861712-Adding-The-X-Forwarded-For-Header-and-Configuring-IIS-Logging) if required.  Thanks for the write-up, @KempTechnologies
* Dirty internet rodents trying to steal your cheese

## Starting with a User

Step 1:  Long tail

If you choose a user


>I see **SO MANY** folks configure logging but without injecting the IP address during a NAT transversal.  IF your source logs say 10.blah.bloh.bleh when you know the traffic is coming from the internet, go yell at the router / switch / load balancer / nGinX / [Firewall](https://isitthefirewall.net) person to enable X-Forwarded-For!

