---
title: Mapper Parser Exception REVISITED
date: 2019-08-30 18:30
categories:
  - blog
  - howto
tags:
  - graylog
  - Logging
  - Troubleshooting
published: true
---

# ARGH!  Parser Exception!

## My tactics to quickly hunt a pesky field

You're going along with your day, eating some amazing desk-ramen and listening to the soothing sounds of the locals and their [desk pops](https://www.youtube.com/watch?v=2U3Ka0ECbPE) 
when you pull up your Overview page in Graylog and see a bunch of suck. 

| There were 12,520 index failures in the last 24 hours.

*OH NOT YOU DIDN'T*

A lot of times you can get a clue from the field names but in this case I had no idea.  This was the message: 

```
{"type":"mapper_parsing_exception","reason":"failed to parse field [rt] of type [date] in document with id '67b4e3f0-cb49-11e9-815d-005056aed827'","caused_by":{"type":"illegal_argument_exception","reason":"Invalid format: \"Aug 30 2019 13:12:29 GMT-04:00\""}}
```

I know only 2 things: 
1.  It's a fieldname of *rt*
2.  It annoys me.

I searched for other logs with rt and I had a few wildly different sources so that didn't help me at all either. 

This means we have to blindly fix it and go look for the silly box that is dumping in a date format that elastic doesn't like. 
### Easy way

Create a custom index map to text so the parser can go back to licking the glue out of the jar. 

I found a few guides on this but they were a bit out of date. 
I can tell from the error I'm looking for index graylog_159 so I VI'd this file: 

* Note the `"type" : "text"` line.  A lot of guides have "String" but ES has sent that packing in favour of 'Text' or 'Keyword'. 

```
$vi rt-troubleshoot.json
{                                                         
        "template" : "graylog_*",                         
        "mappings" : {                                    
                "message" : {                             
                        "properties" : {                  
                                "rt" : {                  
                                        "type" : "text"   
                                }                         
                        }                                 
                }                                         
        }                                                 
}
```

Next step is to add that file to the custom index mapping: 

```
curl -X PUT -d '@rt-troubleshoot.json' 'http://localhost:9200/_template/graylog-custom-mapping?pretty'
```

But NO!  THE GODS HAVE OTHER PLANS FOR YOU!

```
{
  "error" : "Content-Type header [application/x-www-form-urlencoded] is not supported",
  "status" : 406
}
```

### Easy fix. 
Add the custom mapping take 2:
```
curl -X PUT -d '@rt-troubleshoot.json' 'http://localhost:9200/_template/graylog-custom-mapping?pretty' -H 'Content-Type: application/json'
```

I always rotate the index just to be safe.  

Now you'll see the ugly little *rt* field show up with its silly date format.  I'm not above name-shaming, this is Trend Micro and they arebeyond notorious for the ugliest syslog messaging I have to deal with.  They are great at fixing things but you eventually get tired of sending in more messages. 

Proof:  Here is an [EASY](https://help.deepsecurity.trendmicro.com/10/0/Events-Alerts/syslog-parsing.html) Trend Micro Syslog document.  They have much more convoluted ones. Also notorious for doing JSON INSIDE of K=V messages INSIDE of CEF. Not kidding.

I digress.  Back to the game. 

Once you have done the above, you can simply search for your funky field `_exists_:rt` and go find the guilty culprit.  

For posterity - this is the pipeline rule I applied: 
```
rule "Cleanup - Trend DDI Date"

when
    has_field("rt") AND 
    to_string($message.dvc) == "10.25.100.124"
then
    let new_date = parse_date(
        value: to_string($message.rt), 
        pattern: "MMM dd yyyy HH:mm:ss 'GMT'Z"); 
   // This particular product ships logs at intervals - so I want to replace the timestamp with this rt field
    set_field("timestamp", new_date); 
    remove_field("rt");
end
```
