# Another Big Bifrost Update (Really Close To Finalized Plan!)
**Created on January 10th, 2021**

Hey everyone,

I just wanted to provide another update on Bifrost. I believe we're getting really close to a finalized plan and you can read my last update [here](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Anycast_Network/Updates/Bifrost-Plans.md). In my last update, I understand I stated it was finalized, but it was also unapproved because I needed to completely think everything out. It turns out I highly overlooked something which I'll go into detail below. I also want to say thank you to @Dreae, @Synk, and @ghostthemarine for helping with the planning and support 🙂 (yes, I remember you were looking for other ways we could route traffic from the POP servers while in the Discord VC a couple weeks ago)

There are still a few challenges we're facing, but we're getting really close to a finalized plan and once we have a plan, developing everything shouldn't take very long, plus it should be pretty exciting. I've spent the last few weeks also working on my XDP Forwarding [program](https://github.com/gamemann/XDP-Forwarding) and also learning a lot more about AF_XDP which may be used as stated below. This has given me a ton of experience and I've been pretty happy with the progress I've been making. Just keep in mind that projects like these are very complex and also very low level. This is ultimately why everything is taking so long, especially since we want to do everything within the [XDP](https://www.iovisor.org/technology/xdp) hook using DRV or HW mode for the fastest packet processing possible so we can drop as many malicious packets as possible.

## Using XDP Primarily
Firstly, as stated in my last update, I'd really like to use XDP primary for everything. This includes the forwarding, firewall, and caching aspects of Bifrost. The reason I want to use XDP primary is because I feel it'd be too much of a pain using a combination of XDP, TC, Netfilter, and IPTables/NFTables. We would have to keep the orders of all the BPF programs in TC, keep track of the forwarding rules in NFTables/IPTables (in which, IPTables doesn't include an API and NFTable's does include one, but I'm not sure how easy it would be to use), and more. Additionally, using a combination of everything wouldn't be as fast as using XDP since we're be using user-space without a fast path for certain aspects. Plus, we were planning to use NFTables/IPTables for plain NAT which is no longer the case as you'll see below.

One big concern for developing everything within the XDP hook was the limitations within the BPF [verifier](https://elixir.bootlin.com/linux/latest/source/kernel/bpf/verifier.c). For example, the default max jump sequence is 8192 while the max insns is 1 million (basically the max size of the BPF program). Both of these were too small for my XDP Forwarding program if I wanted to use more than 21 source ports. However, thankfully, I was able to raise these limitations [here](https://github.com/gamemann/XDP-Forwarding/tree/master/patches). Since we're not using source port mapping when forwarding packets as you'll see below, I'm not sure if we'll need a custom kernel with these limitations raised. However, if we need to, we can easily do it and just run our POP servers on the custom kernel, thankfully.

Since we have smaller POPs as of right now, being able to forward traffic as fast as possible would be best in my opinion. Therefore, using XDP primarily will be ideal.

## Forwarding Legitimate Traffic
As of right now, we use the [IPIP](https://developers.redhat.com/blog/2019/05/17/an-introduction-to-linux-virtual-interfaces-tunnels/#ipip) protocol/Linux tunnel to forward game server traffic in [Compressor](https://github.com/Dreae/compressor). This adds an additional IP header to the packet which results in each packet we're forwarding itself being 20 bytes bigger than the initial packet. This is needed because we need to know which server the packet is directed towards when it enters the game server machine and having one IP header is just not enough, unfortunately, without running each game server on different ports that is.

Anyways, our initial plan with Bifrost was to use plain NAT either with IPTables/NFTables or within XDP that supported source port mapping. This would remove the need of encapsulating each packet which would also save 20 bytes and that would actually be really nice since each packet adds up. I made an XDP Forwarding [program](https://github.com/gamemann/XDP-Forwarding) that did exactly this and it actually performs a lot better than IPTables from the benchmarks I've performed (even using SKB mode!). I was using this program as practice for Bifrost. However, one thing I highly overlooked was the packets forwarded from the POP to the game servers would look like they're coming from the POP server and not the actual client. I thought @Dreae and I had a solution to that for some reason because I was aware of it, but it turns out there was no solution.

Therefore, it's honestly impossible to use plain NAT in this case unless if we wanted the IP of each client having the POP IP which is obviously not a good solution for many reasons. Therefore, we will still need to use a tunneling protocol. Another thing to note with using source port mapping is we'd only have 65535 max source ports to work with per bind address (but for performance, you'd probably want that lower unless if you have very beefy hardware). While my XDP Forwarding program does prioritize connections with the most packets per nanosecond, this could definitely be a weak point as well. For example, if somebody launches an attack targeting the same POP coming from 65535+ IPs consistently, it could overwhelm the entire source port map and cause everybody to timeout or have extreme packet loss. Again, we'd probably use something far less than 65535 as well, but at the same time, the chances of an attack targeting one POP with non-spoofed IPs consuming more PPS than regular clients and exhausting our source port map is pretty rare. Just something to note, though 😉

We could either use IPIP again or [FOU](https://developers.redhat.com/blog/2019/05/17/an-introduction-to-linux-virtual-interfaces-tunnels/#fou) which is implemented into the UDP protocol. I believe we're most likely going to go with FOU due to how much easier it is to handle. FOU is similar to IPIP, in fact, it has an outer IP header added with `IPPROTO_IPIP` set as the IP protocol, but it also has an outer UDP [header](https://en.wikipedia.org/wiki/User_Datagram_Protocol) which is 8 bytes in size. Therefore, each packet being encapsulated will have an additional 28 bytes in size instead of 20 bytes like IPIP. This isn't a big deal, though, especially since we plan to use XDP in DRV mode, so we'll be saving CPU cycles either way and things will still be fast. This will result in more bandwidth though, but I don't see that as a huge issue.

One big challenge we were facing with these protocols is they only support one remote IP on the tunnel endpoint. Therefore, while packets will come from the POP server set as the outer IP header, it will only forward back to one single host which as of right now, is the Anycast IP. This results in all outbound traffic from the game server sending back to the closest POP to the game server machine. We ran into this issue a while back and it resulted in us needing a very powerful POP with more CPU cores and bandwidth where the game server machine's outbound traffic routed to. It also created a big single-point-of-failure. Thankfully, I made [this](https://github.com/gamemann/IPIPDirect-TC) TC egress program that scans for all outbound IPIP traffic and sends it back as the Anycast IP directly. This helped a lot and resulted in game server traffic going from the game server machine to the client directly.

While this was nice, it wasn't ideal. This is because for one, the route the game server takes back to the client will likely not be the same route the client takes to their closest POP. Therefore, it doesn't stay within the network. For example, a disadvantage for this is if the client took a route to the POP that had only 20ms latency and then from the POP to the game server machine had 10ms latency. This was a total of 30ms latency. Now if the route the game server takes back to the client is 40ms latency, it would obviously be faster sending it back to the POP it came from which would keep it at 30ms latency back in normal situations. Therefore, it could increase latency in certain situations. It could also technically reduce latency if it's the other way around. However, I'd still like it to be more consistent.

For two, if we're going to be inspecting handshake validations and so on from the server-side, we need the packets the server sends back to the client to go back to the same POP the client is routing to. We could technically send an additional packet back the POP it came from the game server machine, but this isn't ideal. It'd be just better to keep it within the network and send the packet back to the POP it came from for consistency.

With that being said, @Dreae suggested creating a FOU driver that basically maps the client source IP to the POP server. Basically, incoming FOU packets will be scanned and we'll save the inner IP header's source IP (the client IP) to the outer IP header's source IP (the POP IP) for a period of time. Additionally, in the case the client's route changes to another POP, we should check on existing mappings if the POP IP is the same or not. If it isn't, simply update the POP IP to the new IP address. When the FOU tunnel sends packets back, we simply change the outer IP header's destination IP to the POP IP that the inner IP header's destination IP is set to (which would indicate the client for outbound FOU packets).

I believe this would be the best solution and this is definitely the game plan for forwarding traffic in Bifrost. Additionally, we've been having a lot of IPIP endpoint issues which has to do with Linux and DockerGen themselves. For example, the IPIP tunnel won't exist/create within the Docker container/network namespace after the game server starts which results in the game server not being able to bind to the internal IP and crashes. Restarting the game server repeatedly until this works is the best solution unless if [this](https://gflclan.com/forums/topic/65638-ipip-tunnel-bug-we-experienced-last-night/) bug occurs. This is quite annoying and I understand why our Server Managers and above get frustrated over this. Therefore, I believe setting everything up within the Docker container itself and not relying on DockerGen would be the best solution. This will require more modifications to our Docker [images](https://github.com/GFLClan/Anycast-Endpoint/tree/master/images), but that is okay and I believe it'd be for the best anyways since we're eliminating DockerGen. We could also make Bash scripts or programs to check if the IPIP tunnel comes up easier within the Docker container itself because doing that in DockerGen is buggy (e.g. you'll risk other servers not starting properly if you're stuck in a while loop for some time when waiting for another IPIP tunnel to come up).

## Handling Cached Packets
One big topic is how we're going to be handling cached packets with Bifrost. Since we want everything to be done within XDP, this is going to be tricky. The reason it is going to be tricky is because I've had zero luck convincing the BPF verifier when using dynamic payload. I've made a GitHub repository [here](https://github.com/gamemann/XDP-Dynamic-Payload-Matching) along with a XDP Newbies mailing list thread [here](https://marc.info/?l=xdp-newbies&m=158894658804356&w=2) regarding this. This was a struggle I was running into while making my XDP Firewall [here](https://github.com/gamemann/XDP-Firewall). According to the GitHub [issue](https://github.com/gamemann/XDP-Dynamic-Payload-Matching/issues/1) on my repository mentioned earlier, somebody was able to get it working when using a length of 12. However, since this is going to be dynamic, I'm going to need something longer than 12 bytes in size. Though, this may be fine for matching the cache packet request/response headers.

I'm even having issues pushing/popping headers into my XDP Forwarding program [here](https://github.com/gamemann/XDP-Forwarding/blob/master/src/xdp_prog.c#L205). Even though I start from the packet's data end and subtract four bytes into the position (since I was using an unsigned 32-bit integer), the verifier disagrees with it somehow. I'm honestly not sure what else I can do here, but I can give some other things a try to see if I have any luck. This all relates to the same issue in my opinion and it seems the XDP programmers haven't been able to get over this limitation according to some things I've seen in the mailing list thread I mentioned earlier.

Now this is in regards to matching the cached packet's request and response headers. The next thing is what we want to use to handle the cached packets themselves. We have two options here. We could either use [AF_XDP](https://www.kernel.org/doc/html/latest/networking/af_xdp.html) which is already used in Compressor [here](https://github.com/Dreae/compressor/blob/master/src/compressor_cache_user.c) or Netfilter. We could also use the user space, but that would copy the packet from the kernel which wouldn't be ideal in my opinion (e.g. specific attacks against cached packets could consume a lot more resources).

AF_XDP is basically user space sockets, but they support zero-copy. Non-like regular Linux sockets in the user space, this results in the packet not being copied to the user space from the kernel. XDP creates a fast path within the kernel to these sockets. While AF_XDP is nice, it is also a huge pain to maintain. Now, the code I linked above for Compressor, that code was very complex because it involved setting the frame size, descriptors, headroom, and so much more for the socket. This requires a lot of knowledge in hardware, the Linux kernel, and more. While this is something I love learning about (along with the Linux kernel in general), it is quite complex still (I'll likely understand a lot more over time). Thankfully, there is newer AF_XDP code from the XDP tutorial [here](https://github.com/xdp-project/xdp-tutorial/tree/master/advanced03-AF_XDP) that simplifies everything in my opinion. I've made a test program with it [here](https://github.com/gamemann/AF_XDP-Test) and it was working without any issues. It appears the constants they have for frame size, headroom, etc. are globalized as well meaning we'll be able to use it for any NIC driver, but I'll need to confirm. I know it works fine for the virtio_net driver which is what we'll be using mostly anyways.

Additionally, Compressor's AF_XDP program breaks on kernel 5.6 as mentioned in [this](https://gflclan.com/forums/topic/68487-sydney-pop-issue/) thread (I have not tested it with > 5.6 kernels yet). I believe the new AF_XDP code will resolve the issue though because Compressor's AF_XDP code is highly outdated and uses so many different components compared to the up-to-date AF_XDP code.

Now, I've been also having bad luck in getting AF_XDP sockets to bind properly using the virtio_net [driver](https://elixir.bootlin.com/linux/latest/source/drivers/net/virtio_net.c). Basically, when loading the XDP program in DRV mode, the AF_XDP sockets bind once, but do not bind afterwards due to the device or resource being too busy. I was suspecting at first this was an issue with the virtio_net driver, but after finding [this](https://www.spinics.net/lists/xdp-newbies/msg01342.html) mailing list thread, I'm starting to think it's a different issue and a potential solution would be to use the first XSK socket's umem in all other XSK sockets (shared umem). I did however, try to develop my AF_XDP program to used shared umem on all sockets, but unfortunately, ran into the following errors.

```
dev@test02:~/AF_XDP-Test$ sudo ./afxdp_loader 
Created thread 0
libbpf: Error: shared umems not supported by libbpf supplied XDP program.
ERROR: Can't setup AF_XDP socket "Device or resource busy"
libbpf: Error: shared umems not supported by libbpf supplied XDP program.
ERROR: Can't setup AF_XDP socket "Device or resource busy"
libbpf: Error: shared umems not supported by libbpf supplied XDP program.
ERROR: Can't setup AF_XDP socket "Device or resource busy"
libbpf: Error: shared umems not supported by libbpf supplied XDP program.
ERROR: Can't setup AF_XDP socket "Device or resource busy"
libbpf: Error: shared umems not supported by libbpf supplied XDP program.
ERROR: Can't setup AF_XDP socket "Device or resource busy"
Starting program...
```

Also, I found [this](https://www.kernel.org/doc/html/latest/networking/af_xdp.html#xdp-shared-umem-bind-flag) and it states if we're using shared umem, we need to bind each XSK socket on the same queue ID. This won't work for us because the packets are coming in on 0 - x RX queues (this is within the hardware as well so you can't use the same queue ID via `bpf_redirect_map()` in the XDP program itself) and there's really not a way to change that unless if you use [ethtool's flow-type](https://man7.org/linux/man-pages/man8/ethtool.8.html) to direct all traffic to a specific RX queue which'll impact performance and I couldn't get it to work on the virtio_net interfaces anyways (I don't think it has support for that specific driver). [Libbpf](https://github.com/libbpf/libbpf)-loaded XDP programs not supporting shared umem and the issue mentioned earlier in this paragraph has made me decide to not even try to experiment with shared umem any longer because it won't work for us either way.

We could also use Netfilter which is a hook in the Linux networking path that occurs after SKB allocation (NFTables and IPTables both use this hook to my understanding). I've made some test programs using Netfilter before [here](https://github.com/gamemann/Test-Kernel-Modules) for those interested. This wouldn't be as fast as AF_XDP sockets, but it'd still be decently fast since we're not copying the packet to the user space. We'd need to create character devices to establish communication from the kernel module to the user space which isn't a huge deal though. There's also complications like how we'd parse the responses in the kernel from the user space since we wouldn't have any user-space support to use libraries like [json-c](https://github.com/json-c/json-c) for example.

If we were to use Netfilter, we'd still need to get past the request/response matching within the XDP program, but on match, we could simply just do `return XDP_PASS;` to pass the packet down the network stack which'll go through Netfilter and its modules.

Those are the struggles we're dealing with regarding cached packets handling and also an issue when it comes to matching dynamic payload data within the firewall aspect of the program as well. Hopefully we can find a solution, though. I'm determined to get this working!

## Modules For Handshake Inspection & Whitelisting
One big thing I'm looking forward to in Bifrost is creating modules for games we host game servers in. Basically, we'll be inspecting the handshake the client makes with the game server on both sides and if it succeeds, we'll be allowing the client through and their traffic will be forwarded to the game server and back. I made a thread [here](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Anycast_Network/Notes/Anycast-Filtering-Notes-DDoS-Attacks-Filtering-And-More.MD) that goes more into detail on this aspect. However, the one thing I wanted to note is this plan is still active since we'll be receiving the outbound traffic to the POP the client is routing to due to the traffic forwarding plan announced earlier.

Additionally, I also wanted to go into detail on our plan on making this module system. Since we want to have the option to disable the module itself along with only enable it for specific forward rules, we're going to have a header in the main Bifrost program that looks like the following.

```C
#define SRCDS_HANDSHAKE	// Standard SRCDS server validation (e.g. CS:GO, CS:S, GMod, TF2, and more Source Engine games).
#define RUST_HANDSHAKE	// Rust server validation.
#define UE3_HANDSHAKE	// Unreal Engine 3 validation.
```

This would be the `module_config.h` header file.

Within the XDP program itself, we'll be doing the following for example.

```C
#include "module_config.h"
  
#ifdef SRCDS_HANDSHAKE
/* Do SRCDS handshake validation... */
#endif

/* ... */
```

This will all be put into account on compile time. So if a module is enabled on a global level, Bifrost's back-end will simply need to recompile the endpoint program with the `module_config.h` header altered. If a module needs to be disabled, it simply prepends `//` before the `#define` representing that module so it is ignored (i.e. a comment). I was thinking of making it so on the Bifrost back-end, an SSH key can be generated and we then use the SSH key on the POP servers to allow the Bifrost back-end to execute shell commands. The public SSH key would need to go into a user's `~/.ssh/authorized_keys` file on the POP server of course and that user will need to be privileged (or root).

Each module will have a BPF map indicating which forwarding rules it has it enabled for. We'll use `bpf_map_lookup_elem(&map, &fwdrule);` within the XDP program to check if it has it enabled and go on from there if it's enabled for that specific forward rule.

## Conclusion
There's obviously a lot more work and planning to be done, but we're making a lot of great progress in my opinion. I understand this is taking very long, but this is a very complex project and I've had a lot of time restraints recently as well along with being overloaded/extremely burnt out. As I mentioned in the past, this is the type of project you'd assign paid engineers to make for a good hosting company to handle fast packet processing/forwarding and mitigating larger (D)DoS attacks. We're designing this network to be as fast as possible along with being able to handle any type of (D)DoS attack as long as we have the global capacity for it (upgrading our infrastructure is an entire different story and ballpark, but that's something I'm making updates on as well and responsible for). I do truly believe once we have the plan finalized completely, we'll be able to get this project developed fairly quickly compared to how long it has taken since its announcement.

If you have any questions, please let me know!

Thank you for reading.

**[Original Source](https://gflclan.com/forums/topic/68519-another-big-bifrost-update-really-close-to-finalized-plan/)**