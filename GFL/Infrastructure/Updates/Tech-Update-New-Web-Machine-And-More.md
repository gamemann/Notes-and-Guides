# Tech Update - New Web Machine, What I'm Working On, and More
**Created on March 16th, 2020**

Hey everyone,

I just wanted to provide an update regarding the technical side of GFL. I just came back from my first legitimate break in GFL (that lasted over two weeks somehow). I just wanted to be transparent about these projects so people know what we're working on, etc.

## New Web Machine
Recently, Xy and I decided to purchase a new web machine. This machine is still from OVH, but it is slightly cheaper than our current machine. With that said, the new machine's processor has twice the amount of cores, a higher clock speed, and a newer architecture. The machine does come with less disk space, but that'll be manageable with the changes we're making. It has the same amount of RAM as well.

The one big change with moving to the new web machine is how we're allocating resources to each service. With the new machine, each service we host will be sharing resources (CPU, RAM, etc). This is a lot more efficient than what we are currently doing. This will result in better performance.

Xy has been doing most of the work on the new machine and has done a great job so far! We've already moved our GitLab [website](https://gitlab.gflclan.com/) to the new machine and it has been performing well. We will continue to slowly move services over and we will give everybody a heads up when moving websites, etc. that'll need downtime.

We will start moving more important services once I review the new machine's configuration which should be done very soon.

## New Software That'll Run On Our Anycast POPs
This is definitely one of the bigger projects on my list and the one I'm investing the most time in. I am planning to write software that'll run on our Anycast POPs. This software will be responsible for forwarding traffic from our Anycast POPs to our game server machines. With that said, it'll also be responsible for absorbing (D)DoS attacks and more. I am trying my best to micro-optimize everything within the program as well so it's 'lightweight'. The program will be dropping malicious traffic via [XDP](https://en.wikipedia.org/wiki/Express_Data_Path) which is a lot faster than things like IPTables. [Here's](https://blog.cloudflare.com/how-to-drop-10-million-packets/) an article by CloudFlare that shows the differences between XDP and others. The nice thing about having everything performed in the user space is that we can check the load of the program via `top` or `htop`. Our current software does everything in XDP which is nice, but since it's done in the kernel, we cannot see the actual load of the software. We can use the 'perf' command, but reading that can be a pain at times.

Recently, I've decided to start learning complex network programming in C. So far, I've made [programs](https://gflclan.com/forums/topic/53370-c-udp-sender/) that act as simple UDP floods (for testing purposes against my receiving/forwarding software) along with a program that forwards traffic to a machine via IPIP packets. Today, I just got done completing the test program that forwards traffic via IPIP packets. The program is completely multi-threaded and handles the traffic (both sending and receiving) using raw **AF_PACKET** Linux sockets. I plan to release the code on [GitHub](https://github.com/gamemann) and I'll be posting it in the Projects section [here](https://gflclan.com/forums/forum/950-projects/). In the meantime, if you're interested, you can check out [this](https://gflclan.com/forums/topic/53372-c-unofficial-code-to-capture-all-packets-and-forward-via-ipip-help-needed/) thread that provides a lot of code I've used in the program I plan to release.

Here are the next things I need to learn:

* How to collect packet stats per client and report it to a database elsewhere while achieving maximum performance (e.g. the amount of data per second a client is sending/receiving, the amount of packets they're sending/receiving,  protocol specific stats, and more).
* Creating filtering rules and matching the filtering rules with the entire frame/packet (this won't be too difficult).
* Dropping traffic that is matched by a 'block' filter rule via XDP. I'll have to use an eBPF map to communicate from the user space program to the XDP program.
* How to cache specific packets on the POP level every **x** seconds which will be based off of caching rules. I will more than likely be using Redis to store the cached packets.
* Building a website and backbone that all POP servers will connect to. This will allow me to perform global functions on selected or all POP servers (e.g. capturing all packets on all POPs and more). This will also store all the forwarding information (e.g. the bind address, the forwarding address, NAT address, and more). I plan to write the website using Go Lang.
* Analyzing traffic in the backbone and if an attack is detected, spin up additional temporary POP servers in locations where existing POP servers are overloaded from the attack. POP servers should be load-balanced in each location. After the attack is finished, it will shut down and destroy the temporary POP servers after an hour or so.

This is a very big project and as I said before, this is consuming most of my time in GFL at the moment. I am not sure whether I will be releasing this project publicly or not. If I make it portable, I might, which is what I'm trying to do. I was thinking about starting a new company that would use software similar to this once I get everything created, etc.

I have no ETA on when this project will be completed. I've made great progress on it so far and the forwarding aspect is pretty much done. However, there are still so many things I need to learn and implement. I am also trying to take it slow so I don't burn out, but I do really enjoy learning everything I have been. To be honest, this is the type of project that a large cooperation assigns a team of high-paying engineers to make. Therefore, it's going to take me some time since I'm the only one working on it at the moment.

## The Anycast Infrastructure Itself>
When I get closer to completing the new software that'll run on our Anycast POPs, I also need to continue to expand our Anycast network. Since we've launched the Anycast network, I've probably contacted 40+ hosting providers to see if they would be fit for us (I've probably exchanged over 100 emails total, lol).

I plan to do this again, but with the intentions of finding 3 - 5 solid hosting providers for our Anycast network. We will be going with quantity over quality in this case so we aren't relying on built-in (D)DoS protection by our hosting provider (although, it'd be nice if they do provide it). The goal is to have around 60+ global POP servers worldwide (multiple POP servers per location assuming load-balancing is enabled with the hosting provider).

Unfortunately, we have a lot of strict requirements that not many hosting providers offer including:

* BGP sessions (this will allow us to announce our IPv4 block under our ASN with the hosting provider).
* XDP-native [support](https://github.com/iovisor/bcc/blob/master/docs/kernel-versions.md#xdp) in the physical NIC driver. This will allow XDP to truly drop the malicious traffic in the kernel.
* VPSs that cost $10 or less. The hardware doesn't have to be powerful. As I stated before, we've chosen to go with quantity over quality (both have their pros and cons, but I believe the pros outweigh the cons with the quantity approach).

This alone takes a lot of time because of the communication with the hosting providers via phone calls and emails. However, this is something we need to do.

## Issues With NYC Machines And Anycast POP
I've addressed this issue in previous threads, but I just wanted to provide an update on it. I am fully aware of some of our servers experiencing terrible performance in our New York City location. The biggest servers this is happening to are our Rust Modded servers. Unfortunately, the issue is very complicated and debugging the issue is a pain. Basically, when the game servers send traffic back to clients, it has to send the traffic to the closest POP server and the POP server will send it back to the client. For some reason, packets are being dropped with connections from the game server machine to the closest Anycast POP at higher player counts on the servers. I tried upgrading the POP to four cores and that didn't resolve the issue. Our Dallas POP has only two cores and is handling all the game servers over there without any issues. I believe this issue might be something on the game server machine itself.

The first thing I tried doing was finding a new hosting provider and ordering a dedicated server to act as the POP server in NYC. While this was an improvement, the server still has bad performance at high player counts. The dedicated server had a dedicated and strong CPU. However, I think the issue was with the NIC driver not supporting XDP-native. I made a long thread going into details [here](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Anycast_Network/Updates/Another-Update-On-NYC-POP.md) regarding this issue.

I've exchanged emails with the XDP [maintainer](https://github.com/netoptimizer) as well which was neat and it's awesome to see he is willing to help others out! However, I haven't debugged the issue further yet. Instead, I've been trying to work on another solution that would result in not needing the game server machines to route to the closest Anycast POP. This not only would result in better performance, but it would eliminate the need of a stronger POP server to handle traffic from the game server machines. Therefore, our Anycast network wouldn't cost as much and so on.

What I am trying to do is basically make it so the game server machine sends traffic back to the client directly, but spoofs the source address as the bind/Anycast address. Technically, this should work by setting the default device to the interface that normal traffic route out of on the game server machine (instead of the IPIP tunnel). We then would need to add an IPTables rule to source/spoof the traffic out as the bind/Anycast address. We'd also need to ensure our hosting provider doesn't have blocks in-place for spoofing out as our Anycast network (I already made sure our hosting providers made exceptions for this).

Unfortunately, this isn't working. I am suspecting it is because outbound traffic goes through the game server's main interface, but it is also expecting the response on that main interface instead of the IPIP tunnel where the game server is bound to. I know traffic is definitely being sent back though, which is a good sign. I've been discussing this issue with networking experts (who are awesome, by the way) in this awesome Networking Discord [server](https://discord.me/networking) I'm a part of and hope to find a solution soon. Sadly, it isn't possible to configure the IPIP tunnel endpoint to send traffic back directly to the client without the outer IP header (for IPIP) attached to the frame.

Once I have an update on this, I will let everyone know! If you have any suggestions, please let me know!

## Conclusion
Overall, those are the main things I'm working on. I just wanted to make this post so everybody is aware of what's going on in the tech back-end of GFL.

If you have any questions, please let me know!

Thank you for reading.

**[Original Source](https://gflclan.com/forums/topic/53600-tech-update-new-web-machine-what-im-working-on-and-more-3-16-20/?do=findComment&comment=246322)**