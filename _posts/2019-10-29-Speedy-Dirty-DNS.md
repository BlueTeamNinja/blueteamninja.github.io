---
title: Speedy, Dirty DNS & Port scan
date: 2019-10-29 21:30
categories:
  - blog
tags:
  - Windows
  - Powershell
  - DNS
published: true
---

# I made a thing
## Exactly what the title says.

It's a native powershell/.net zone transfer... without the zone transfer.  It's on [GitHub](https://github.com/BlueTeamNinja/Tools/blob/master/Network%20Tools/Get-DNSHostRecords.ps1)

I feel like this should turn into `IMAT:`

Either way - it gets your workstation DNS, goes to that server and dumps out all the A, AAAA and CNAME records (hence the name Get-DNS*HOST*Records) and dumps them out with domains as an object.

A few thousand devices across 8 or 9 domains is about 3 seconds so far. 

May or may not require DNS RSAT - I have it.

```
$q = .\Get-DNSHostRecords.ps1
$q | measure
$q | group Domain
$q | group 'Record Type
```

Do whatever you want. 

Here's the hardcore version.  Paste this function into a powershell window (original isn't mine, but I've modified to make it a bit more modular for my uses): 
```
function Test-Port ($hostname,$port,$timeout=40) {
    $requestCallback = $state = $null
    $client = New-Object System.Net.Sockets.TcpClient
    $client.BeginConnect($hostname,$port,$requestCallback,$state) | out-null
    Start-Sleep -milli $timeOut
    if ($client.Connected) { $open = $true } else { $open = $false }
    $client.Close()
    $results = [pscustomobject]@{hostname=$hostname;port=$port;open=$open}
    return $results
  }
```

Now if you put the two together you can quickly find all the web consoles from your DNS.  Before you go all 'USE NMAP' on me - I don't know how to pull multiple DNS zones into it without exporting to a file.  Use the powershell for that. 

Either way... now you have something cool like this: 
```
$q = .\Get-DNSHostRecords.ps1
$webservers = $q | %{Test-port $_.URI 443} | ? {$_.open -eq $true}
$webservers
```

There - find your open 443 sockets across a whole whack of stuff - AND ITS A POWERSHELL OBJECT. I'd recommend filtering out your workstations but whatever makes your bum hum, kids.  

Enjoy - or don't.  *Hat Tips*
