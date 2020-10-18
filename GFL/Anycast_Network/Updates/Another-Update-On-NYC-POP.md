# Another Update On New NYC POP (A Roller Coaster)
**Created on January 24th, 2020**

Hey everyone,

I just wanted to talk about our new NYC POP. As some of you already know, I started announcing our new POP in NYC later [last week](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Anycast_Network/Updates/NYC-POP-Fixed.md) after fixing an issue that lasted two weeks.

While our Rust server was hitting 120+ players, I noticed packets were still being dropped from the machine hosting game servers to the new POP server. This resulted in bad performance. I wasn't sure if these specific packets were being dropped on the POP level or if it was the machine hosting game servers dropping the IPIP packets. Sadly, I was't aware of a way to measure the CPU usage of our packet processing software ([Compressor](https://gitlab.com/Dreae/compressor)) since all the processing occurred in the kernel (within the XDP hook). Therefore, commands like top or htop wouldn't show the CPU usage properly since that's within the user space.

I decided to reach out to the XDP Newbies mailing list [here](https://www.spinics.net/lists/xdp-newbies/msg01531.html). I received a couple replies suggesting to use the Linux '[perf](http://www.brendangregg.com/perf.html)' tool (one of the replies was from the XDP maintainer [himself](https://github.com/netoptimizer) which was honestly awesome!). I did a test report on our NYC POP and sent it to the XDP maintainer (Jesper) and he [replied](https://www.spinics.net/lists/xdp-newbies/msg01537.html) back stating the NIC we're using (running the igb driver) does not support XDP-native. Therefore, packet processing in XDP (via XDP generic in this case) is actually slower than packet processing in the user space. This is obviously a big issue.

I opened a ticket with our hosting provider asking if we could replace the NIC card in our server with something that supported XDP-native. They said this was okay and free of charge, but they only had specific models available. I did research yesterday and was trying to find a driver ([from](https://github.com/iovisor/bcc/blob/master/docs/kernel-versions.md#xdp) this XDP-native supported list) that had upstream support (e.g. we wouldn't need to recompile the kernel with a patch on the driver). Unfortunately, all the drivers that were supported within the upstream were all for 10 gigabit NIC cards from what I found (which were more expensive, obviously) and Hivelocity would only ship out a 1 gigabit NIC card. We agreed to ship a NIC card that uses the e1000e driver. While this isn't officially supported upstream, there is a [patch](https://github.com/adjavon/e1000e_xdp) that adds XDP/XDP-native support into the driver. This means I'll have to grab the kernel's source, modify this driver, and recompile the kernel (this is the first time I'm doing this). They are shipping this NIC card today and expect it to arrive in NYC by next Monday or Tuesday. This weekend I will be working to recompile the kernel and add XDP support to the e1000e driver. After I do this on my test VMs without issues, I will perform the recompile on our NYC POP (we have KVM access, so I'm not worried). After the NIC arrives, I will test everything to ensure XDP-native is supported and things work.

In the meantime, I've stopped announcing the new NYC POP from the network and started announcing the 4-core VPS with Vultr.

I understand this has been a long process (especially with the issue that took two weeks to resolve). I am trying my best to get this all up and running smoothly as soon as possible and will continue to work on it. I just wanted to thank all of you for your patience regarding this.

If you have any questions, feel free to ask!

Thank you for reading.

**[Original Source](https://gflclan.com/forums/topic/51387-another-update-on-new-nyc-pop-a-roller-coaster/?do=findComment&comment=239117)**

## Update
**Updated on January 28th, 2020**

Hey everyone, I just wanted to provide an update.

Our hosting provider will be replacing the NIC card today in our new NYC POP. With that said, I've compiled a custom Linux kernel and I've attempted to add XDP support into the **e1000e** driver. I did this on a test VM and transferred the image over to the new NYC POP. The new NYC POP was able to boot into the custom kernel and Compressor appeared to work fine. We won't know about anything until the NIC card is replaced, though.

My only concern about this case is the **e1000e** XDP patch I found was made for kernel 4.10.2 (we're running a kernel much higher). Unfortunately, I had to add the patch code in manually and change a few things since the patch was outdated. I'm not sure if what I changed will actually work considering I've never written NIC driver code before. It'll be interesting seeing if it works or not, though.

I will do plenty of testing before announcing the new NYC POP to the Anycast network.

Once I have another update, I will let you all know.

Thank you.

**[Original Source](https://gflclan.com/forums/topic/51387-another-update-on-new-nyc-pop-a-roller-coaster/?do=findComment&comment=239707)**

## Another Update
**Updated on January 30th, 2020**

Hey everyone,

I just wanted to provide another wonderful update. Our NIC card was replaced with a NIC card that supported the 'e1000e' driver. I compiled the Linux kernel and tried implementing XDP support via [this](https://github.com/adjavon/e1000e_xdp) patch for this driver. Unfortunately, the POP server was still using "XDP generic". However, thankfully the networking on the machine didn't go down entirely (this was my first time compiling Linux kernels and modifying NIC drivers).

The problem with the patch mentioned above was it is based off of kernel 4.10.2 (our current kernel is more up-to-date and we need to run 4.15+ at least). There weren't just differences between the `e1000e` driver from 4.10.2 and our current kernel. There were also major differences between the networking core (e.g. needing to implement XDP Xmit support and other key changes such as 'ndo_xdp' => 'ndo_bpf'). I tried inspecting the networking core source code, but unfortunately, I don't have a good enough understanding of the kernel-level networking code to implement proper XDP support (yet, this is something I'm very interested in :)).

Thankfully, yesterday, [Jesper Brouer](https://github.com/netoptimizer) (the XDP maintainer) provided two patches for the 'igb' driver (the driver our old NIC used) that, to my understanding, makes it so the 'igb' NIC driver doesn't have to reallocate each packet. Therefore, making the performance closer to the networking stack (faster than before). I've attached the two patches he provided to us in case anyone wants to take a look. While this isn't adding XDP-native support, it is definitely better than what we had before. This is the email he sent me for anyone wondering what the patches do exactly:

> \> I understand the `e1000e` driver doesn't have XDP support upstream.<br>
> \> Therefore, I tried implementing the driver patch that adds XDP support<br>
> \> here <https://github.com/adjavon/e1000e_xdp>. Unfortunately, this patch<br>
> \> was based off of kernel 4.10.2 (I'm using 4.19). Therefore, I had to<br>
> \> manually implement the patch code (this is my first time messing with<br>
> \> NIC driver code). Sadly, it doesn't seem like the patch worked based off<br>
> \> of the 'perf' results which are attached. I still see "do_xdp_generic"<br><br>
> I do think it would be more valuable to implement XDP for driver igb.<br><br>
> Given igb is a "slow" 1Gbit/s interface, we could also just fix XDP-generic to avoid the reallocation of the SKB for every packet... I've attached two patches which does exactly that. It sounds like you know howto apply patches and recompile your kernel(?)<br><br>
> I've tested this on my testlab.  The igb hardware seems is limited to 1.2Mpps.  Using an XDP_DROP test program, I observe the CPU processing packets use 50% CPU time before, and 36% CPU time after the patches.

I just wanted to give a shout out to Jesper Brouer. He has been extremely helpful and considering he's the XDP maintainer (along with a Redhat developer from the looks of it), he is very talented/smart! I've also asked on the mailing list thread if it would be worth implementing XDP-native support to the 'igb' driver. If the performance increase is worth it and I gain more experience, I will see if I am able to do this myself if nobody else is willing to do so due to how outdated the NIC driver is.

With that said, I've found out that our POP servers are also using XDP generic. They're using the "virtio_net" NIC driver (since they're VPSs) which is tricky to add XDP-native support to according to Jesper. The physical NIC card would have to support XDP-native as well. Therefore, I opened a ticket with our POP hosting provider (Vultr) and asked if their node's NIC cards support XDP-native. They told me some do and some do not, which is confusing. Anyways, Jesper mentioned we'd need to disable certain offloads on the server itself in order to get XDP-native working (in the case the node does indeed support XDP-native).

I replied back to the mailing list [thread](https://www.spinics.net/lists/xdp-newbies/msg01531.html) with questions such as which offloads we'd need to disable and if it's worth trying to implement XDP-native support into the 'igb' driver. Here is what I said exactly:

> Hey Jesper and Matheus,<br><br>
> Thank you for the information and assistance! I really appreciate it!<br><br>
> Matheus, I was able to confirm that my POP servers (running as VPSs with the "virtio_net" driver) are indeed using XDP generic based off of the command you provided. I didn't realize it included "xdpgeneric" on the link list. This is definitely an easier way to tell if you're using XDP generic compared to looking at the 'perf' results.<br><br>
> Jesper, thank you for those two patches! I've recompiled the 4.19 kernel with those implemented and have it running on my POP server now. I've started announcing the POP server to the network again and I will let you know how it goes along with getting results from the 'perf' command. Do you think it would be worth trying to implement XDP support into the `igb` driver? I understand if it can't be supported officially, but once I gain more experience, etc. I might give it a try if it would increase performance.<br><br>
> I figured the physical NIC card used to host the VPSs would need to support XDP-native, but I just wanted to make sure. I tried asking my hosting provider if the NIC card uses a driver that is on the XDP-native supported list. They came back stating some nodes do and some do not, sadly. What type of offloads would need to be disabled in order to get it working (in the case the node's NIC driver does support XDP-native)?<br><br>
> Thank you for your time. 

I've asked our hosting provider to switch the networking back to the NIC card with the `igb` driver and I've recompiled our Linux kernel again implementing the patches mentioned above. After some testing, everything appears to be working fine and Compressor was able to route traffic fine to my test server on the new NYC POP. Therefore, I've started announcing the new NYC POP to the network again and will be doing performance benchmarks.

There's another issue I'm trying to get sorted with the hosting provider Dagreek is colocating with in NYC (the machines that host our game servers in NYC). Since they're using Internap, one of their technologies to detect the best route to the server keeps changing the route to our Anycast network constantly (so it'll start going from our NYC POP to our Toronto POP for example, which is annoying). I've submitted a ticket with them last week regarding this and still waiting for an update.

I hope this update helps and we'll see how things go. Once I have another update, I will let you all know.

I just wanted to thank all of you for your patience and just know I am doing my best to get this POP server up and running with the best performance possible.

Thank you for your time.

**[Original Source](https://gflclan.com/forums/topic/51387-another-update-on-new-nyc-pop-a-roller-coaster/?do=findComment&comment=239919)**