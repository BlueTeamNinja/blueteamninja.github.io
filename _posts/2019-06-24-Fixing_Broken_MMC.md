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
