# Certain Users Having Issues Connecting To Anycast Network
**Created on November 7th, 2019**

Hey everyone,

Today, I converted all our PoP servers on the Anycast network to use our [new ASN](https://gflclan.com/forums/topic/48018-new-asn-acquired-moving-popsbgp-to-new-configuration/). However, there are some users not able to route to the network after this change for some reason. I was able to get some information from one of the users experiencing this issue. This was enough information to escalate to our PoP hosting provider.

I have updated the current ticket I have opened with them and told them about this issue.

I am awaiting a reply at the moment.

If you're having issues connecting to any game servers within the **92.119.148.0/24** range, this is more than likely the reason why. If you can, please perform a trace route to the network via the following command in Windows Command Prompt and PM me the results as the more information I have regarding this issue, the better:

```
tracert 92.119.148.4
```

I apologize for the inconvenience and thank you for understanding.

**[Original Source](https://gflclan.com/forums/topic/48087-certain-users-having-issues-connecting-to-anycast-network/?tab=comments#comment-229276)**

## Update #1
**Updated on November 7th, 2019 @ 5:09 PM CST**

I've received three other results and it is all pointing towards AT&T. I believe AT&T isn't able to route to our network at the moment.

I have let our hosting provider know about this and asked if they could investigate.

Once I have an update, I will let you all know.

Thank you to the users who gave me results as well! It really helped narrow down the issue.

**[Original Source](https://gflclan.com/forums/topic/48087-certain-users-having-issues-connecting-to-anycast-network/?do=findComment&comment=229279)**

## Update #2
**Updated on November 8th, 2019 @ 9:26 AM CST**

Update from our PoP hosting provider:

> Hello Christian,<br><br>
> Forgive me for the delay. I am looking into this right now. I will update you as soon as possible.

There are now three ISPs involved here (one in Europe), so it isn't one ISP in question. I have also emailed [Hurricane Electric](https://he.net/) as I found they cannot route to the network as well via their Looking Glass [here](http://lg.he.net/).

![1](http://g.gflclan.com/lt-466-11-08-2019-4144.png) 

![2](http://g.gflclan.com/lt-467-11-08-2019-4217.png)

![3](http://g.gflclan.com/lt-468-11-08-2019-4233.png)

I do apologize for the delay on this. I am doing my best to push our PoP hosting provider to look into this and hoping I can get additional information from Hurricane Electric.

I'd imagine it is possible that Hurricane Electric just isn't announcing to the network. All ISPs listed in the reports I have received all have Hurricane Electric listed as one of their direct peers. So I believe it's possible Hurricane Electric is one of the main issues in this case.

Once I have another update, I'll let you know.

Thanks.

**[Original Source](https://gflclan.com/forums/topic/48087-certain-users-having-issues-connecting-to-anycast-network/?tab=comments#comment-229365)**

## Update #3
**Updated on November 8th, 2019 @ 11:24 AM CST**

Received an update from Hurricane Electric:

> Hello, Christian,<br><br>
> Earlier this year, a number of major networks began using strict IRR filtering for their route tables. For consistency and security, we followed suit. I see you don't have an IRR object set up for your ASN. Here's ARIN's quick start guide to IRR:<br><br>
> https://www.arin.net/resources/manage/irr/quickstart/ <br><br>
> Please let us know if you have any additional questions.

I was not aware an IRR was required for some major ISPs. I have submitted an IRR request to ARIN and waiting for it to be approved.

Thank you!

**[Original Source](https://gflclan.com/forums/topic/48087-certain-users-having-issues-connecting-to-anycast-network/?tab=comments#comment-229373)**

## Update #4
**Updated on November 8th, 2019 @ 11:52 AM CST**

The IRR request was successfully approved. I have also submitted a route object. We can delete this later if needed, but this should help with verifying where our IPv4 block is supposed to go through ARIN.

I will be testing everything out in a bit to see if there's any progress.

Thank you!

**[Original Source](https://gflclan.com/forums/topic/48087-certain-users-having-issues-connecting-to-anycast-network/?do=findComment&comment=229374)**

## Update #5
**Updated on November 8th, 2019 @ 1:45 PM CST**

The route object was also approved.

We're now just waiting for everything to propagate which could take up to 24 hours as far as I'm aware.

I've also emailed Hurricane Electric back asking if anything else needs to be done.

Thank you again for your patience.

**[Original Source](https://gflclan.com/forums/topic/48087-certain-users-having-issues-connecting-to-anycast-network/?do=findComment&comment=229379)**

## Update #6
**Updated on November 8th, 2019 @ 9:36 PM CST**

I just wanted to give a shout out to **[Hurricane Electric](https://he.net/)**. They've been working with me throughout the day to confirm everything is setup correctly and have given me **GREAT** support (we're not even a direct customer with them)! They've given me query results from the IRR database confirming that our PoP hosting provider has our ASN listed in their **AS-SET** object which is **required** from most of these ISPs which have stricter IRR filtering (those not routing to our network at this moment). With that said, apart from that, there isn't really much more we/I can do other than give it time and wait for the networks to pick up the IRR object. I also confirmed the IRR maintainer object exists for our account:

```
roy@test02:~$ whois -h rr.arin.net MNT-CMD-69
% This is the ARIN Routing Registry.

% Note: this output has been filtered.
%       To receive output for a database update, use the "-B" flag.

% Information related to 'MNT-CMD-69'

mntner:         MNT-CMD-69
descr:          Dummy description for MNT-CMD-69
admin-c:        DUMY-ARIN
auth:           MD5-PW $1$anntMjx4$h5F7JUovDWX6XVwrmBeN8/
mnt-by:         MNT-CMD-69
source:         ARIN # Filtered
remarks:        ****************************
remarks:        * THIS OBJECT CONTAINS PLACEHOLDER DATA
remarks:        * Please note that all data that is generally regarded as personal
remarks:        * data has been removed from this object.
remarks:        * To view the original object, please query the ARIN Database at:
remarks:        * http://www.arin.net/whois
remarks:        ****************************
```

I am still trying to verify if the 'route' object exists on our ASN, but I'm sure it does since it stated it was successfully created by ARIN. I just need to find out how to query it.

Here's proof showing that our PoP hosting provider (Vultr) has our ASN (AS398129) listed in their **AS-SET**:

```
roy@test02:~$ whois -h whois.radb.net AS-VULTR | grep AS398129
members:    AS397735, AS397800, AS397945, AS398085, AS398129, AS39962, AS40138, AS40181, AS40216, AS40680, AS40818, AS41281, AS41578, AS4176, AS41838, AS41954, AS41992, AS42119, AS42198, AS42214, AS42495, AS42612, AS42615, AS42692, AS42807, AS42960, AS42962, AS42978, AS42989, AS43011, AS43069, AS43090, AS43092, AS43126, AS43131, AS43138, AS43181, AS43288, AS43308, AS43449, AS43483, AS43689, AS43737, AS43927, AS44239, AS44258, AS44345, AS44364, AS44421, AS44754
```

I will admit, I am pretty new to this, but I am learning fairly quickly and this is the type of stuff I want to get into as a career :)

I do apologize for the wait again on this. I've been trying my best to get this pushed forward all day (I've been in contact with multiple providers and people regarding this just trying to learn as much as I can), but I believe I'm at a stopping point since now it's all about propagation time with other networks. I am relieved that everything seems verified on my side, though. Unfortunately, I wasn't aware we needed an IRR object and this change was implemented at the beginning of the year. This shouldn't happen again in the future as well now that I'm aware. This is a pretty big deal in itself (having your own ASN that is). We now have complete control over our network with this change.

If you have any questions or concerns, please let me know. I'm hoping tomorrow the traffic can start flowing through for users experiencing issues now. It should be sooner. If I don't see any changes tomorrow, I will contact Hurricane Electric again to see if there's additional information/steps missing. Though, they said we should be good to go.

Thank you!

**[Original Source](https://gflclan.com/forums/topic/48087-certain-users-having-issues-connecting-to-anycast-network/?do=findComment&comment=229439)**

## Update #7
**Updated on November 9th, 2019 @ 11:25 AM CST**

Alright, so I just wanted to provide yet another update. It still appears the ISPs in question aren't routing to our network. I've sent another email to Hurricane Electric to see if they can pin down the issue since they're one of the providers not routing to us.

I've been doing a lot of research this morning and querying many databases to try to find as much information as I can. I believe there are three possibilities here which I will list below:

1. We still need to give it more time for all ISPs to see the new IRR policy along with the ASN itself. According to [here](https://stat.ripe.net/AS398129#tabId=routing), only 84% of RIS peers are seeing the network.

1. Our PoP hosting provider's ASN is not listed as a peer under our IRR policy. I don't believe this is required and Hurricane Electric told me it wasn't. However, I still think it's a possibility.

1. The existing ROA (Route Origin Authorization) signed with our **92.119.148.0/24** block was against our PoP hosting provider's ASN. We haven't signed any ROA with the new ASN. I have requested an RPKI with ARIN. Once this is approved, we can sign a ROA for the new ASN. The RPKI request takes up to two **business** days to process. I'm crossing my fingers and hoping they can magically approve it over the weekend.

I'm really hoping it's either situation #1 or #2 (#1 preferably, but less likely). I'm going to wait for a reply back from Hurricane Electric before proceeding with any other steps unless the RPKI gets processed. According to Hurricane Electric's routing filtering algorithm which can be found [here](https://routing.he.net/algorithm.html), only the IRR policy was included. I don't see anywhere it mentions the ROA needing to be valid for the ASN in question.

### Conclusion (Non-Technical)
As it stands, unfortunately, I have no ETA on when things will be up and running for the players affected. I'm trying my best to get this moving forward as fast as possible and again, I do apologize for the wait on this. I wasn't aware of the ROA not being valid until late last night when the route propagated on HE's BGP site [here](https://bgp.he.net/).

If it's confirmed we need to wait a few more days, I'm going to try setting up a temporary server with a VPN and allow our users to play through that in the meantime (I'll be setting up individual users as well). This will route you through another server that can reach the network without any issues. Keep in mind though, your latency will more than likely be higher with this (depending on your location and the VPN's location). But it is the best workaround I will be able to offer until the main issue is resolved.

Once I have another update, I'll let you guys know. In the meantime, I'd suggest trying out some of our servers that are hosted by NFO.

### Technical Information I've Gathered
This is some of the technical information I've gathered about the situation. I'm not expecting really anybody in GFL to understand this considering this is pretty complicated, but who knows, maybe somebody will :)

Firstly, I just want to provide our AS which is [AS398129](https://bgp.he.net/AS398129) and our IPv4 block which is [92.119.148.0/24](https://bgp.he.net/net/92.119.148.0/24). Secondly, I want to provide these set of databases that I've found EXTREMELY helpful:

* [Hurricane Electric's BGP](https://bgp.he.net/) - Extremely useful for looking up IPv4 blocks and ASNs. Includes routing information, prefixes, IRR policies, and more. I use this a majority of the time.

* [RIPE Stat](https://stat.ripe.net/) - Useful for looking up helpful information on the ASN and IPv4 block specified. Shows TON of information I will list below along with statistics I've provided above.

* [RADb](https://www.radb.net/) - Useful for looking up ASN's IRR policies. Unfortunately, we aren't listed here at the moment and I'd imagine it's due to propagation time (e.g. the ARIN IRR database hasn't synced with theirs).

* [IRR Explorer](http://irrexplorer.nlnog.net/) - This is what Hurricane Electric provided me with for looking up our IRR along with where our IPv4 block is originating at.

* [Hurricane Electric's Looking Glass](https://lg.he.net/) - I've been using this to test routing to our network. Obviously, it has been timing out. But once this starts replying, it will be a good sign.
 
#### Proof Our ROA Is Invalid
On a number of databases I listed above, our ROA is invalid. I'm just going to screenshot the RIPE Stat database [results](https://stat.ripe.net/92.119.148.0%2F24#tabId=routing) specifically though:

![4](https://g.gflclan.com/1325-11-09-2019-NLRvTU2o.png)

#### Proof Our IRR Exists
Apart from the results from the follow command which is provided in my previous reply:

```
whois -r rr.arin.net MNT-CMD-69
```

Most of the databases besides RADb have picked up our IRR policy to my understanding.

Here are the [results](http://irrexplorer.nlnog.net/search/398129) from IRR Explorer (which only mentions the ROA is invalid, which is stated above):

![5](https://g.gflclan.com/1326-11-09-2019-JkHdTE4l.png)

Here is the results from HE's BGP website showing our IRR is valid for the route to the new ASN:

![6](https://g.gflclan.com/1327-11-09-2019-jE79Cins.png)

One thing that is strange is when viewing the ASN itself, the IRR [tab](https://bgp.he.net/AS398129#_irr) is still blank. I have mentioned this in my email to Hurricane Electric since it appears it was also updated late last night (after the IRR was processed):

![7](https://g.gflclan.com/1328-11-09-2019-2a9sFevI.png)

#### Route Propagation Populated
It appears the [route propagation](https://bgp.he.net/AS398129#_graph4) also populated which is a good sign:

![8](https://g.gflclan.com/1329-11-09-2019-WmtQ5MrS.png) 

#### Proof The Routing Object Exists Through The IRR
This is proof I've found that shows the route object exists from the RIPE Stat [database](https://stat.ripe.net/widget/whois#w.resource=92.119.148.0%2F24) (I was trying to verify this last night):

![9](https://g.gflclan.com/SMrjm9TNPS.png)

That's all the information I've concluded so far. If you have any questions, please let me know!

Thank you.

**[Original Source](https://gflclan.com/forums/topic/48087-certain-users-having-issues-connecting-to-anycast-network/?do=findComment&comment=229480)**

## Update #8
**Updated on November 9th, 2019 @ 6:55 PM CST**

I received an email from Hurricane Electric. They told me while having a ROA is optional, if you have a ROA already setup that is invalid (which is our case), it will cause more issues than not having an IRR policy (more networks would have issues with our network). The ROA that is invalid is on RIPE's side and the company we purchased the IPv4 block from only has access to manage it (and I'd assume they can remove it as well). I emailed our IPv4 block provider around 5 - 6 hours ago asking if they could please remove the invalid ROA. Hopefully they respond back soon. When next week rolls around, I'll more than likely proceed with the ROA through ARIN and get one that is valid for the our ASN.

Apart from that, Hurricane Electric also stated our PoP hosting provider isn't announcing to all locations and this is the reason Hurricane Electric cannot see the network's announcement whatsoever. It is not even getting to their filtering system. This would be on our PoP hosting provider's end. I've sent our PoP hosting provider an email regarding this and told them this needs to be looked into as soon as possible.

On the bright side, the IRR policy seems to be correctly setup and there aren't any more steps aside from the two above. Here's what I'm hoping happens in the next day:

* Our IPv4 block provider removes the invalid ROA which is pointing towards our PoP hosting provider's ASN. Hurricane Electric stated networks should pick up the change in a few hours.

* Our PoP hosting provider corrects the issue on their end that is resulting in our ASN not being announced in specific locations/sets.

I'm thinking there may be a chance our PoP hosting provider isn't announcing to certain locations because of the invalid ROA. But I cannot confirm (I'll need them to).

### EDIT
I also had Hurricane Electric rule out situation #2 from my previous post. We do **not** need to list our peers/upstreams in the IRR policy via import/export statements. So that's good. Those were used for informational purposes by older tools.

Once I have another update, I'll let you all know.

Thank you for your patience.

**[Original Source](https://gflclan.com/forums/topic/48087-certain-users-having-issues-connecting-to-anycast-network/?do=findComment&comment=229525)**

## Update #9
**Updated on November 9th, 2019 @ 7:19 PM CST**

Another update, I just received this from our IPv4 block provider:

> Hi Christian,<br><br>
> thank you for letting me know - I have updated the ROA record for 92.119.148.0/24 to AS398129.

It looks like they just changed the existing ROA record to point towards our ASN from their end. If this is the case, we probably won't even need to proceed with a ROA signing with ARIN.

I'm going to push our PoP hosting provider to look into their issue. There's a possibility the issue was linked with the invalid ROA, so there's a chance things will start working again soon. We'll see.

Thank you!

**[Original Source](https://gflclan.com/forums/topic/48087-certain-users-having-issues-connecting-to-anycast-network/?do=findComment&comment=229531)**

## Update #10
**Updated on November 9th, 2019 @ 7:59 PM CST**

Receiving reports the networks that were experiencing issues are starting to route to the network again! :D

Hurricane Electric is also able to reach the network now:

![10](https://g.gflclan.com/1334-11-09-2019-4ReR7F23.png)

I guess it only took an hour or so for the ROA record to update!

Please let me know if anybody is having issues still.

Thank you!

**[Original Source](https://gflclan.com/forums/topic/48087-certain-users-having-issues-connecting-to-anycast-network/?do=findComment&comment=229567)**