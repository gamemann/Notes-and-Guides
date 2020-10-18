# Another Update On Performance Issues With NYC Servers
**Created on March 31st, 2020**

Hey everyone,

I just wanted to provide an update on resolving performance issues with our NYC servers. You can read more about the issue itself at the end of [this](https://gflclan.com/forums/topic/53600-tech-update-new-web-machine-what-im-working-on-and-more-3-16-20/) post.

Recently, I've been trying to figure out how to get the game server machines to send traffic back to the clients directly instead of back through the IPIP tunnel (and closest POP server). This would result in less load on our POP servers which appears to be the main issue right now. With that said, this would result in less latency and overall better performance/consistency. Plus less bandwidth overage fees that we're paying at the moment :P

Unfortunately, SRCDS doesn't appear to support binding to two separate interfaces (one for sending and one for receiving). Therefore, the easier solution of just adding a veth pair inside of the network namespaces the IPIP endpoint tunnel and game servers reside in, making the veth peer the default route (along with next-hop IP to the bridge on the main host), and adding an SNAT rule to the POSTROUTING IPTables chain under the NAT table is sadly not possible.

Instead, I am trying to create a program that modifies outgoing IPIP packets. Since the IPIP tunnel sends traffic back out the default interface on the server, I can just attach to that interface. Originally, I was trying to make a program that uses **AF_PACKET** sockets. However, the initial approach was somewhat incorrect since I wasn't binding to the default interface, but instead binding to the IPIP tunnel itself inside the namespace. For some reason the program wasn't able to send packets back out through the network namespace. I made a Stack Overflow thread [here](https://stackoverflow.com/questions/60904961/linux-c-program-wont-send-packets-through-network-namespace-using-af-packet-soc), but didn't receive any response (probably because not many people do this and I'm sure I was missing some sort of IPTables rule, lol). The other problem with this approach is using standard **AF_PACKET** sockets results in the kernel copying each packet to the user space (not zero-copy). This is A LOT slower. However, I just wanted to see if my theory would work.

A couple hours after trying to create the program above, I found out about the default interface sending IPIP outgoing packets and I also thought of a better idea. What if I made an XDP program that modified these outgoing packets?! XDP would be a lot faster than standard **AF_PACKET** sockets and does its work inside the kernel. Last Sunday I spent most of the day making this program. However, after debugging and research, I found out that XDP does NOT support the TX path at the moment which is needed for outgoing packets. This was a huge upset for me since I had the program pretty much made and good to go. I even released the source code on [GitHub](https://github.com/gamemann/IPIP-Direct-XDP-TX-Not-Supported-) for when XDP does support the TX path (read below). Some changes will probably need to be made once support is added, but there shouldn't be many.

After this upset, I decided to make a thread on the XDP mailing list [here](https://www.spinics.net/lists/xdp-newbies/msg01628.html). I just wanted to know what others thought and if **AF_XDP** sockets (XDP userspace/ex-AF_PACKETv4 sockets) would support modifying outgoing packets. Unfortunately, but as expected, **AF_XDP** sockets only support receiving traffic from the XDP program via the redirect function. However, XDP TX path support will be added in the future and it's currently being worked on by David Ahern [here](https://github.com/dsahern/linux/commits/xdp/egress-rfc5-06). I really appreciate all the work and help these people put into this project! Anyways, there is no ETA on when XDP TX path/egress support will be added.

Therefore, I continued asking for more help [here](https://www.spinics.net/lists/xdp-newbies/msg01631.html) and clarified our situation. I received the following response this morning from Toke Høiland-Jørgensen:

> I think you could do this with the TC hook? You can install BPF programs there that have then same ability to modify the program as XDP does. And since the packets are coming from an application, you don't gain any speedup from XDP anyway (since the kernel has already built its packet data structures).<br><br>
> -Toke

While I've heard of the TC Hook before, I haven't actually made any programs that uses it. This will be my first attempt and since there is no ETA on the XDP TX path support, I'm going for it. I found this useful guide on making TC programs (along with XDP, but I already know how to make those). I'll have to make the BPF program and load it onto the interface. This shouldn't be too hard once I learn how to load TC BPF programs onto the interface using a loader C program along with learning how SKBs work (something I've seen, but wanted to learn for quite a while now). I will read this article after work today or during my lunch break. I have a skeleton of the TC BPF program made now:

```C
#include <linux/bpf.h>
#include <linux/pkt_cls.h>
#include <iproute2/include/bpf_elf.h>
#include <linux/if_ether.h>
#include <linux/udp.h>
#include <linux/tcp.h>
#include <linux/icmp.h>
#include <linux/ip.h>
#include <linux/in.h>
#include <inttypes.h>
#include <bpf/bpf_helpers.h>

struct bpf_map_def SEC("maps") interface_map =
{
    .type = BPF_MAP_TYPE_ARRAY,
    .key_size = sizeof(uint32_t),
    .value_size = sizeof(uint32_t),
    .max_entries = 1
};

SEC("egress")
int tc_egress(struct __sk_buff *skb)
{

}
```

In conclusion, I understand this issue is impacting our NYC servers quite a bit (especially Rust Modded). I hope you can understand I am trying the best I can to get this issue resolved in a timely manner. Unfortunately, the issue itself is very complex. I might try something else in the meantime which is booting up more POP servers in NYC specifically. Technically, each game server should route to different POP servers in that location that is handled by a load-balancer (round robin). This may resolve the performance issues as well, but we'll have to see.

Thank you for reading.

**[Original Source](https://gflclan.com/forums/topic/54275-another-update-on-performance-issues-with-nyc-servers/?do=findComment&comment=248573)**

## Update
**Updated on April 2nd, 2020**

I am happy to announce that I've created a project that achieves what I've wanted! This project uses the TC egress hook/filter and searches for outgoing IPIP packets. Once an outgoing IPIP packet is found, it strips the outer IP header, changes the inner IP header's source address to the forwarding server (game server IPs in our case), recalculates all the header's checksums, and sends the packet to the upper layers. This results in the packet being sent back directly to the client instead of going back through the forwarding server (in our case, the game server's closest POP). I've been working very hard on this the last couple days and while I've ran into multiple headaches, I'm very happy I was able to get everything pretty much working.

I still need to do more testing, specifically with TCP traffic, but so far everything is good! I tested this on my local environment and everything worked. I was able to play on the game server fine and there were no performance issues.

The TC hook is pretty fast as well. Therefore, this program should achieve high performance.

One thing I want to note is a newer kernel version is required to run this program on our game server machines. Therefore, once I get done testing everything and know for a fact that it'll run well, I will need to schedule maintenance for each game server machine and upgrade the kernel.

## Advantages To Running This Program
Here are the benefits to running this program on our game server machines:

* Less load on our POP servers since all the game server traffic won't be routed to the server's closest POP server on our network.
* Less latency since packets won't be routed back through the closest POP. Instead, they'll be directly sent back to the client. Our game servers have good routing. So I'm not concerned there.
* Eliminates a single-point-of-failure.
* Due to less load on our POP servers, performance should be increased a ton and the lag issues we've been experiencing in NYC should come to an end.
* Since all game server traffic won't be going through the closest POP, we won't need a beefy POP server in each locations our game servers reside in.
* Less overage fees (this is a killer for us right now and costing us $300 - $400/m).

There aren't really any cons if this program works properly and TC does a good job at modifying the packets.

I've released the project here on [GitHub](https://github.com/gamemann/IPIPDirect-TC).

Once I have another update, I will let you all know.

Thank you!

**[Original Source](https://gflclan.com/forums/topic/54275-another-update-on-performance-issues-with-nyc-servers/?do=findComment&comment=249025)**