---
title: Broken MMC
date: 2019-06-24 13:30
categories:
  - blog
tags:
  - troubleshooting
  - Windows
  - Logging
  - Event Viewer
published: true
---

### MMC has detected an error and will unload the snap-in

So.  Just like me, you were trying to do some more log enrichment and massage some of those Windows logs back into their proper docile form. 
That is exactly when MMC and your beloved Event Viewer crapped itself.  

There are a number of fixes out there, but most of them revolve around custom views - the fix is very similar for Subscriptions. 

Run powershell as an admin: 

```
get-process mmc | stop-process
cd '\ProgramData\Microsoft\Event Viewer\SubscriptionFilters'
start .
```

This will kill mmc, open explorer and there is probably a single XML in there. Rename it or wipe it. 

Launch eventvwr.msc back up and enjoy your day. 

Read more if you choose: https://support.microsoft.com/de-de/help/4508640/event-viewer-may-close-or-you-may-receive-an-error-when-using-custom-v
