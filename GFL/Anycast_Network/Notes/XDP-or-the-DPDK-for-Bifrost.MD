# XDP Or The DPDK For Bifrost?
**Created on August 18th, 2020**

Hey everyone,

Before continuing, I just want to state information in this thread is based off of what I've read online **so far**. This means information in this thread is subject to change over time. I'm going to be continuing to read more about the differences between XDP and the DPDK along with occasionally posting updates with new information I find, etc.

With that said, if you see incorrect information in this thread, please feel free to reply with corrections. I am trying to understand and learn as much as I can regarding XDP and the DPDK because I want to use both for personal and GFL projects in the future.

## Quick Introduction
[Dreae](https://github.com/dreae) and I (Christian) are going to be creating custom packet processing/filtering software that runs on POPs within our Anycast network. This software will be responsible for forwarding all legitimate game server traffic to the appropriate game server machines and dropping any other traffic (including malicious). Before continuing, I'd recommend reading [this](https://gflclan.com/forums/topic/59681-anycast-expansion-plansapproaches/) and [this](https://gflclan.com/forums/topic/57006-anycast-filtering-notes-ddos-attacks-plans-to-mitigate-and-more/) thread to learn more about our plans with Bifrost along with the Anycast infrastructure itself.

In doing so, a very important part of the software is what we're going to be using to drop traffic that we don't see as legitimate/needed. Initially, our plan was to use a combination of XDP, TC, and NFTables. We'd use XDP to drop any traffic we don't need (or malicious), TC to handle outbound game server traffic (wrapping in FOU-IPIP) along with handshake processes with our game servers, and NFTables to forward legitimate traffic to the game server machines. However, I found that this setup might be a bit messy since we'd possibly have packets going through the XDP program down to the TC filters. With that said, we'd need to order all of our TC BPF programs under the same qdisc which will take some work.

I also want to note that the DPDK and XDP are both great solutions for (D)DoS mitigation in my opinion. I'm just trying to find out what would be best for Bifrost. I will now make a summary of the DPDK vs XDP along diving into the specifics.

## XDP vs The DPDK (Summary)
[XDP](https://www.iovisor.org/technology/xdp) (E**x**press **D**ata **P**ath) is a low-level hook within the Linux kernel. The hook is implemented into the NIC driver assuming the NIC driver supports XDP-native. If that specific NIC driver doesn't support XDP-native, the program will need to use XDP-generic which is a hook later on in the networking path (after SKB allocation). While the XDP-generic hook is a lot slower than the XDP-native hook, I do believe it has similar performance to NFTables/IPTables. At the moment, we use [Compressor](https://github.com/Dreae/compressor) as our packet processing software which was made by Dreae and also uses XDP. Unfortunately, loading Compressor with XDP-native breaks the AF_XDP program either due to outdated AF_XDP code or a bug within the virtio_net NIC driver. Either way, I figured we might as well work on Bifrost instead of rewriting the AF_XDP code (which is complicated and AF_XDP is just buggy based off of my experience).

The [DPDK](https://www.dpdk.org/) (Data Plane Development Kit) is a collection of libraries and drivers which can be used for fast packet processing. This was made by Intel. The DPDK is considered a kernel-bypass framework because it creates a fast-path (zero-copy) from the NIC to the DPDK application running within the user space. With that said, the DPDK includes a PMD (Poll Mode Driver) that constantly polls the NIC for packets. What this means is instead of the NIC raising an interrupt to the CPU when a frame/packet is received, it constantly polls the NIC. Though, note this requires CPUs to be dedicated to the DPDK application and they will be utilized at 100% CPU at all times. This results in less latency when processing packets when using the DPDK.

I will now write bullet points of what I've found for XDP vs the DPDK (the pros of each).

### XDP Pros
* Less things to learn/keep track of.

* Allows us to use kernel functionality.

* Allows the option of using "busy" polling or interrupt driven networking.

* Removes the need of dedicating specific cores to the program.

* Removes the need of installing third-party dependencies which is needed for the DPDK.

* Wouldn't need to rewrite the entire network stack including TCP/IP, ARP, etc. (assuming we don't use [KNI](https://doc.dpdk.org/guides/prog_guide/kernel_nic_interface.html) with the DPDK or an already-made library on top of the DPDK, read below).

* Dreae and I are already familiar with writing XDP programs.


### The DPDK Pros
* Faster than XDP (I believe by around 5 - 10%). [Here are](http://vger.kernel.org/lpc_net2018_talks/presentation-lpc2018-xdp-future.pdf) benchmarks I found from 2018. [Here's](https://www.net.in.tum.de/fileadmin/bibtex/publications/papers/ITC30-Packet-Filtering-eBPF-XDP.pdf) another PDF I was reading that includes information regarding performance with the DPDK vs XDP.

* Since the DPDK application runs within the user space, we should be able to implement all the functionality we need into this single DPDK application instead of having it spread out between XDP, TC, and NFTables.

* Wouldn't have to use eBPF maps to transfer data from the user space to the program processing packets.

Note that the three pros for the DPDK would be very beneficial in our situation in my opinion. However, there are also a lot of drawbacks with the DPDK depending on how we approach it. I will go into detail below.

## XDP
As noted above, XDP was our initial solution to dropping unneeded/malicious packets. I'm not entirely sure how we would go about the program itself. We could either have the XDP program running at all times and search the source IP on an eBPF map and if the source IP is found within this map, drop the packet (this map would act as a blacklist map). Otherwise, we could have something inspect packets later on in the networking stack and if malicious traffic is detected, spawn an XDP program that would drop said traffic.

XDP is a great solution, but the DPDK is still technically faster when processing/dropping packets. With that said, our setup will be a bit more messy with XDP since our plan is to only have XDP drop the packets themselves (for now). Therefore, we'll need to setup TC BPF programs as well and if we were to take a whitelist approach, we'd need to figure out how to pass initial connections through the XDP program without having XDP drop them by default (a whitelist approach was our initial plan as well).

Additionally, we'll need to have NICs where the driver supports XDP-native since using the XDP-native hook would be the best for performance. This shouldn't be an issue, but it's worth noting. Thankfully, I found a new presentation recently that goes over the process of implementing XDP-native support into a NIC driver [here](https://people.redhat.com/lbiancon/conference/NetDevConf2020-0x14/add-xdp-on-driver.html). This'll be useful if we ever need to do anything like that in the future (and it's something I'm interested in as well).

## The DPDK
I still have a lot of exploring to do with the DPDK, but I will put down the information I know as of right now. The DPDK is a pretty big framework to learn. For example, [here](https://doc.dpdk.org/guides/prog_guide/) is the programmer's documentation that is 65 sections alone (some sections being VERY long such as sections 3 and 12). With that said, if we were to build our own DPDK application from scratch, we would need to implement our own network stack for handling TCP/IP connections along with ARP requests. Maintaining our own TCP/IP network stack for a firewall isn't ideal and that'd be very time-consuming to create and manage over time. That said, maintaining the performance of the TCP/IP stack is an entire different story. This is something I'm interested in doing in general, but not for a single firewall application and I don't believe I'd learn all of this any time soon.

Now, there are two possible approaches we could take that'd result in us not having to implement our own network stack to my understanding.

Firstly, we could use DPDK's [KNI](https://doc.dpdk.org/guides/prog_guide/kernel_nic_interface.html) library (Kernel NIC Interface). I haven't read the specifics of this yet, but to my understanding it'd act as a TAP device within Linux. This would allow us to send packets we don't need to process down the default network stack on Linux. The only con I've heard of with doing this is performance degrades. For example, take a look at the replies from [this](https://stackoverflow.com/questions/26734955/is-it-possible-to-write-a-web-server-using-intel-dpdk) and [this](https://stackoverflow.com/questions/19313834/how-can-i-use-dpdk-to-write-a-dns-server) Stack Overflow thread. People have suggested using the KNI library. However, also mentioned that it does come with a performance impact. Now, what's not made clear within these replies is whether this performance degrade occurs when sending packets down the default Linux network stack or if it includes packets that we want to process with the DPDK. If this impacts only packets sent down to the network stack (AKA packets we mostly don't care about such as traffic to the POP itself, BGP, and so on), this wouldn't be a big deal because having a slight performance degrade with them won't matter much. However, if the performance is impacted on packets we want to drop/process via the DPDK application while using the KNI library, there really isn't a point in using the DPDK to begin with since our main goal is to achieve the maximum performance possible.

Secondly, we could use an already-written TCP/IP stack that is built on-top of the DPDK. I've seen some people suggest this, but they also mentioned it most likely comes with performance degrades depending on which library you go with. This is definitely something I'll continue looking into, though.

I still need to do a lot more research regarding this. However, if we can use KNI or an existing TCP/IP stack (along with ARP) with no performance degrades on the packets we want to process/drop within the DPDK program, I still feel the DPDK is something we should definitely consider with Bifrost.

## Summary
Overall, the DPDK and XDP are both great solutions to what we want to accomplish with the network (being able to mitigate large (D)DoS attacks). However, both have their own pros and cons. I still need to do a lot more research on the DPDK before making a decision and I will continue to post updates here as I find new information. I've been reading a lot of code from open-source DPDK applications on GitHub (e.g. searching for [this](https://github.com/topics/dpdk) topic) and from [here](https://github.com/DPDK/dpdk/tree/main/examples) as well. This has been helping greatly with understanding the DPDK 

Thank you for reading!

**[Original Source](https://gflclan.com/forums/topic/61430-xdp-or-the-dpdk-for-bifrost/?tab=comments#comment-273385)**

## Update
**Updated on August 23rd, 2020**
I just wanted to provide an update on this. I was told that we should get the same performance using the DPDK [KNI](https://doc.dpdk.org/guides/prog_guide/kernel_nic_interface.html) library compared to using the DPDK without it when processing packets with the DPDK application (e.g. packets we truly care about). However, packets that we decide to send down the network stack (e.g. packets we don't care to filter/forward) will suffer a slight performance impact which makes sense because we're intercepting it with the DPDK application and sending it down its way through the network stack.

This was more so what I was hoping for if we used the DPDK KNI library. Truthfully, traffic that we'll be sending down the network stack will more than likely be traffic to the POP itself (SSH, package updates, BGP, and so on). Having a slight performance degrade on this specific traffic wouldn't mean much since we don't truly care to filter that as of right now. We will still be dropping any unneeded/malicious traffic using the DPDK application itself.

I do plan on making benchmarks against XDP and DPDK (with and without the KNI library) in the future. Hopefully this'll confirm everything :)

Honestly, I think we're going to use XDP to drop packets still. I say this because for our POP servers, XDP will be able to drop 10 gbps with ease which will be the max NIC speed anyways that we go with. It'd make sense to write a DPDK application if we had to drop 10+ gbps and if the network equipment supported ASIC (hardware integrated into the NIC that allows for faster packet processing).

**[Original Source](https://gflclan.com/forums/topic/61430-xdp-or-the-dpdk-for-bifrost/?tab=comments#comment-274399)**

## Another Update
**Updated on August 25th, 2020**
I just wanted to provide another update. I still need to talk to Dreae about this to make sure he agrees.

As stated in my last post, I believe it's best to use XDP to drop malicious traffic instead of the DPDK for Bifrost. The reason I believe this is because using the DPDK would be a lot more complicated and honestly, we don't need the 5 - 10% performance increase if we're planning to use BGP Flowspec to push policies to our upstreams to have them mitigate certain types of attacks. XDP is still a great solution for (D)DoS mitigation regardless and since our POPs will have either one or ten gbps NIC speeds max, there is no need to use the DPDK in this case over XDP unless if we already had programming experience with the DPDK, of course. I still plan on learning the DPDK regardless because I'd like to use it for some personal projects I plan to work on in the future and perhaps I could make a Bifrost module that would utilize the DPDK.

Now, I did rethink the design of the packet flow with Bifrost because I wasn't sure how we'd take a whitelist approach if we used XDP to block unneeded/malicious traffic.

I believe it's best if we use XDP **only** to drop malicious traffic that we detect. Therefore, we won't be dropping unneeded traffic with it. Instead, unneeded traffic will go through the standard Linux network stack and either be discarded or sent to the appropriate service. This shouldn't impact performance by much at all since unneeded traffic won't have a high volume.

We will include a whitelisting option for our forwarding module. What this means is we'll have TC BPF (both ingress and egress) that will decide whether packets should be forwarded via NFTables/IPTables by marking the packet. So incoming traffic to our Anycast IPs will go through these TC programs and if the packet matches the initial handshake or the source IP was already validated, it will mark the packet and send it on its way down the network stack. NFTables and IPTables will only forward these packets if they have, for example, a marking of **0x7**. Now obviously, you won't need to take a whitelisting approach if you don't want to. We're planning to have everything configurable.

Now, the packet filtering aspect of Bifrost will be a module on its own. I'm still unsure what we want to use to inspect incoming packets. But I believe we could just use NFTables or IPTables for this as long as it includes inspecting each and every part of the packet and being able to use an API of some sort in C so we can match it against our filtering rules. With that said, any packets that match filters that have a block time set should be added to the XDP blacklist map for a period of time so they can be dropped by the XDP program. The only thing we need to make sure of is we're not copying the packet to the user space for inspection because this would obviously cause a big performance degrade. However, NFTables and IPTables do zero-copy to my understanding since it comes before the user space (they utilize net filter). One thing to note with this module is it is stateless meaning we're just matching each incoming packet against a set of filters and aren't keeping track of connections.

With the above said, I ALSO want to make a module in the future that is more stateful for filtering packets. This would be a separate module because we won't have individual filters. Instead, we'll have this module try to detect patterns in connections/packets (UDP, TCP, and any other protocol) and then you can set what to do with the packets (either block them via XDP with a block time, push BGP Flowspec policies, drop the packet individually, or just alert the admins). We would probably utilize TC for packet inspection on this since we'd like for it to be as fast as possible. However, I'll need to see if BPF would allow me to do payload matching with TC (I know it doesn't like it with XDP, but I've noticed TC programs being a bit less strict with the BPF compiler for some reason). Otherwise, I suppose we could use IPTables or NFTables, but I don't think that'd give us full control over the packet inspection.

Our POPs will basically be responsible for absorbing attacks that either have changing characteristics with each packet (e.g. random source IP, protocol, packet length, TTL, and so on, basically packets we can't find any patterns for) and packets we drop via a payload match since BGP Flowspec doesn't support payload matching right now (it DOES appear to be in-progress though according to [here](https://tools.ietf.org/html/draft-khare-idr-bgp-flowspec-payload-match-05)). Any other attacks should be detected and a BGP Flowspec policy should be pushed to drop the traffic at the upstream so the POP isn't responsible for that. I do believe our future POPs should have a 10 gbps NIC and link just so we can support dropping up to 10 gbps on each POP.

Anyways, other than that, the caching aspect of Bifrost will also be a module. When Bifrost is booted up on the servers, they will use a TCP encrypted connection to communicate with the centralized server to retrieve settings and other information such as filters, forwarding rules, etc. I think we should make our own encryption for this using [Libsodium](https://libsodium.org/) and [Erlang Crypto](https://erlang.org/doc/man/crypto.html) which Dreae and I already have experience with (Dreae a lot more than me, but I did make [this](https://tools.ietf.org/html/draft-khare-idr-bgp-flowspec-payload-match-05) haha).

Apart from that, the other neat thing with GSK (our primary POP hosting provider going forward) is they allow us to purchase hardware and put the hardware in-front of our servers to my understanding. Therefore, if we ever had the money and wanted to, we could purchase a router or switch that has NICs with ASIC built-in to process specific type of packets/packet flows via TCAM which is A LOT faster since it results in no load being put onto the CPU. GSK's upstream/network also has switches and routers that have ASIC built into the NIC. I know they support layer 3/4 filtering via ASIC which we'll be utilizing via BGP Flowspec policies. But I do not know if they have a router that has specific ASIC built-in to handle payload matching (there are some out there, but I'm not sure if GSK has any).

Things are looking good though and I'm confident this'll be a much cleaner system than what I initially had planned (which was honestly delaying a lot of the development on Bifrost since my previous plan was messy).

Thank you!

**[Original Source](https://gflclan.com/forums/topic/61430-xdp-or-the-dpdk-for-bifrost/?tab=comments#comment-275017)**