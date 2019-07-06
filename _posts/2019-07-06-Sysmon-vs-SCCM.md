---
title: Sysmon 10 vs SCCM 1902
date: 2019-07-06 03:18
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
### A.K.A.A troubleshooting breakdown written at 3am. 

#### By Abe

If you are a boring person and you're just here for the solution: https://github.com/BlueTeamNinja/Tools/tree/master/Installers/Sysmon

If you are slightly less boring and just want to know how - here you are: 

Create an SCCM Application that: 
1. Copies Sysmon(64) to a Directory outside of ```%SystemRoot%``` aka **NOT** *'C:\Windows\'* 
2. Sets the **TMP** Environment Variable to the directory above.  Note the lack of an 'E'
3. Install sysmon e.g. ```sysmon.exe -i -accepteula -c "C:\Temp\config.xml"```
4. [**Difficult Mode**]  Tattoos the registry with a version number for easy detection and upgrades
5. [**Hardcore Mode**]

    a. Tattooed Registry

    b. Compare PathName of EXE from Service to known version

    c. Compare running config to assigned config

Are you even more boring?  
This road leads to candy (and working installation code) 

PSADT Edition: https://github.com/BlueTeamNinja/Tools/blob/master/PSADT_Tools/Deploy-Sysmon.ps1
Simple Posh Edition: 

I had the most lovely oppurtunity of deciding whether or not to deploy Sysmon to a larger scale of PCs.  I turned to my trusty copy of SCCM and decided to push out Sysmon.  It repeatedly failed.  So I built a simple PowerShell script.  More fail. 

Then I got down and dirty and pushed it out via the trusty PSADT - which isn't my usual flow, but I figured it wouldn't hurt.  **MORE FAIL**.  At this point, the headknocking begins. 

Oddly, running the same identical PowerShell scripts or command line installations worked perfectly in the console.  
Even more bizarre, they *worked perfectly running as **SYSTEM!!!***

Here is another massive oddity.  My actual event collector was getting logs from about 15-20% of the clients in the deployment collection.  

### Troubleshooting the madness

Grab some popcorn.

I could NOT for the life of me figure out what was going on.  The service was installed in both the working and non-working cases.  I needed to figure out what was different.  After a near-concussion from smashing my forehead against my desk until the good idea fairy came along, I used an old ninja mind trick to get the path of the service.

```PowerShell
// This is how I type my code all pretty for you great folks. 

Get-WmiObject -class win32_service | Where-Object {$_.Name -like "sysmon"} | Select-object -expandProperty PathName
```
#### JUST KIDDING

```PowerShell
// Omit Needless Words in 1liners. -Aristotle

(gwmi win32_service| ? Name -like Sysmon).PathName
```
^^^ That one is way cooler.  They both provided me with ```C:\windows\CCMTEMP\Sysmon.exe``` 

Why SCCM?  WHY!?  

Since I had no idea how to avoid this nonsense, I wanted to know what was different between a SYSTEM context install and my own.  So, I made a powershell script that just dumped environment variables.  After some google-fu, iced coffee, and a bit of rapid knuckle cracking I decided to try looking for that folder in the Environment Variables.  I made a really simple powershell and deployed it to my PC via SCCM: 

```Powershell
Get-Childitem Env: | Where-Object {$_.Value -like "$env:SystemRoot\CCMTEMP"}
```

Just kidding.  You know better, I know better.  Shorthand those 1-liners, kids.  Full verbs are for scripts. 

```Powershell
gci env: | ? Value -like "*CCMTEMP*"
```
# BAM

```
Name                           Value
----                           -----
TMP                            C:\Windows\CCMTEMP
```

**OBVIOUSLY** now at this point we know the SysInternals installer uses the TMP environment path, which SCCM version 1902 screwed around with.  

SOOOO.... to install Sysmon via SCCM you just need to modify the TMP variable.  I used the install location, you do whatever blows your hair back.

```PowerShell
//Set the configPath variable to your copy of SwiftOnSecuritys XML file

$env:TMP = "C:\Temp"
start-process -FilePath "C:\Temp\sysmon.exe" -ArgumentList "-accepteula -i -c $configPath" -wait

```

I'm tapping out.  Enjoy your morning. 