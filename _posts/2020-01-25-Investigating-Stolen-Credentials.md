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
published: true
---

# Weaponized Defense
## A 60-second spot check for stolen credential activity

### tl;dr Use known behaviour data to find anomalies

**Story time:** You wake up.  You grab your coffee, sit down, quickly scan your favourite headline feed and catch a glimpse of a corporate business partner splashed all over the headlines.  Their **everything** was jacked straight off their precious servers, or from a mishap, ransomware, open S3 bucket, whatever.  Either way, you know you have a LOT of accounts from your domain on that platform.

Some call it a nightmare from hell.  The rest of us call it Tuesday. 


Regardless of your experience in this scenario, you know there are a lot of Kyle@yourdomain.com and Karen@yourcorp.net with their suspiciously strong passwords of `Password!#` all over that single-factor hot mess.  Nobody ever suspects the [octothorpe](https://www.lexico.com/en/definition/octothorp) at the end of a password.  Nobody.  It's definitely not in the list of 1 billion hashes per sec my 3-year laptop can rip through hashcat. 

Let's see if anyone is already perusing the inboxes of those precious Kyles and Karens shall we?  

## The Logs for this setup

I'm going to use an On Premises Exchange and Graylog for this example.  Obviously, the concept is the same for any web-based login page.  Here is what you should be logging for any authentication that faces the internet at a *bare minimum* to have any investigative or alerting value: 

*  Authentication Pass / Fail
*  At least 2 identifiers (and the more the merrier):
    * Username
    * Source IP
    * User Agent String
* The ability to filter out Authenticated from Unauthenticated Traffic
    * Remove all IIS responses except 200
    * Remove Authentication Failed
    * Remove Internal Traffic

##### Note 2

### The Toolbox for this setup

* Graylog
    * GeoIP lookup configured
* Exchange
    * [IIS Logging on](https://docs.microsoft.com/en-us/iis/manage/provisioning-and-managing-iis/configure-logging-in-iis)
        * With [X-Forwarded-For in your logs](https://support.kemptechnologies.com/hc/en-us/articles/360002861712-Adding-The-X-Forwarded-For-Header-and-Configuring-IIS-Logging) if required.  Thanks for the write-up, @KempTechnologies
* Dirty internet rodents trying to steal your cheese

## Start with pretty circles

Filter your logs according to the list above.  You can focus in on a single user or just start hunting for oddball things.  I always start with a Geo-Map.  It is far from bullet-proof but it is usually **extremely** obvious right off the bat when credentials are out of line when you view traffic geographically. 

This user lives in Toronto and has not recently travelled to the United States.  Notice the dot over by Los Angeles (that's to *the left* for my fellow non-Americans)

![](/assets/images/2020-01-24-17-24-11.png)

So.  I see an angry red dot. 
Better yet - I break out the various GEO fields when I bring them in.  If you look at the image of the map you can see I use the format `client_` to indicate a client-side connection field.  I also have client_city which leads to this easy Quick Value screen as well showing actual numbers for an easy [long tail analysis](https://en.wikipedia.org/wiki/Long_tail): 

![](/assets/images/2020-01-24-17-29-03.png)

## Move to the Ugly Words

My next favourite tool in hunting for credential abuse is the useragent string.  Of course it is easy to fake, but you have to do a lot of work to fake the 'footprint' of a user.  The more you log, the more this is true.  What makes this nice, is that big ugly user agent strings actually work in your favour instead of parsing them out. 

Take this login of stolen credential: 
![](2020-01-24-17-40-21.png)

## Nope.  Never.  Not from a windows box.
This particular user is more likely to find the perfect pumpkin spice latte than they are to be caught dead using a windows machine.  2% of these logins were from somewhere else and we can begin the hunt.  

Another example:
![](2020-01-24-17-52-44.png)

## Its OK to be Canadian

Nobody is SOOO Canadian that they own a BlackBerry in 2020.  


## For one more look

Follow the numbers.  We have some pretty dedicated folks around here, but we definitely don't have anyone writing down all of their emails folder by folder at 4am.

![](/assets/images/2020-01-24-17-58-35.png)


# Closing Remarks

Some of this is pretty old hat to my fellow log and SIEM superstars but it never hurts to go over the basics again.  You stood up your log management to do this kind of thing quickly and easily.  Better yet, you *have badass dashboards and alerts* for these behaviours!  EVEN BETTER YET, the [firewall team](https://isitthefirewall.net) did their job so your logs are pretty boring. 

Keep backflipping my fellow ninjas. 

---

>**Note 1:** I see **SO MANY** folks configure logging but without injecting the IP address during a NAT transversal.  IF your source logs say 10.blah.bloh.bleh when you know the traffic is coming from the internet, go yell at the router / switch / load balancer / nGinX / [Firewall](https://isitthefirewall.net) person to enable X-Forwarded-For!

---

> **Note 2:** This second point is actually a really useful later stage pipeline rule.  It's one thing to see 100,000 attempts per second of authentications on some system via its API or Web Service that you don't control.  It's much scarier when they suddenly stop.  IF there is an authentication at that point - guess what you just found.  **Stolen Credentials**.  Or a 1 in a zillion coincidence.  I tend to add these fields at the appropriate times during ingestion: 
>- External: Boolean True/False
>- Authenticated:  Boolean Yes/No
>- Browsing: True/False (False is things like POST traffic, 3xx,4xx,5xx etc)
