# Bifrost's Nearly Finalized Plans
**Created on November 23rd, 2020**

Hey everyone,

This thread is serving as a low and high level overview of Bifrost. [Dreae](https://github.com/dreae) will be reading over this as well and will be seeing if anything can be improved, etc. Once everything is approved, I plan to document our plans into either a separate repository [here](https://github.com/BifrostTeam) or as a "Wiki" in an existing repository (using Markdown).

These plans may be slightly altered as well, but I believe this is very close to being completely finalized.

## What Is Bifrost?
Bifrost is a project [Dreae](https://github.com/dreae) and I are creating that will act as a firewall and will also be used on GFL's Anycast network. This project will be responsible for forwarding legitimate traffic to our game server machines and dropping malicious traffic as fast as possible (originating from (D)DoS attacks for example). Bifrost was formerly known as CompressorV2 which was supposed to be a revamped version of [Compressor](https://github.com/dreae/compressor) created by Dreae. Compressor is currently utilized on our Anycast network and POPs in Europe/Asia have filters I've implemented into Compressor outlined [here](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Anycast_Network/Notes/Anycast-Filtering-Notes-DDoS-Attacks-Filtering-And-More.MD) and [here](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Anycast_Network/Updates/Update-With-New-Filters-On-Compressor.MD).

## Why Has Development Taken So Long?
Development for Bifrost started around a year ago or so. However, it has taken a very long time to come up with a game plan and a lot of this has been my fault along with time restraints on both Dreae and I's side (I still have my full-time job, GFL itself, and the Anycast network's infrastructure to maintain). With that said, a year ago when development started, I was not experienced with low-level C programming and network programming. In fact, I didn't start digging deeply into this until March of this year and I've made all of my progress mostly open-source via my GitHub profile [here](https://github.com/gamemann/).

With that said, the last few months I've been trying to look into the fastest ways possible to drop packets and to be honest, this has overcomplicated the project since.

## Thank You For The Support!
Before going into my plans for Bifrost, I just wanted to thank everybody including those on Linkedin for the amount of support they've shown regarding this project. The last few months I've been posting updates regarding Bifrost and network/programming in general which started receiving more attention/support than I had imagined. This has given me a huge motivation boost and I've also been learning a lot from others as well (I still feel I have so much to learn and I'm sure I'll never stop feeling that way which is good and bad sometimes haha, but I feel I've made great progress as well since March). I wanted to give shout outs to the following individuals for being specifically helpful to Bifrost and GFL's Anycast network development:

* [Dreae](https://github.com/dreae) - Dreae is who introduced me to network programming and somebody I highly look up to. If it weren't for him, I likely wouldn't have made the Anycast network and have such a big passion for mixing networking and programming today.
* Renual - Renual is the owner of [Gameserverkings](https://www.gameserverkings.com/) (GSK). We've started using GSK in the middle of this year for our game server infrastructure along with some of our Anycast POPs. Renual has also been extremely helpful in regards to advice for Bifrost, BGP, (D)DoS mitigation/prevention, networking in general, and so much more (he is also very busy in general and still takes the time to go into in-depth detail on so many things I ask him about). He is one of the smartest people I've talked to and I've been learning so much from him. I'd suggest giving [this](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Infrastructure/Updates/Big-Game-Server-Machine-Upgrades.md) thread a read to see how beneficial GSK has been to us.
* [Pavel Odintsov](https://github.com/pavel-odintsov/) - Pavel is a CloudFlare Engineer who connected with me after discovering one of my posts on Linkedin and has since then been very supportive of my projects. It meant a lot after seeing him support my projects and I knew of him beforehand when I discovered his (D)DoS Analyzer software called [FastNetMon](https://github.com/pavel-odintsov/fastnetmon). He also recently suggested FreeBSD's [netgraph](https://www.freebsd.org/cgi/man.cgi?netgraph(4)) for the project which I will go into detail below.

## Choosing What To Drop Packets With

The most complicated part when coming up with a game plan for Bifrost was choosing what networking path/hook/library to use to drop packets as fast as possible. I made a thread [here](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Anycast_Network/Notes/XDP-or-the-DPDK-for-Bifrost.MD) that went into detail on XDP vs the DPDK from my point-of-view. However, recently, after talking to Pavel as mentioned above, he introduced me to FreeBSD's [netgraph](https://www.freebsd.org/cgi/man.cgi?netgraph(4)) ([this](http://www.netbsd.org/gallery/presentations/ast/2012_AsiaBSDCon/Tutorial_NETGRAPH.pdf) gives an easier overview of what netgraph is in my opinion).

To be honest, all three of these were really great options and in regards to GFL, all perhaps overkill for what we have right now (we're not making this firewall for just GFL, though). The reason why Pavel suggested netgraph specifically was because Dreae and I had plans to make Bifrost's back-end basically a packet engine that would be able to use ingress and egress hooks from XDP, the DPDK, TC (Traffic Control), netfilter, and more. We had also planned to make our own programming language that would be used to create modules within Bifrost (these modules would utilize networking hooks as mentioned earlier). It seems netgraph would have been great to use for this, though, I'm unsure how this would be implemented into the DPDK for example.

Unfortunately, due to FreeBSD's netgraph being very new to me and its complexity, I've decided not to use this for Bifrost. It also didn't have much documentation, but I was able to find many open-source modules made for it on Google [here](https://www.google.com/search?q=site%3Agithub.com+netgraph). However, I do plan on using it in a future firewall I plan to build after Bifrost.

This narrowed down the choices to the DPDK or XDP. They both have their pros and cons. When it comes to performance specifically, the DPDK would be the better choice due to busy-polling, etc. However, this would require dedicated cores and since GFL's Anycast network has many POPs with two cores, to my understanding we'd only be able to utilize one for the DPDK application. With that said, we will likely be getting NICs for our GSK POPs that support XDP offload (which compiles into the NIC's hardware to my understanding) and therefore, I believe this would be even faster than the DPDK (though, I haven't dug deeply into this yet).

With the above points stated and also the fact that I still haven't dug deeply into making applications for the DPDK, I decided it'd be best if we used a combination of XDP, TC/BPF (Traffic Control), netfilter, and NFTables for Bifrost. The DPDK is still something I will be learning in the future and making firewalls with.

## What Will Make Bifrost Stand Out?
I personally haven't seen many open-source firewalls that aim to do what Bifrost intends to. However, a few key things I believe will make Bifrost stand out include:

* Using XDP (in either offload, native, or generic modes depending on what the NIC/NIC driver supports) to drop malicious traffic via firewall rules using most packet characteristics besides dynamic payload matching which was attempted [here](https://github.com/gamemann/XDP-Dynamic-Payload-Matching), but failed. The XDP program will also include rate-limiting.
* BGP [Flowspec](https://blog.marquis.co/what-is-bgp-flowspec/) support for attacks detected with source/destination IPs, source/destination ports, and more. This will result in upstreams dropping the specific traffic instead of the server itself which is better.
* TC/BPF/netfilter modules for whitelisting traffic via handshake sequences (for game servers) and more. These TC BPF programs will add to the XDP BPF map if a certain packet's characteristic isn't changing that the XDP program supports blocking for a certain amount of time. These modules can be enabled on a per destination IP basis (or depending on other characteristics within the packet) within our back-end portal for servers that are supported.
* Hoping to make a netfilter kernel module that inspects all traffic and looks for anomalies that can be caused by a (D)DoS attack. These inspections can happen for all traffic or traffic containing certain characteristics.
* Creating a netfilter kernel module to inspect certain packets depending on the configuration via payload and pushing firewall rules if we find a common characteristic so XDP can drop the packets. Otherwise, netfilter will be dropping the packets.
* An IP/port forwarding module via NFTables that can be used to forward traffic to services that cannot be completely Anycasted (such as game servers for example) or for load-balancing purposes.
* The ability to cache certain packets for a certain amount of time in seconds which should help limit damage done by layer-7 attacks on certain applications.
* Being able to manage global/server-specific configuration in a back-end web portal that the servers communicate with to retrieve information. I want as many things being configurable from this back-end portal as possible.
* Server-specific statistics that will be reported to and graphed in Bifrost's back-end including network statistics such as packets per second and bytes per second (conversions will be supported within the back-end portal for kbps, mbps, and gbps).
* A very flexible REST API that can be used to change any global or server-specific configuration in the matter of seconds.

With the above points outlined, I feel this will be a very unique open-source firewall.

## Communication Between The Server And Bifrost's Back-End
I'd like to make the main application that runs on the servers themselves in C because this will allow us to read/write from the BPF maps easier. In addition, I'd like to use an HTTP/HTTPS library for talking to Bifrost's back-end/API. I found a C library named [libhttp](https://github.com/lammertb/libhttp) and I think it'd be beneficial using this to send regular HTTP/HTTPS requests or setting up a web socket.

I believe using a web socket would be better because to my understanding, it'd keep the connection alive and in return, will result in (on a very micro level) less bandwidth/resource consumption since we aren't establishing a three-way TCP handshake each HTTP/HTTPS request we send out.

Since the three-way TCP handshake cannot be spoofed, I do believe just having an IP/port validation on the Bifrost back-end will be good enough for validation from the server to Bifrost's back-end. However, we'll also look into implementing more security checks if we can including a token, certificates, and more.

## The XDP Program
The XDP program will contain two BPF maps that contains all the firewall rules for passing and dropping traffic besides payload matching (since as explained above, I wasn't able to find a way to get dynamic payload matching to work in XDP [here](https://github.com/gamemann/XDP-Dynamic-Payload-Matching)). The passing BPF map will be checked first to ensure attackers don't spoof as a service we need and get the source IP rate-limited which would cause service interruption. From here, the blocking BPF map and rate limits will be checked. Afterwards, the BPF map containing all drop rules from the firewall will be inspected.

This XDP program will also include a rate-limit feature so we can block source IPs or other characteristics based off of rate limits (per second, per minute, and more).

Two other BPF maps will need to be created. One BPF map will use an unsigned 32-bit integer as the source IP lookup key and an unsigned 64-bit integer as the value to indicate when the source IP should be unblocked. This will operate as the source IP blacklist map. We may also add additional blacklist maps for specific characteristics so we don't have to inspect deeper into the packet each time (which allows us to drop packets even faster).

The last BPF map will have an unsigned 32-bit integer as the source IP again and then how many packets it has sent in the last second (PPS) which will be used for rate-limiting. We can also do some math and make it so we can block based off of packets per minute and a certain amount of time as well. I'm also going to add rate-limiting for the amount of bytes per second the source IP sends (or in minutes or a custom timeframe as well).

Other than dropping specific traffic within our upstreams by BGP Flowspec policies, we would want to drop as much malicious traffic as possible within the XDP program since that's faster than anything else we utilize within Bifrost at this moment.

## BGP Flowspec
BGP [Flowspec](https://blog.marquis.co/what-is-bgp-flowspec/) support is only a feature certain hosting providers such as GSK will allow us (GFL) to benefit from. I'm hoping to make it so the BGP Flowspec part of the C program will parse all firewall rules and push BGP Flowspec policies for any firewall rules that have the "Push BGP Flowspec Policy" option enabled on.

BGP Flowspec will allow us to push policies to our upstreams and have them filter certain traffic instead. This is truthfully the fastest way to drop malicious traffic and since the upstreams are blocking them, this would result in no resource consumption from the attack on our POP servers.

While BGP Flowspec is a very powerful tool, it can also do a lot of harm if used incorrectly. For example, we need to ensure we aren't pushing any policies that could take down our BGP sessions (e.g. such as blocking TCP/179). There have been many outages in the past (most recently, an [outage](https://blog.cloudflare.com/analysis-of-todays-centurylink-level-3-outage/) that occurred with CenturyLink) caused by BGP Flowspec policies that have impacted a good part of the Internet. Thankfully, we won't have to worry about taking down a good percentage of the Internet as a whole, but we could take down our entire Anycast network if used incorrectly.

I'll be making sure we take into account the whitelisting aspect of Bifrost when pushing out BGP Flowspec policies.

## Firewall Rules
Bifrost will include a module that supports setting up firewall rules. Firewall rules can be applied globally or on a per-server basis. Each firewall rule will also include an option to push via BGP Flowspec if available (most characteristics should work besides payload since that isn't supported within BGP Flowspec at the moment).

The following options will be supported with firewall rules to match on. Please note that all of these besides "Enabled" and "Action" are optional.

* Enabled - Whether to enable the rule or not.
* Action - Whether to pass or drop the packet matching.
* Source IP addresses (or CIDR ranges).
* Destination IP addresses (or CIDR ranges).
* Source ASN (this will support ASN prefix lookup so you can match on all prefixes from a specific ASN).
* Minimum/Maximum IHL - IP header's min/max IHL to determine the IP header's length (IHL * 4).
* TOS - Type of Service to match on within IP header.
* Minimum/Maximum ID - IP header's min/max ID to match on.
* Minimum/Maximum fragment offset - IP header's min/max fragment offset.
* Minimum/Maximum TTL (Time To Live).
* Minimum/Maximum packet length.
* IP protocol - E.g. TCP, UDP, and ICMP.
* IP flags - IP header's flags as a 3-bit integer).
* Additional IP header options.
* UDP source ports.
* UDP destination ports.
* TCP source ports.
* TCP destination ports.
* TCP Flags (SYN, ACK, PSH, FIN, RST, and URG).
* Minimum/Maximum TCP Sequence and Acknowledgement numbers.
* Minimum/Maximum TCP Window size.
* Additional TCP header options.
* ICMP code/type.
* Minimum/Maximum PPS (Packets Per Second).
* Minimum/Maximum BPS (Bytes Per Second).
* Minimum/Maximum payload length.
* Payload exact/partial match (a string in hexadecimal indicating bytes to match on).
* Possibly more.

For more in-depth packet inspection, creation of modules using TC/BPF or netfilter are encouraged. Payload inspection will occur inside a Linux kernel module utilizing the netfilter networking hook (examples I made may be found [here](https://github.com/gamemann/Test-Kernel-Modules)). The main thing I need to figure out is how to interact with the XDP BPF maps from within the Linux kernel module itself.

Initially, I had planned to add these firewall rules to NFTables or IPTables and using IPTable's [NFQueue](https://home.regit.org/netfilter-en/using-nfqueue-and-libnetfilter_queue/) to add the source IP, etc. to the XDP BPF block map. However, this wouldn't support zero-copy. Therefore, there's a high chance an attack would consume all of the server's resources just from these actions. Therefore, I decided we should do everything we can within the XDP program or inside the kernel.

## Cached Packets
Bifrost will have the ability to cache specific packets. What Bifrost will do is have a process in the background within the back-end that sends the request for a certain packet type on a certain interval and store the response somewhere for the server's themselves to retrieve. From here, when an incoming packet matches the request, the server will respond with the cached response instead and drop the initial incoming packet. This will help mitigate damage from layer-7 attacks.

These packet types will be defined in the back-end portal and either on exact/partial matches within the packet/payload using a string using hexadecimal to represent each byte.

## IP/Port Forwarding
For services that cannot be Anycasted or for load-balancing purposes, Bifrost will be able to forward traffic on a destination IP/port match via NFTables. We're going to call these rules "connectors" and each connector will start off with a destination IP to match on and another IP to forward the traffic to. From here, you'll be able to add port mappings inside this connector to map the bind port and port we're forwarding the traffic to. You'll also be able to use wildcards/port ranges in these mappings if you want to forward all traffic on an IP level or use port ranges.

Additionally, we'll also have some modules to handle other traffic such as the FOU Wrapper/Unwrapper module which can be found [here](https://github.com/gamemann/Compressor-V2-FOU-Wrap-Unwrapper) (it needs some changes, but should give an overview of what we're planning to achieve and will be documented later on).

## Game Server Handshake Validating
We're going to be making TC BPF programs (both ingress and egress) to validate handshake sequences our clients make to our game servers. I won't go into much detail here since I already did on this thread [here](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Anycast_Network/Notes/Anycast-Filtering-Notes-DDoS-Attacks-Filtering-And-More.MD) for those interested. However, this will help us a lot with preventing malicious traffic from being forwarded to our endpoint game server machines.

Additionally, I also plan on pushing any validated source IPs to a global list provided by the back-end that the TC BPF maps will be checking for (to ensure the client is on the handshake-validated map). This is so if somebody's route starts going through another POP server while connected to our game servers, they won't timeout and need to reconnect to reinitiate the handshake sequence again (this is a problem at the moment with our temporary filters for Compressor).

## Conclusion
This is a general overview (both high and low level) of our Bifrost project. I'm pretty excited for this and believe this is really close to what we're rolling with. There may be some things that need to altering depending on how we approach certain problems. However, this is the closest we've gotten to finalizing our game plan.

As stated at the beginning of this post, once everything is finalized, we will be storing our game plan inside either a separate repository under the Bifrost organization [here](https://github.com/BifrostTeam) or under "Wiki" within a repository (all plans will be written in Markdown).

If you have any questions, suggestions, or want to be involved with the creation of Bifrost, please reach out to [Dreae](https://github.com/dreae) or I.

Thank you for your time!

**[Original Source](https://gflclan.com/forums/topic/66274-bifrosts-finalized-plan/)**