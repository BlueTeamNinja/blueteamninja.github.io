---
title: 'Badass Intelligence Part 2: The Listener'
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

## Oh Me, Oh My, a Powershell API !?

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

That has the bits and pieces from the upper example and everything else I've commented on.  Read the comments to get caught up with everything that has gone wrong in your life up until now.  Smile and carry on.  You got this.

The basics of what I'm doing to the same snippet:

1. Making the variables pretty
2. Recursively searching for the Manager details as well
3. Storing them in PSObjects and converting back to JSON to return out to the API
4. One more *SUPER* ninja trick:  I'm adjusting my input for re-usability: 
  * username
  * CN=username,DC=blueteam,DC=ninja
  * SID, etc

```powershell
##  I like to keep my scripts and their modules together.
##  This makes them a lot more portable and easier to automate and commit to code repos etc

### CHANGE ME FROM RELATIVE TO SCRIPTROOT WHEN FINISHED ### 
Import-Module .\Polaris\Polaris.psd1

New-PolarisGetRoute -Path /lookup -Scriptblock {

  if ($Request.Query['user']) {

    #I have a bad habit of using single letters for queries
    #and I WONT CHANGE  Not for ANY PONY IN EQUESTRIA

    $q = $Request.Query['user']
        if($q -match '^CN=')   {
        $r = ([adsi]("LDAP://$q")).Properties
        } elseif ($q -match '([a-zA-Z\-]+\s?\b){2,}'){
        $r = ([adsisearcher]("CN=$q")).FindOne().Properties
        }else {
        $r = ([adsisearcher]("samAccountName=$q")).FindOne().Properties
        }

        $title = $r.title
        $email = $r.mail
        $boss = $r.manager
        $name = $r.displayname
        $branch = $r.description

    ## This is the lookup syntax when you have an exact string that you want.  Also seen above in the IF statements
    $boss = ([adsi]("LDAP://$boss")).Properties
        $bossemail = $boss.mail
        $bossTitle = $boss.title
        $bossname = $boss.name

  #Tidy the results into some objects because 
  #WE ARE NOT SAVAGES
    $qmanager = @{

      ## Take special note of the variable inside the double-quotes
      ## This is a cool hack to convert a single object entity to a string otherwise it would
      ## have { } around it and it would be ugly and I prefer my people to be ugly and my data to be beautiful
        "title" = "$bosstitle"
        "name" = "$bossname"
        "email" = "$bossemail"
    }

    $qresponse = @{
        "title" = "$title"
        "name" = "$name"
        "email" = "$email"
        ## You may notice that qresponse is converted to JSON but not qmanager?
        ## That's because its nested within.  Works out nicely - order of this operation matters somewhat.  
        "manager" = $qmanager
        "branch" = "$branch"
    } | ConvertTo-Json

    $Response.Send($qresponse)
  } else {
    $response.send("Try again, friends.")
  }
  }
  }

Start-Polaris -Port 8080 -hostname $env:COMPUTERNAME
```

### AWESOME.  YOU DID IT

#### You copy/pasted my stuff to look like a hero!

It's cool.  I don't mind. This is only the first half.

Also, at this point you may have run into an error or two if you are playing along.  
You'll need these:  

* `Remove-PolarRoute` and
* `Stop-Polaris`

as you follow along.

Forget error-checking and messing around though, go play with your new toy!

put `http://YourPolarisServer:8080/lookup?user=CHIEF` into a web browser!  Obviously, your username isn't Chief.  Well, except for my friend Cristal Hief but she's not you.


## We've done a userlookup - Now a GPO lookup

Exact same concept.  You even did the snippet in part 1.  This time we will chain some Polaris Routes together with our intelligence, add a dash of error-checking and Voila!: 

```powershell
Import-Module "${PSSCriptRoot}\Polaris\Polaris.psd1"

New-PolarisGetRoute -Path /userlookup -Scriptblock {

if ($Request.Query['user']) {
    $q = $Request.Query['user']

        if($q -match '^CN=')   {
            $r = ([adsi]("LDAP://$q")).Properties
        } elseif ($q -match '([a-zA-Z\-]+\s?\b){2,}'){
        $r = ([adsisearcher]("CN=$q")).FindOne().Properties
        }else {
        $r = ([adsisearcher]("samAccountName=$q")).FindOne().Properties
        }

        $title = $r.title
        $email = $r.mail
        $boss = $r.manager
        $name = $r.displayname
        $branch = $r.description

    $boss = ([adsi]("LDAP://$boss")).Properties
        $bossemail = $boss.mail
        $bossTitle = $boss.title
        $bossname = $boss.name

    $qmanager = @{
        "title" = "$bosstitle"
        "name" = "$bossname"
        "email" = "$bossemail"
    }

    $qresponse = @{
        "title" = "$title"
        "name" = "$name"
        "email" = "$email"
        "manager" = $qmanager
        "branch" = "$branch"
    } | ConvertTo-Json

    $Response.Send($qresponse)
} else {
    $response.send("Try again, friends.")
}
}

New-PolarisGetRoute -Path /gpolookup -Scriptblock {
  if ($Request.Query['guid']) {

    $q = $Request.Query['guid']
    $objStr = "(&(objectCategory=groupPolicyContainer)(name={$q}))"
    $objName = ([adsisearcher]$objStr).findone().properties
    if ($objName) {
        $GPDisplayName = $objName.displayname.trim('{}')
        $GPCreated = $objName.whencreated
        $GPLastModified = $objName.whenchanged
        $GPResponse = @{
            "DisplayName" = "$GPDisplayName"
            "Created" = "$GPCreated"
            "LastModified" = "$GPLastModified"
        } | ConvertTo-Json

    $response.send($GPResponse)
    }else{
        $GPError = @{
            "Error" = "Not Found"
        } | ConvertTo-Json
        $response.send($GPError)
    }
} else{
    $GPInputError = @{
        "Error" = "No valid input"
    } | ConvertTo-Json
    $response.send($GPInputError)
}
}

Start-Polaris -Port 8080 -hostname $env:COMPUTERNAME
```

Have a play with it in a web browser:
http://YourServerName:8080/

## Cool.  

We'll finish up our API tool with the GPO stuff and get it into the Elastic Stack in [Part 3]({% post_url 2020-05-03-Badass-Intelligence-Part-3 %})

