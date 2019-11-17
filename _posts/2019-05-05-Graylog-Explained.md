---
title: Graylog Explained
date: 2019-05-06 21:30
categories:
  - blog
tags:
  - graylog
  - Windows
  - Logging
published: true
---

# Let's talk logs, shall we?

## Amazing Open-Source Log Management for your everyday Windows Ninja 

GrayLog is a Log Management platform.  It's primary purpose is to injest large amounts of log data, then process and store it according to your design.  The Web interface built on top of it allows you to search through millions of log records and quickly retrieve your information. You can also sort it into visually appealling dashboards with charts, tables, trend indicators and more for great insight and analysis.  

I am going to break down all of the concepts of GrayLog that I struggled to wrap my head around.  After this guide, you can start making Operational, Security, and Business Intelligence displays with live information. 

If you have every tried to put all of your logs in one place, or even thought about it - that has most likely lead to some google results looking into tools like Splunk, LogRhythm etc.  These tools are awesome.  So are their price tags.  

Here we are.  

GrayLog.

I first stumbled on GrayLog about a year ago.  I'm not going to lie, I was overwhelmed.  Like, well and truly overwhelmed.  I'm a competent Linux User - but a far cry from a seasoned Administrator.  I can easily drive 1,000s of Windows Workstations and hundreds of Windows Servers (I hope; it's what I do during the day) - but GrayLog just perplexed me.  Installation and basic configuration was easy enough, but *how do I **design** this thing?*  *How do I view Windows Logs in this beast?*

Now that I have wrestled most of the concepts down I wanted to share what I have learned.

### Bias:  Latest version

I'm starting my GrayLog adventure on GrayLog 3.  I grabbed the beta as soon as it was out just so I started off on the latest platforms and concepts.  From what I can gather on the Community Forums a lot has changed from 2.X to 3, and truthfully, I'm not too interested in the old way - however if you're trying to match up some of the concepts to your current or POC environment it is probably a version difference. 

### Key Concepts

This is where I struggled the most in figuring out how to really unleash the power of this system.  While I could stand up a GrayLog instance and start collecting syslog events, I wanted this to be a system that was still useable in a year.  I wanted to be able to update an internal KB and have someone take over when I get hit by a ~~bus~~ *better offer*.  Most importantly, how do I keep *some* logs or retain some logs **longer** than others?  

The most fundamental pieces to me are, in no order:

* [Inputs](#inputs)
* [Log Collectors & Beats](#log-collectors-and-beats)
* [Indices (Index Sets)](#indices--index-sets)
* [Streams](#streams)
* [Pipelines](#pipelines)
* [Pipeline Processing Rules](#pipeline-processing-rules)
* [Extractors](#extractors) (Bonus: [Grok? Huh?](#what-does-grok-mean))
* [Message Processing Order](#message-processing-order)
* [SideCar](#sidecar)

## Inputs

This is where you actually tell GrayLog to listen for incoming logs.  If you have played with GrayLog a little bit, by now you have likely created a couple of inputs.  There are a ton of pre-built inputs and a capability to add more via plugins.  I have yet to need an outside plugin for any purpose so far.  The built-in plugins basically determine a port, a protocol, and a default parsing mechanism.  I tend to group my inputs by delivering systems but this is entirely up to you.  I spread all the inputs around as well and others use as few as possible.  Further considerations are upstream firewalls.  The less ports you use, the less change management to request some holes poked for logs. 

**Rookie Mistakes**:  One of the first thing that folks get hung up on is that they created an input, they see network traffic, but no logs when they click into it. 

**PRO TIP:** This has two really common causes for beginners.

**Timezones** or **Parsers**.  Either your logs are sending in a timestamp WITHOUT A TIMEZONE and GrayLog is adjusting it to your local timezone.  Search your logs for a day in the future or the past to see if your live logs are there.  For the latter, delete your input and try it as 'Raw' input to rule out any network connectivity. 

## Log Collectors and Beats

Many of the inputs on GrayLog are meant for devices/applications to send various forms of Syslog or GELF messages into the various places with GrayLog.  However, there are just as many scenarios where you want to collect an actual 'connections.log' file from an application folder, or within ```/var/log```

This is where "Beats" come into play.  These are applications/binaries built and maintained by ElasticSearch.  They can be easily (and often required) created by hand or by automation using the sidecar collector service.  

I have done everything so far using only 3 different beats: 

  * FileBeat - Cross-platform binary that is configured to send entries created in a log file to the GrayLog service.  
  * WinLogBeat - Windows tool used to send in logs from Windows Event Viewer.  Examples are Event ID 4624 for "User Logged in" or workstation 'Error' messages.  Can you imagine the surprise of your users when you call them **BEFORE** they ever had or reported an issue?
  * PacketBeat - Sent packet trace events with a big collection of prebuilt network signatures.  Examples are DNS (port 53) or DHCP. 

## Indices  (Index Sets)

I think this was the hardest for me to grasp.  As a Windows admin, concepts like ElasticSearch and MongoDB are just words.  

Basically, an Index Set is your retention policy.  In fact, to me, the wording is crazy (until I learned much more about the whole ELK stack, now it makes a lot more sense).  The **Index Period** is how often you want the logs to rotate.  You can choose to rotate logs by size or period.  In other words, do you want to keep 10GB of Webfilter Logs, or 180 days of Webfilter logs.  You can apply any **STREAM** to an Index Set.  Design your index sets as "Categories" of log retentions.

#### Architectural Example
In my environment I set up the following Index Sets: 

* Compliance:  These are logs I want to keep for a long time.  Mostly they are the audit logs where permissions, or authentications were actually changed.  E.g.  Active Directory Group membership Changes, Multi-Factor enrollment etc.
  * Samples:  User Create/Delete, OU Changes, Group Membership Changes,
  * Retention:  60 Months
  * Naming:  **CMP-**

* Security:  These are my Incident Response logs.  I always try to refer back to an Incident Response plan.  When everything goes to hell in a handbasket and you have 4000 workstations asking for bitcoins - what do you want to look at?  Hopefully, your offsite GrayLog instance with good Security logs.
  * Samples:  Modified Domain Admins, Add/Remove/Change Public Firewall Rules, RDP Sessions, Login Success/Failures
  * Retention: 18 Months
  * Naming: **SEC-**

* Performance:  These are the logs I intend to use for handing off to specific System/Server admins (or just put your other hat on in a smaller shop).  
  * Samples:  IIS Logs, Load Balancer Logs, Wifi AP Logs
  * Retention:  5 Weeks (Long enough to make a 'Last Month' visual)
  * Naming: **PER-**

* Metrics:  These are the really loud, noisy things that are mostly cool to look at.  The kinds of things that can show a problem, but typically when the quantities change HUGE. 
  * Samples:  DNS Queries, AD Authentications (Non-Interactive)
  * Retention: 10 days
  * Naming: **MET-**

* Test/Dev:  When you start building more and more pipeline rules, you will inevitably accidentally slam millions of logs into your 5-year retention rule.  Then, when you try to remove just those new, unfiltered entries - you'll then learn you can't delete individual logs - only individual indicies.  I'll save you some Derp moments.  We both know you'll make this mistake, though.  At least I told you so. 
  * Samples:  Anything half-baked. 
  * Retention:  3 days (So you can resume Fridays work on a Monday.  Trust me.)
  * Naming: **TEST-**  <-- Yes - its 4 characters instead of three so that its EXTREMELY obviously in a giant list of index sets which ones are still being made.

## Streams

Once you have a handle on Index Sets - Streams become much easier to sort out.  You assign a stream to a single Index Set.  By default, all messages go into the cleverly titled "All Messages" stream.  Since you assign a single Index Set to a Stream, they are basically an extension of Retention policies, only more granular.  Using the examples above, you might keep certain Active Directory Changes for 5 years, and some events for only 18 months or 1 month.  

Which Stream a log message goes into is based on a set of rules, but each rule is on a single field.  Some of the collectors sending in logs allow tags, and I use these extensively from Windows servers and applications to route logs into Streams.  You can allow a single message into multiple streams as well.

Given that I operate in a world with a small number of hands in the logging pies, I keep my Streams closely resembling the input logs.  

#### Example
_When I use the term **Audit**, I am typically referring to a compliance (PCI, MFIPPA, PIPEDA - and their American equivalents)_

The following examples are really high-level as we haven't discussed Rule Matches to any level, but just realize I am giving you a name of a Stream followed by a rough idea of how I would set it up in a standard environment. 

  * Audit:  Active Directory
    * Index Set:  **Compliance**
    * Rule Match:  *Event log tag "AD_AUDIT"*
  * Event:  Active Directory
    * Index Set:  **Security**
    * Rule Match: *Event log tag "AD_EVENT"*
  * Corporate Web Traffic - Standard
    * Index Set:  **Performance**
    * Rule Match: (*From: Firewall* && *Action: Not Blocked*) 
  * Spam Emails
    * Index Set: **Metrics**
    * Rule Match: (*From: MailFilter* && *Severity: Low*)

## Pipelines

Pipelines contain *Stages* of **Pipeline Processing Rules**.  You can apply a near-infinite amount of logic and processing to your incoming logs here.  While inputs and Index Sets create magic - Pipelines are the gritty work that make your GrayLog environment incredibly **Valuable**.  

Each Stage can take actions and either proceed to further stages or not.  Once you have an idea on how Pipeline Processing Rules work, designing Stages will be up to your own imagination. 

My real basic flow for a pipeline is:  
1. Discard
2. Tidy ugly logs
3. Tag logs
4. *Any further special processing *

#### Example

*Stages default at 0, but you can use whatever scheme you want.  Sometimes a 100,200,300 rule makes sense in case you want to drop something in between without further editing*. 

  * Firewall Pipeline
    *  Stage 0:  Discard Unwanted Messages (Noise)
    *  Stage 1:  Adjust Timezones
    *  Stage 1:  Normalize fields (turn all client, source, clt, src, src_ip fields into *Client* ) 
    *  Stage 2:  Tag IPS Traffic
    *  Stage 2:  Tag Webfilter Traffic
    *  Stage 2:  Tag Administrative Change/Events
    *  Stage 3:  Identify Internal only traffic (10.x.x.x to 10.y.y.y)

## Pipeline Processing Rules

These are the scripts / configurations the do the actual processing of the fields.  You can pretty much do anything you want to a message at this stage of the game.  

>  Pro Tip:  When you first start with GrayLog, you are likely going to build a bunch of 'Extractors' to get the Data you like from a message.  As I learned more, I found I used extractors a LOT less as I found ways to massage most of my messages at this stage.

The most common uses: 
  * Drop unwanted messages
  * Combine or Append Fields(i.e. FQDN = Hostname + "." + Domain)
  * Remove/Rename Fields - Very common activity using Collectors or standardized Sidecar configurations. 

  #### Examples

This rule checks if a message has a field called "Debug" with a value of 5.

  ```
  Rule "Remove 
when
  has_field("Debug") AND
  to_string($message.Debug) == 5
then
  drop_message();
end
```

Another example is modifying message fields.  If you use a collector such as FileBeat, the default configuration will include a bunch of metadata such as the filebeat version, etc.  Also, you may want the ```filebeat_client_IP``` translated into something like ```source``` to standardize your searchs. 

This rule checks that both a ```filebeat_client_IP``` exists and that the value is within a specific IP range (e.g. workstations on the WiFi subnet).  Then it sets two fields accordingly and removes the unwanted remaining fields. 

```
rule "remove and clean filebeat log"
when
    has_field("filebeat_client_IP") and
    cidr_match("10.0.100.0/24", to_ip($message.filebeat_client_ip))

then
    set_field("wifi", "Yes");
    remove_field("filebeat_client_IP");
    remove_field("filebeat_beats_version");
end
```

>  PROTIP:  When I build a pipeline rule, I go look at the log and select the fields from the view on the left.  Then I paste into Notepad++ or any advanced text editor.  Then I use the find/replace tool and regex for something like "

```Find all ^; 
Replace with remove_field\(\";
Find all $;
Replace with \)\;
```

## Extractors

This is a great way to process a single field.  Extractors are organized per-input so you can carry out actions on a certain field or parse out larger fields into smaller ones.  A simple example is an IIS log.  These come with all the W3C standard fields, so you can create and extractor on that one input, and use Grok / REgex to split it up into all the necessary smaller fields. 

##### What does Grok mean

This is also a great place to touch on some of these terms that aren't very common in the Windows world. 

  * Regular Expressions (Often called Regex)
     *  Regex is a language used to match queries within a larger string.  It carries a fairly large learning curve, but once mastered you can do some insane things on it.  A really simple example already used in this document includes ```^``` which indicates the start of a line and ```$``` which indicates the end of a line.  Other often used regex expressions are things like ```(a-z){4}``` which means "Match and character between a and z (notice the lowercase), and do it 4 times.  So ```food``` is a match but ```Food``` or ```dog9``` would not match.
  * Grok
     * I always describe Grok to others as "Bookmarks for Regex".  It also carries an easy mechanism for discarding, or renaming fields. The basic scheme is %{**GROK_EXPRESSION**:*LABEL*}
     
#### Example: 
    
A message like:

```10.10.20.50 Webfilter Block https://www.reddit.com Non-Productive Resource``` 

could be parsed as

```%{IPV4:Source_IP} %{DATA:Category} %{DATA:Action} %{URI:Destionation_URL} %{GREEDYDATA:Reason}```

This would result in a message looking like this:

| Field | Data | 
|---|---|
| Source_IP | 10.10.20.50 | 
| Category | Webfilter | 
| Action | Block | 
| Destination_URL | https://www.reddit.com | 
| Reason | Non-Productive Resource | 

Grok makes parsing out messages very easy and has a way lower learning curve than Regex itself.  As you use it more and more, and discover limitations - you'll likely end up learning Regex by accident.  :)

## Message Processing Order

With a basic understanding of Extractors and Pipeline Processing Rules, you may have already stumbled on a dilemma.  What happens if I have a discrepancy between a pipeline processing rule and an extractor?  Both of them can extract or remove data from a field.  You can make a decision in the global configuration of GrayLog that determines whether messages are handled at the Pipeline level or the Extractor level first.  This is a brilliant option and really helps plans a deployment and architecture model.  Many administrators advise one method or the other but that depends on how you have chosen to design your system overall. 
    
## Sidecar

If you want to pull logs from more than 1 of any kind of server Sidecar is a total weapon.  Sidecar is easiest described as a 'Configuration Manager for Log Collectors'.  You install sidecar, ensure it has authentication and network access to the GrayLog api (i.e. port 9000 and an API key) and this allows you to distribute multiple configurations for various types of Log collectors. 

I typically do this for any server that has more than 1 of the same type.  Most environments have multiple IIS servers, or multiple authentication/Radius servers.  If you want to draw Windows Events, sniffed packets on specific ports (DNS, SIP, DHCP etc) then you are going to get involved in those specific collectors.  Sidecar helps manage those collectors in the sense that you can template the configuration of the various types of collectors, and then create more granular individual configurations and even stop or start the various processes. 

The limitation, is installing collectors that don't ship with sidecar itself.  The sidecar in Graylog3 contains FileBeat (for flat log files) and WinLogBeat (for Windows Events) with the windows package.  This is enough to get started for sure and become comfortable.  As you progress, you will build and deploy installations for PacketBeat and others - and continue to manage their configurations through the sidecar page.

Using the Beats family of Collectors allow you to easily create new inputs as you further your variety of log injestion quickly and easily.  Beats is a default input type and requires very little configuration other than ports and SSL information. 

#### Examples

* Sidecar:  "Domain Changes"
    * Log Collector:  WinLogBeat
    * Configuration:  Active Directory Events
    * Purpose:  *Collect specific Event ID like 4720 - User Created, Monitor for anomalous activity.

* Sidecar:  "DNS Events" 
  * Log Collector: Packetbeat
  * Configuration: DNS / Port 53
  * Purpose:  Collect and normalize DNS events, Monitor for anomalous values.

* Sidecar:  "IIS Events" 
  * Log Collector: Filebeat
  * Configuration: %systemroom%/inetpub/LogFiles/*/*.log
  * Purpose:  Monitor for Response Code (5xx, 4xx) and anomalous volumes.

## Closing out

This wraps up my best "Brief Overview" of the crazy extent of GrayLog.  Drop a disqus line below and let me know what you think. You can also find me asking slightly less 'derpy' questions than a few months ago on the GrayLog Community forums.  I even chime in once in awhile with my $0.02. 
