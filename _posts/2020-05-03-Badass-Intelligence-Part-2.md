---
title: 'Add badass intelligence to logs - Part 2'
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

### This Part 2 of the series

* [Part 1:]({% post_url 2020-05-03-Badass-Intelligence-Part-1 %}) Intro
* Part 2: Creating the API *(You are Here)*
* Part 3: Querying with Logstash (HTTP Filter)
* Part 4: 

## Oh Me, Oh My, a Powershell A P I!?

You can do it in .NET or basically just let some of the bored folks (literally) at Microsoft do it for you.  Some of those great kids have given us **Polaris**. 

### Get your tools

You need:

* Polaris:  Don't bother installing from the command-line - just go get the latest code.  The Powershell Gallery version was ancient last I looked.
* Enthusiasm

Install Polaris:
**Here you go** - you lazy sods:  

```powershell
iwr https://github.com/PowerShell/Polaris/archive/master.zip -OutFile Polaris.zip;Expand-Archive .\Polaris.zip;Remove-Item .\Polaris.zip
```  

This will dump the module whereever you are sitting in your Ninja Console.

The very basics of Polaris: 

* You create a new 'Polaris Route' (path and query, etc)
* You create a 'Polaris' (hostname and IP, etc)

 *Polaris Route* directs incoming HTTP traffic where you want it.  In most cases, that means you want to respond to an HTTP GET from a client.  That is what any webpage you are looking at is doing - you, proud of your opposable thumbs, browse to a URL and your browser does HTTP GET.  The server responds with this beautiful blog in the form of HTML.  The browser enjoys this and responds accordingly.  Not long after they are laughing, dreaming of the future, making mix-tapes and trusting each other with overdue Blockbuster rentals.  It's beautiful. 

*Polaris* Basics of the server config.

If I want to use the URL `http://NinjaStar.blueteam.ninja:8080/lookup/user="CHIEF"` and have it respond with the details of CHIEF: 

* Polaris:  
  * Hostname: ninjastar.blueteam.ninja
  * port:8080
* PolarisRoute: 
  * Method `GET`
  * Path `/lookup`

Let's see a simple example of how to set that up: 

```powershell
##  I like to keep my scripts and their modules together.
##  This makes them a lot more portable and easier to automate and commit to code repos etc

### CHANGE ME FROM RELATIVE TO SCRIPTROOT WHEN FINISHED ### 
Import-Module .\Polaris\Polaris.psd1

New-PolarisGetRoute -Path /lookup -Scriptblock {

# IOU - Working code goes here

}
Start-Polaris -Port 8080 -hostname $env:COMPUTERNAME
```

Now we want some API code itself that can look for data with the name user which will have the value CHIEF

Polaris does most of the heavy lifting, so we just have a value called `$Request`
Remember part 1?  Let's build on it some more

Check this out: 

```powershell
$adSearch = $Request.Query['user']
[adsisearcher]("samAccountName=$adSearch")
```

That has the bits and pieces from the upper example:

```powershell
##  I like to keep my scripts and their modules together.
##  This makes them a lot more portable and easier to automate and commit to code repos etc

### CHANGE ME FROM RELATIVE TO SCRIPTROOT WHEN FINISHED ### 
Import-Module .\Polaris\Polaris.psd1

New-PolarisGetRoute -Path /lookup -Scriptblock {
$adSearch = $Request.Query['user']
[adsisearcher]("samAccountName=$adSearch")
}
Start-Polaris -Port 8080 -hostname $env:COMPUTERNAME
```

At this point you may have run into an error or two if you are playing along.  
You'll need these:  

* `Remove-PolarRoute` and 
* `Stop-Polaris` 

as you follow along.

