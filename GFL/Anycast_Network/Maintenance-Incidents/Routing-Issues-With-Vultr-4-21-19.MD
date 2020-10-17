# Routing Issues With Vultr
**Created on April 21st, 2019**

Hi all,

I just wanted to make everyone aware of some of the routing issues that have been occurring recently, especially in Texas. As of right now, we primarily use [Vultr](https://vultr.com/) to host our PoP servers (Point of Presence) on our new Anycast [network](https://gflclan.com/forums/topic/36839-what-is-gfls-network/). We can only use them at the moment since we don't have our own ASN right now. We need to use their public ASN and they're the only hosting provider we've found that have good prices with many locations around the world.

Anyways, one of Vultr's main peers is Telia ([AS1299](https://bgp.he.net/AS1299)). Vultr peers directly with Telia which is demonstrated [here](https://bgp.he.net/AS20473#_graph4). From what I've seen, Telia isn't a good peer. I've noticed Telia adding many unnecessary hops to routes in my opinion which increases the chance of packet loss and higher latency. There was even a Reddit post made about this regarding Black Ops 4 servers [here](https://www.reddit.com/r/Blackops4/comments/asqaaf/beside_the_bad_netcode_the_servers_from/).

Currently, my home network is going through Telia to get to our PoP servers. Recently, it started routing me to the Los Angeles PoP instead of the Dallas PoP (I am located in San Antonio, TX, so I should be routed directly to the Dallas PoP). With that said, beforehand, one of the last Telia hops was having high latency issues which is shown within my MTR [here](http://g.gflclan.com/lt-339-04-14-2019-0658.mp4) (hop #11 and ignore the annoying latency spikes, it's Spectrum and I don't see them with my UDP connections to our game servers).

All in all, it's safe to say that Telia doesn't appear to be a good peer. Now, I've seen some peers that Vultr does peer directly with that has better routes from what I've seen. For example, GTT ([AS3257](https://bgp.he.net/AS3257)) and NTT ([AS2914](https://bgp.he.net/AS2914)). However, no matter what I do with Vultr's BGP communities on our Dallas PoP (configured via BIRD), I cannot:

1. Force our clients to go through GTT or NTT.

1. Stop announcing to Telia altogether.

I believe we should be able to do this since Vultr appears to peer with them directly, at least according to the routing propagation graph linked above. [Here](https://www.vultr.com/docs/as20473-bgp-customer-guide) is a list of Vultr's BGP communities showing that the BGP community `64600:XXXX` (X's referring to the ASN you want to not announce to). However, unfortunately, no matter what I try, I cannot change the routing. Perhaps it varies between each Vultr data center or depends on the client's ISP itself. I still have a lot to learn, haha :P

Anyways, I addressed the above issues to Vultr themselves and have been troubleshooting with them. Today I got a response in regards to myself being routed to LA instead of Dallas:

> Hello Christian,<br><br>
> Just so you know, other customers have indicated similarly situation regarding routing via Telia. Let me discuss it with the rest of the team and I will let you know first part of next week. 

I'm hoping Vultr is able to do something about this, but I'm afraid Telia is an all-in-all bad peering provider right now and apparently has always been that way. With that said, I've noticed many other routing issues from other clients in the thread I made [here](https://gflclan.com/forums/topic/36680-new-anycast-network-need-trace-route-results/). Vultr has very bad peering points, especially in Dallas and Seattle I've noticed.

The good news is, once we get our ASN we'll be able to choose other hosting providers that has better peering. I am investigating each hosting provider that suites our needs and looking at their peering. If the peers they mainly use has a good reputation, I'd say we should go with them. If we had the money (I'm really not sure how expensive this is and it's pretty complex), we could even go as far as getting our own IP transits. The difference between peering and IP Transits can be found [here](https://www.telstraglobal.com/insights/blogs/blog/peering-vs-transit). However, I don't think this is something we can:

1. Afford.

1. Need.

Though, we may consider it if we ever need it I guess.

Overall, that's it. I just wanted to make everyone aware of the issue and I will continue to work to improve the network. However, we're kind of stuck and waiting on Vultr until we get our own ASN.

If you have any questions or even advice, please reply!

Thank you.

**[Original Source](https://gflclan.com/forums/topic/39891-network-routing-issues-with-vultr/?tab=comments#comment-199580)**