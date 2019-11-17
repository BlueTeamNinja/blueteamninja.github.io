---
title: Your annoying logs hate this one weird trick
summary: Green Lasers.  Pew pew.
date: 2019-07-09 18:22
categories:
  - blog
tags:
  - Linux
  - Graylog
  - Troubleshooting
  - ProTips
published: true
comments: true
---

# Ninja Protip to test out network traffic

### Also an easy way to analyze raw graylog messages

This is my little ninja trick for getting *raw log data* to paste into the Pipeline Simulator.  We can talk more about that later.  It is also a decent trick to double-check your connection protocol (TCP/UDP), network connectivity, firewall, etc.  If you see the messages coming in hot, but graylog is drawing blanks - your config is the enemy.  If you don't see any messages - then you got 99 problems but your config ain't one.

Pay close attention, kids.  I don't want to lose anyone in the middle of this rodeo.

Step 1:  Stop the input (and take note of the port and protocol)
> Example: Syslog UDP Input on Port 4550

Step 2:  SSH into one of your Graylog input servers

Step 3:  ```nc -lvu 4550 ```

Step 4:  Wait.  Profit.

IF you need a TCP listener, drop the _u_ from the flags above.  If you need TLS you need to wait until the next time I use it and hopefully remember to type it out for the listeners at home.

You should see your logs coming in raw to the terminal.  Everyone forgets about ```netcat```.  Everyone **except** this kid.
