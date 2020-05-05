---
title: 'Add badass intelligence to logs using an API... made in PowerShell?'
date: 2020-05-02 03:15
categories:
  - blog
tags:
  - elastic
  - powershell
  - Polaris
  - Windows Events
  - Threat Hunting
published: false
---

### This Part 1 of the series

* Part 1: Intro (You are Here)
* [Part 2:]({% post_url 2020-05-03-Badass-Intelligence-Part-2 %}) Creating the API
* Part 3: Querying with Logstash (HTTP Filter)
* Part 4: 

### Shortcuts

* [The Tutorial](#the-actual-tutorial)
* [The snippet you need](#the-actual-snippet)
* [Make a Ninja Console (Optional)]({% post_url 2019-11-20-Supercharge-your-terminals %})

## Adding badass intelligence: Part 1 - Introduction

First, I'm going to explain the problem I've come across.  Basically - you can grab a lot of good info about windows events out of the box, but there are also a lot of gaps.  These gaps are even more noticeable when you really dive under the hood.  One of my most memorable cases of this was trying to simply track changes made to GPOs.  Yes, I am well aware there are third-party products that do this but that isn't always an option.  Whether its resources, time, or basically alert fatigue is so severe that you only want to know about the stuff you can actually get a grip on.  

I'm going to use this exact example for this guide.  In this case, I am talking about Windows Event ID 5136 (Changes made to a Directory Object).  This event ID covers a lot of interesting things and I will show you the steps I took to narrow this down to just my goal:  Changes made to Group Policy Objects (GPOs).  

Let's look at that exact log: 

> This is a beautiful screenshot of a very important log, particularly for sysadmins but the cyber folks as well.  Changes made to Active Directory Objects.  Notice what is missing - the *NAME* of the object that was changed!  Those of you who are more observant than me will notice that Name is blurred out - but that is the name of my internal domain NOT the name of the object that changed. 

|![Event ID 5136](/assets/images/5136.png)

What is the security relevence?  It's always nice when your default domain policy gets modified to include a new Local Administrator, right?  That's a cute thing to see. 

However, you pull that log up in your *Thrunting* fashion in Kibana and BAM!  There is all your goodness, but without a freaking clue about what is going on.  I mean, it is really nice that there was a modification to `{6AC1786C-016F-11D2-945F-00C04fB984F9}`.  That is great news to a robot somewhere.  What I told you there was a modification to **Default Domain Controllers Policy**?  That actually means something to us meat popsicles.

### Dive in those dumpsters - it's not there

I tried and tried and tried.  So have smarter people than me.  That information isn't in the log.  We need to correlate.  That is what a SIEM does - and in this case, you're reading my article because *you* are the SIEM.

### Query for it

We have a few options, but to do this at scale, I wanted to be able to process upwards of a few thousand a minute.  Not a huge flood, but way more than enough to have to stop and think about the speed of whatever my solution is.

First choice for most:  Python or anything else.   **Cowards**. 
Firtster choice:  Powershell!

### Get your tools

You need:

* Polaris:  Don't bother installing from the command-line - just go get the latest code.  The Powershell Gallery version was ancient last I looked.
* Enthusiasm

Install Polaris:
**Here you go** - you lazy sods:  

```powershell
iwr https://github.com/PowerShell/Polaris/archive/master.zip -OutFile Polaris.zip;Expand-Archive .\Polaris.zip;Remove-Item .\Polaris.zip
```  

This will dump it whereever you are sitting in PoSH.

The next thing I wanted was a way to convert the data of `{6AC1786C-016F-11D2-945F-00C04fB984F9}` and convert that to hairflip beautiful english.  Everything on the information superhighway tells you to use `Get-ADObject` or `Get-GPO` etc.  These are great, but they have dependencies, they involve extra modules and most importantly - they are slow.  Slow like "How high are you? ... Yes." level of slow. 1-2 seconds will suck with traffic going through.  

### The actual tutorial

The first thing we want to do for this shindig is to get some powershell working that can translate our data.  This is a honed-in example, but you can follow the rest of the guide applying the exact same concepts to your logs.  You can build out a pretty sweet API and I'll add some more examples at the end.

#### Converting a GUID to a GPO Name

1. Use ADUC / DSAC - Who knows what their real names are, that is how you launch them from command line.  Either way - we don't want to clicky mcClickFace our way through this.  Don't be boring.
2. `Get-GPO` - It has dependencies and I found it slow.  YMMV.
3. VBScript.  (Get Out meme)
4. My way.  You can agree with me or be wrong.  Your choice. 

Sweet, now that we have settled on #4, let's do this. 

#### .NET Accelerators for the Win

I love this dirty trick.  `[adsisearcher]` is one of my favourite utilities to abuse in a powershell script for speed.  Pretty much all of the accelerators rock.  It's right in the name.

Try this on a domain-joined windows box:

```powershell
[adsisearcher]("samAccountName=$env:USERNAME")
```

You'll see a bunch of nonsense.  Let's build on it.  Wrap what you typed in brackets and add `findone()` to the end:

```powershell
([adsisearcher]("sAMAccountName=$env:USERNAME")).findone()
```

Nothing changes - except its a good practice to avoid ambiguity.  Purely good habit.  Let's go *slightly* further: 

```powershell
([adsisearcher]("sAMAccountName=$env:USERNAME")).findone().properties()
```

Hot damn.  We now have a bucket of info about... you.  Good sport old boy. The adsisearch accelerator is very versatile and hate typing again and again, so in the spirit of me being awesome.  I'm going to make a string out of my query and then call adsisearcher with that variable. 

```powershell
$adsiQueryStr = "sAMAccountName=$env:USERNAME"
([adsisearcher]("$adsiQueryStr")).findone().properties()
```

The results are cool and you can start to see where this is going: 

```text
Name                           Value
----                           -----
lastlogoff                     {0}
mstsexpiredate                 {4/14/2015 7:21:42 PM}
department                     {Badassery Wizards}
msexchpreviousrecipienttype... {1}
msexchrecipienttypedetails     {1}
msexchwhenmailboxcreated       {11/3/1984 2:27:16 PM}
primarygroupid                 {513}
msexcharchivequota             {104857600}
title                          {Chief of Amazingery}
mail                           {chief@blueteam.ninja HMU}
whenchanged                    {4/26/2020 3:04:13 PM}
givenname                      {M'Lord}
```

Take note of the curly braces.  That means its a list / array.  In most cases, it is only one object but take note of it.  We'll need to account for that, but not right now, chief.  Slow down a touch.  Let's say we want to access my department.  You can call an individual property - but it's a LOT easier to assign it to a variable then call the properties.  Let's make this entire list into simply `$r`.  Hit up in your [ULTIMATE NINJA CONSOLE]({% post_url 2019-11-20-Supercharge-your-terminals %}) and change it to 

```powershell
$r = ([adsisearcher]("$adsiQueryStr")).findone().properties()
```

Now you can try calling a single value using ```$r.department` and you should see `{Badassery Wizards}` unless you work somewhere boring, then you'll see a real department.

Now that we have gone through this little exercise, I'll save you some research.  Instead of doing an ADSI search for your own logged-in username, we're going to do it for a GUID.  Since we could theoretically get GUIDs mixed up with other things, and more importantly for speed, we'll narrow our search down to just the objects we want using the LDAP filter syntax: `&(objectCategory=groupPolicyContainer)` followed by our query: `(name={$ValueOfGUID})`

```powershell
$q = "6AC1786C-016F-11D2-945F-00C04fB984F9"
$adsiQueryStr = "(&(objectCategory=groupPolicyContainer)(name={$q}))"
$objName = ([adsisearcher]$adsiQueryStr).findone().properties
```

You should be staring at your default domain policy at this point.  If not, I failed you horribly and I will have someone discipline me later.  Meanwhile, go grab a GUID from Group Policy Manager and swap it out for the `$q` variable above and re-run it.  

>Normally, this is when I would simply wrap this up as a function then save it in my profile.  It's pretty useful to do this with pretty much anything you've ever had to google.

However, in this case I don't want a function - I want an API I can query!  I will toss in a bit of error checking and prettiness: 

#### The actual snippet

```powershell
$q = "6AC1786C-016F-11D2-945F-00C04fB984F9"
$adsiQueryStr = "(&(objectCategory=groupPolicyContainer)(name={$q}))"
$objName = ([adsisearcher]$adsiQueryStr).findone().properties
  if ($objName) {
    $GPDisplayName = $objName.displayname.trim('{}')
    $GPCreated = $objName.whencreated
    $GPLastModified = $objName.whenchanged
    $GPResponse = @{
      "DisplayName" = "$GPDisplayName"
      "Created" = "$GPCreated"
      "LastModified" = "$GPLastModified"
    }
```

Save your file as `snippet_GUIDtoName.ps1` and head over to [Part 2]({% post_url 2020-05-03-Badass-Intelligence-Part-2 %})