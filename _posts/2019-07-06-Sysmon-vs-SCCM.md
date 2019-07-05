---
title: Sysmon 10 vs SCCM 1902
date: 2019-07-04 19:30
categories:
  - blog
tags:
  - SCCM
  - Windows
  - Sysmon
  - Troubleshooting
published: true
---
# Installing Sysmon via SCCM

### The tale of Sysmon and SCCM:  A Tragedy

#### By Abe

If you are a boring person and you're just here for the solution: 

Create an SCCM Application that: 
1. Copies Sysmon(64) to a Directory outside of ```%SystemRoot%``` aka **NOT** *'C:\Windows\'* 
2. Sets the **TMP** Environment Variable to the directory above.  Note the lack of an 'E'
3. Install sysmon e.g. ```sysmon.exe -i -accepteula -c "C:\Temp\config.xml"```
4. [**Optional**]  Tattoos the registry with a version number for easy detection and upgrades
5. [**Detection**]

    a. Tattooed Registry
    b. 

Are you even more boring?  
This road leads to candy (and working installation code) ### GITHUB PSADT LINK ###

I had the most lovely oppurtunity of deciding whether or not to deploy Sysmon to a larger scale of PCs.  I turned to my trusty copy of SCCM and decided to push out Sysmon.  It repeatedly failed.  So I built a simple PowerShell script.  More fail. 

Then I got down and dirty and pushed it out via the trusty PSADT - which isn't my usual flow, but I figured it wouldn't hurt.  **MORE FAIL**.  At this point, the headknocking begins. 

Oddly, running the same identical PowerShell scripts or command line installations worked perfectly in the console.  
Even more bizarre, they *worked perfectly running as **SYSTEM!!!*** 

Here is another massive oddity.  My actual event collector was getting logs from about 15-20% of the clients in the deployment collection.  

### Troubleshooting the madness

