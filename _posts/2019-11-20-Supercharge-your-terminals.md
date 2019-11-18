---
title: Ninja Console!
date: 2019-11-16 21:05
categories:
  - blog
tags:
  - Windows
  - Console
  - Powershell
  - Random Ninjitsu
published: true
---

# Juice your Shell!

###### Be the hero your systems need you to be
Straight to [the guide](#the-actual-guide)

![](/assets/images/2019-11-17-14-05-46.png)

The *nix crew has had the fun for WAY too long.  I use Yakuake (Non-gnome) or Guake (Gnome) for a single reason:  Quake-style slider.  Not only because its unbelievably handy - but it fondly reminds me of my youth where I could smash any internet stranger in a Rail Arena or Rocket in Quake 2.  Oh... youth is wasted on the young. 

Yes... I even map it to a variation of <kbd>`</kbd>.  

Not that I have ADD, but I'm pretty sure I was using this site 20 years ago to find [Quake Servers](http://q2servers.com/)



### The wish list: 

* Quake-style drop-down
* Transparent **
* **Easily** connect to some of my admin shells (*PowerCLI*, *Exchange Management Shell*, *Skype for Business*, and *SCCM*)
* A useful launch / MOTD
* A picture of a Ninja somewhere
* Colourful *
* **Easily** switch between 
**elevated** and daily driver console - because neither **you** nor I are silly enough to use our workstation as admin.  Don't be daft. 

#### Notes
> *Colour is spelled with a 'u' - Fight me.
> 
> **So I can the `sfc /scannow` advice from technet while I'm in the shell.  Transparent shells make following guides / docs easy.  Just learn to love it.

## IT CAN BE DONE!

### Windows Edition

Meet the stars of performance 
* [ConEmu](https://conemu.github.io/)
* PowerShell [Get-MOTD](https://github.com/mmillar-bolis/ps-motd)
* Some [Profile-Fu](http://q2servers.com/)
* The Remote Shells
* The appreciation of fruit on Pizza.

### MacOS Edition

* Using iTerm2 instead of Conemu
* Tweaking the app file to stop being a jerk and just act like the console it is

```
vi /Applications/iTerm\ 2.app/Contents/Info.plist
```

Find the bottom two `</dict></plist>` keys and add: 

```
<key>LSUIElement</key>
<true/>
```

right above them.  If `vi` isn't your jam, its <kbd>: <kbd>w</kbd> <kbd>q</kbd></kbd> to save and exit from that madness
* It's an actual *nix Shell and I don't have Office 365 - so I'm driving blind as far as getting a Mac -> Exchange Shell with a hotkey

### The actual guide

1. Install [ConEmu](https://conemu.github.io/)
2. Use My ultra ninja [conemu.xml](https://github.com/BlueTeamNinja/Tools/blob/master/General%20Tools/Configs/conemu.xml) profile
3. Download [Get-MOTD](https://github.com/mmillar-bolis/ps-motd) 
4. Make your profile (if you haven't already) 
`New-Item $Profile -type File -force`
5. Tweak your profile
```
get-content ~\Downloads\ps-motd.ps1 | out-file $PSProfile -Append
Write-Ouput 'Get-MOTD' | Out-file $PSProfile -append
```

This is just the cheezy posh version of `cat somefile >> saved_contents`


#### Add this to your profile as well
Replace `$exch = "EXSERV01"` with your own data
```
notepad $profile
```

CopyPasta the following into notepad at the end

```
function connectExchange () {

}
$exch= "EXSERV01"
$Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri http://${exch}/PowerShell/ -Authentication Kerberos
Import-PSSession $Session
}

New-Alias -Name ConnectTo-Exchange -value connectExchange
```

**This gives you the superpower of typing** `ConnectTo-Exchange` at any point and administering exchange.

6. Walk around like a boss.  

>Don't use a clipboard, use post-it notes and a sharpie - it makes everyone far more nervous for some reason. If you are in an office/cubicle farm setting - friendly protip:  Construction helmets and safety vests when completely unexpected give you +30 saving throw vs nosy managers.

Keep your eyes peeled.  I might roll this all out into a much nicer, smoother, easier to use package. 
Or I might not.  Either way... Yell at me on [Twitter](https://twitter.com/BigAbe20)
