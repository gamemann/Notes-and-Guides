# Anycast Filtering Notes, (D)DoS Attacks, Plans To Mitigate, And More 
**Created on May 25th, 2020**

Hey everyone,

I'm creating this thread to store my notes regarding findings with filtering rules that I plan to implement into our Anycast network when Dreae and I complete [Compressor V2](https://gitlab.com/srcds-compressor/compressor/-/issues). I also address recent (D)DoS attacks on the network and things we're implementing in the future to mitigate these attacks.

I am making this thread public to educate others or perhaps myself by somebody who may have looked into this before. If this is an interest to you, please feel free to reach out or post a reply :) The more help with these findings, the better.

## Our Problem
We've been getting continuously hit by (D)DoS attacks recently and since we don't have many filtering rules applied at the moment (besides generic SRCDS hardening rules), all malicious traffic is forwarded to the game server machines creating a single-point-of-failure. While there is no way to **completely** remove this single-point-of-failure because game servers simply can't be "Anycasted", we can apply filtering rules to drop as much malicious traffic as possible on the POP-level.

## Why This Isn't As Simple As People Think
I've seen some people complain recently about the network and how it should be easy for us to find a new hosting provider or get better (D)DoS protection. While I understand the frustrations from these users, I just wanted to briefly explain why this is a lot more complicated than it looks. It's simple actually, we own our [Anycast](https://en.wikipedia.org/wiki/Anycast) network (we have our own [IPv4 block](https://bgp.he.net/net/92.119.148.0/24) and [ASN](https://bgp.he.net/AS398129)). This network sits in-front of all our game servers and is responsible for forwarding traffic to our game servers. This network has been responsible for a lot of the success we've had in the past year as well and it's really the thing I'm most interested in on GFL (on a technical-level). Therefore, removing the network isn't really an option and I will admit, I would probably lose a lot of motivation if it was.

Since we own the network, we need to also implement processing and filtering software. As of right now, we use [Compressor](https://github.com/Dreae/compressor), a project made by [Dreae](https://github.com/dreae) (one of the smartest people I've seen in networking and network programming). Unfortunately, Compressor V1 doesn't include in-depth filtering rules at the moment. This is our main issue. With that said, since Compressor V1 doesn't connect to a backbone to handle game server connectors (it relies on a config file on each POP), it's not easy to spin up many POP servers as well due to maintenance. These are all things that will be tackled with Compressor V2 (read below).

To conclude, adding protection to our network is a lot harder than it looks. Thankfully, this is something Dreae and I are very interested in. I'm still pretty new to this area as well, but I've made a lot of success the past few months with network programming and so on. Therefore, we're doing everything we can to protect the network. However, Compressor V2 will take some time to make. Keep in mind, hosting companies pay developers a ton of money to implement filtering rules and so on for (D)DoS protection. Dreae and I are both doing this for free and open-source.

I do understand the frustration regarding downtime from malicious attacks, but I just wanted to go over why it's a lot harder than people think to upgrade our network capacity and (D)DoS protection. This is still nothing compared to what we used to see back in 2014 - 2016. We were having servers getting nulled daily for 4 - 12 hours at a time back in those days (a null route is when a hosting provider sends all traffic to a specific IP to a blackhole via BGP usually). Thankfully, servers are very hard to null route on an Anycast network since we're using the overall network capacity from multiple data centers.

## Temporary Solution
Since Compressor V2 won't be available for some time, I'm implementing a temporary solution in hopes to block this malicious traffic at the POP level. Due to the nature of Compressor and XDP, there's no real way to perform packet captures on the POP level. Technically, you could use `bpf_trace_printk()` (a [BPF Helpers](http://man7.org/linux/man-pages/man7/bpf-helpers.7.html) function) to print to `/sys/kernel/debug/tracing/trace_pipe` within the XDP program, but the output looks horrible, only three arguments are supported, and performance would be highly impacted. However, you may do packet captures on the game server machine. One thing to note is the format is in IPIP (basically an outer IP header is added with the size of 20 bytes) and sometimes that's hard reading with something like Wireshark or capturing with `tcpdump`. For example, I still haven't found a way to sort by the inner IP header's source/destination on Wireshark and `tcpdump`. I'm pretty sure it's possible with `tcpdump` though. I just need to find out how.

Anyways, I hard-coded these filters into Compressor V1.

The challenge I've mostly faced with implementing these filters is trying to whitelist Steam traffic. I was able to do so after some time and multiple packet captures. I had to do this with each game we host under the network at the moment because some games use TCP to communicate with Steam while others use UDP. It's a mess in my opinion, but I believe I got it figured out and I've tested this 5 times per game with all successful attempts.

As of right now, I'm waiting for our POP hosting provider to fix our BGP issue which is causing a roadblock with deploying the temporary solution. I'm hoping to move away from this provider eventually or at least get more hosting providers in the future (read below) so we aren't relying on just our current provider (their support has been **horrible** in my experience). Since we've got our own ASN, we're considered multi-home and can find more hosting providers. I've already done this when we setup our Hivelocity POP in NYC a few months back (this was discontinued due to unrelated issues and pricing). Therefore, I know what I need to do for the future.

This temporary solution should stop forwarding TCP/UDP floods along with reflection attacks to our game server machines. There's still a chance the POP could be overloaded and if it is, only traffic routing to that POP will be affected. From what I've seen, the attacks recently are just typical UDP floods. There's nothing special about them from what I've seen.

It's also important to get the TC BPF program I made [here](https://github.com/gamemann/IPIPDirect-TC) working on GS12 so we don't have another single-point-of-failure. I'm still waiting for our hosting provider to look into why certain upstreams are filtering traffic spoofed as our Anycast network (needed for the TC BPF program to work properly). I will request an update on this tomorrow. All other machines are using this program successfully.

## Permanent Solution
The permanent solution to this issue will come with Compressor V2. While Compressor V2 will support both a whitelist and blacklist approach, we (GFL) will be taking a whitelist approach to all game server traffic (client => game server) which is a lot safer in my opinion.

The goal is to make it so no malicious traffic will ever be forwarded to the game server machines. If this is the case, the (D)DoS protection will primarily rely on network capacity and resources. The plan with this is we're going to be heavily expanding our network by getting 60 - 100+ POPs along with 2 - 3 solid new hosting providers (I've sent many emails to hosting providers in the past and continue to do so). With Compressor V2, we're planning to automate literally everything. Therefore, all new POPs will be setup automatically with Compressor V2 using API scripts, etc. This shouldn't be too difficult and we'll just need a template for the BIRD config for BGP.

With that said, we plan to implement a monitoring system that will check the resource usages (CPU and network) at each POP and location. If a location or POP is found with high resource usages, we'll automatically spin up temporary POPs to load balance the traffic. Most hosting providers load-balance traffic between POPs at each location such as our current provider (a round-robin method). This is so we're not wasting money on POPs we don't necessarily need while attacks aren't going on, but if there's a long (D)DoS occurring, temporary POPs will be spun up to absorb the attack.

## Filtering Structure For Compressor V2
I'm going to share an image of my idea for implementing filtering rules and modules (for whitelisting clients after validating a game server handshake) into Compressor V2:

![Discord Conversation](https://g.gflclan.com/2775-05-23-2020-nbC0Gn6e.png)

I won't go into detail on this since that's out of this thread's scope. However, I will provide more detail later and I'll likely post something [here](https://gitlab.com/srcds-compressor/compressor/-/issues).

Note that malicious traffic will be dropped via XDP-native (one of the fastest hooks in the Linux networking path besides **maybe** the [DPDK](https://www.dpdk.org/)). I had a discussion on the XDP Newbies Mailing List [here](https://marc.info/?l=xdp-newbies&m=159034844410576&w=2). David Ahern (a super intelligent guy) confirmed that XDP-native is still a lot faster than XDP-generic even on the **virtio_net** driver (what we'll be using since we're going to have a VPS as each POP server). Beforehand, I was under the assumption XDP-native would only be useful if the hosting provider offloaded packets off their cluster's NIC directly to the VPS. This is not the case and there's actually a separate XDP mode for this (**XDP_FLAGS_HW_MODE**) along with only one NIC driver being supported.

## Rate Limiting And Sent/Received Ratio Thresholds
The first line of defense we'll have on the network is rate limiting and sent/received ratio thresholds.

The first part is pretty self explanatory. We're going to be limiting the amount of packets per second (and bytes per second) a source IP can send to our network. If they're found sending more than the thresholds, we'll add them to the XDP program's blacklist map to have them dropped via XDP-native for a certain amount of time (probably 30 minutes or something since they're definitely malicious traffic).

The second part is going to be a ratio for the amount of packets a source IP sends and receives. Empty UDP floods not targeting a specific service will likely not receive a UDP response from our game servers. Therefore, we'll start blocking the source IP after a certain threshold (e.g. 200 sent packets per response). We'll have to find a way to exclude when a game server is down for example as well.

**Note** - Legitimate traffic based off of the whitelist handshakes (explained below) will be accepted before the blacklist map drops traffic. This makes it so an attacker can't start spoofing as a random source IP each packet and possibly get the IP address of somebody on the game server and impact them. We'll still have fair rate limits applied to users on the game servers as well. Therefore, spoofing as a certain player IP will still not be able to take down the network. It may impact that client though (if it does, they'll have to change their public IP). I'd be surprised to see an attacker go that far (I haven't seen it before), but better to be prepared :) 

## Cached Packet Types
Attacks targeting a specific service can be damaging (e.g. the A2S_INFO [query](https://developer.valvesoftware.com/wiki/Server_queries#A2S_INFO)). I've done a lot of pen-testing and it's super easy to take down a server when sending many A2S_INFO requests for example if the game supports these queries (all of our game servers do). Since the server replies to these requests, it's usually easy to make the server use all of its resources by sending a high amount of low throughput packets (e.g. the A2S_INFO request payload only needs to contain bytes `0xFF 0xFF 0xFF 0xFF 0x54`). To prevent this, we will be caching these specific packets. This'll make it so the attackers can't target a specific service within our game servers and cause it to use all of its resources. The load will be distributed all throughout the POPs instead.

**Note** - Some games support caching the A2S_INFO query via an extension. I haven't been able to find a working extension yet, but theoretically, if it was compiled correctly, it would be a decent defense. It's still better to cache the packet on the POP level though since it'll allow you to distribute the load throughout all the POPs instead of having the game server respond to each with the cached response. You can still take down the game server if you use the extension from my findings.

**Note** - Compressor V1 currently caches the A2S_INFO response as well. However, this is hard-coded into Compressor V1 and with Compressor V2, you will have the ability to cache certain packets easily by adding them via a form in the panel. More info on the plan to implement this can be found [here](https://gitlab.com/srcds-compressor/compressor/-/issues/12).

## How To Whitelist Outbound Game Server Traffic?
Since we're taking a whitelist approach with Compressor V2 along with a module system, the first thing I thought about is how we're going to whitelist traffic that the game server sends out (e.g. Steam traffic, API requests from the game servers, MySQL connections, etc). Due to how we're going to have our Docker containers and network namespaces setup, outbound traffic the server sends will be encapped in FOU and will be processed on the POP by the two TC BPF programs I made [here](https://github.com/gamemann/Compressor-V2-FOU-Wrap-Unwrapper). This does NOT include traffic from the game server going back to the players since that's going through a separate route and device not being encapped with FOU (e.g. the veth pair/bridge connecting the network name space to the main host).

[FOU](https://lwn.net/Articles/614348/) is similar to IPIP, but it includes an outer UDP header of 8 bytes as well as the outer IP header (20 bytes). The outer UDP header's source and destination port will represent the FOU port. This supports UDP and TCP for the inner headers.

Therefore, on the POP server, we'll add the inner IP header's destination address to a BPF map for the XDP-native program whitelist for any valid incoming FOU requests for a certain period of time (let's say 45 seconds). We don't want to permanently whitelist these IPs, at least that's not an approach I'd like in the case the malicious attacker spoofed the IP as something on that whitelist.

Ez pz :)  (not all that easy in programming terms, though, due to certain checks we'll need to implement, etc)

## Whitelisting Game Server Traffic
Now this is when things get interesting :) I've been doing plenty of packet captures just trying to understand the handshake process for players to our game servers. I've been using Wireshark to inspect these packet captures, but using tcpdump to record the actual packet captures on our game server machines. I still need to do a lot more digging into the Steam Networking [library](https://partner.steamgames.com/doc/features/multiplayer/networking) to confirm everything below, but here's what I have so far.

Also, here's a screenshot of all my packet captures!

![Game Server Traffic #1](https://g.gflclan.com/2744-05-25-2020-sg8mwcDg.png)

Quite a bit :D

### SRCDS Games
When a client connects to standard SRCDS game servers (in our case, FOF, GMod, CS:S, TF2, and CS:GO), they're usually using 27005 as the default source port. This can be changed by adding `+clientport xxxxx` to your launch options. However, there is rarely ever a need to change it.

When I first connected to my test CS:S server, I sent a request with `0XFF 0XFF 0XFF 0XFF 0X71` as the headers:

![2](https://g.gflclan.com/2745-05-25-2020-UyV5MQas.png)

Afterwards, the server sends back a response with `0XFF 0XFF 0XFF 0X41` set:

![3](https://g.gflclan.com/2746-05-25-2020-iV48uUHb.png)

Then, the client sends a response that differs per game I believe (`0XFF 0XFF 0XFF 0XFF 0X6B` for CS:S, though):

![4](https://g.gflclan.com/2747-05-25-2020-9q8H2llh.png)

Finally, the server sends back a response with `0XFF 0XFF 0XFF 0XFF 0X42` set:

![5](https://g.gflclan.com/2748-05-25-2020-SxabioFf.png)

While I haven't inspected the actual data yet (I plan on doing so), I believe the `0xFF 0XFF 0XFF 0XFF 0X42` request the game server sends back to the client is an indicator that the client is legit and valid. Therefore, with our module system in Compressor V2 and utilizing TC ingress and egress filters, I believe we can whitelist traffic when the server responds to the client with a the first five bytes being `0XFF 0XFF 0XFF 0XFF 0X42`.

I will dig more deeply into this later to confirm as mentioned above. However, that's my theory as of right now.

### Rust
Rust is pretty easy to detect as well. It uses the [RakNet](http://www.jenkinssoftware.com/) networking library that I'm not that familiar with yet, but I do want to look into it more in the future.

The first request the client sends contains a single header byte of `0X05`:

![6](https://g.gflclan.com/VjQj3rUb1S.png)

Afterwards, the server sends a response with a single header byte of `0X06`:

![7](https://g.gflclan.com/flJIrUl4hi.png) 

Then, the client responds back with a single header byte of `0X7`:

![8](https://g.gflclan.com/VP9x8ydJvp.png)

And finally, the server responds back to the client with a single header byte of `0X08`:

![9](https://g.gflclan.com/3n03G7mZCZ.png)

**Note** - I blocked out irrelevant information since I performed this packet capture on a public Rust server. Therefore, I didn't want anybody to get my public IPv4 address.

Anyways, I believe we can whitelist legit Rust player IPs based off of the destination IP of the response the server sends with `0X08` set as the single byte header.

I still need to look into the RakNet library to confirm this specific response means the connection is valid and accepted.

### Other Games

Other games I will be looking into the near future for this include:

* Killing Floor 1/2.

* Left 4 Dead 1/2 (they should be using the standard SRCDS methods, but I just want to confirm).

* Arma 3.

* Red Orchestra 2.

* DayZ.

* FiveM.

* RedM.

* And more.

## Conclusion

I know this is a very long thread, but I hope some people find the discoveries interesting. I also hope this clarifies some of the recent statements regarding our network and shows that Dreae and I are doing everything we can to strengthen our (D)DoS protection.

If you have any questions, feel free to ask!

Thank you.

**[Original Source](https://gflclan.com/forums/topic/57006-anycast-filtering-notes-ddos-attacks-plans-to-mitigate-and-more/?tab=comments#comment-258559)**