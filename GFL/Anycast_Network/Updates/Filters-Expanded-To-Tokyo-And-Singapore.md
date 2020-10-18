# Filters Expanded To Tokyo And Singapore POPs
**Created on June 24th, 2020**

Hey everyone,

I just wanted to let you know that I've extended the new filters I've been working on (outlined [here](https://gflclan.com/forums/topic/57525-preparation-and-plan-for-new-filter-rules-on-anycast-network/)) to our Tokyo and Singapore POPs. The new POPs are actively retrieving a list of all whitelisted services as well and I've made changes to my IP/ASN Go project here to add checks so things are less likely to break (added HTTP status code support, better error handling, and more).

Once everything is deployed, we'll be giving Directors and all of TAs access to whitelist services for these POPs. This will help a lot since things won't rely on only me.

I've been monitoring traffic and players appear to be connecting without any issues.

The new filters are now deployed to Sydney, Tokyo, and Singapore. If things are stable for a few days, I will expand these filters to all of Europe as well.

Thank you!

**[Original Source](https://gflclan.com/forums/topic/58168-filters-expanded-to-tokyo-and-singapore-pops/?do=findComment&comment=262939)**