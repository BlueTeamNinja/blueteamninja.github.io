---
title: 'Badass Intelligence Part 3: The Elasticy pieces'
date: 2020-05-02 03:15
categories:
  - blog
tags:
  - elastic
  - powershell
  - Polaris
  - Windows Events
  - Threat Hunting
published: true
---

### This Part 3 of the series

* [Part 1:]({% post_url 2020-05-03-Badass-Intelligence-Part-1 %}) Introduction
* [Part 2:]({% post_url 2020-05-03-Badass-Intelligence-Part-2 %}) Creating the API
* Part 3: Finishing API and getting into Elastic *(You are Here)*


## Logstash Injestion

This is actually the easy part.  Here is an example of a Logstash config that looks for the right Event and adds the information.  I'm not going to go indepth into Logstash configs for now. 

We'll break down the logic after the code:
{% raw %}
```
 if [winlog][event_data][ObjectClass] == "groupPolicyContainer" {
  grok {
    match => [
      "[winlog][event_data][ObjectDN]",
      "CN\=\{%{GREEDYDATA:[policy][dn]}\}"
    ]
  }
  http {
        url => "http://vsscriptWin:8080/gpolookup"
        query =>  { "guid" => "%{[policy][dn]}" }
        verb => GET
    }
  if [body] {
   json{
     source => "body"
     target => "policy"
   }
   mutate { remove_field => "body"}
  }
```
 {% endraw %}
The if statement looks for any winlogbeat with an ObjectClass of a groupPolicyContainer.  
There are several Windows Event IDs that contain GUIDs of GPO names.  This will run the lookup on all of them, map
the corresponding json results to *policy.dn*.DETAILS. 

If you've finished everything up to this point - you have collected your logs, augmented them with a very flexible powershell interface that allows you to POSH your data up in as many ways as you have Neurons.

A filter in your log collector for `Eventid: 5136` and `ObjectClass: groupPolicyContainer` you can now see all your GPO changes WITH A NAME!

![Diagram](/assets/images/GPOFilter.png)

You can see what you need to see!
![Diagram](/assets/images/NewGPO.png)

I hope you enjoyed this guide.  Click around and hit the Socials. 

Cheers!