# Tech Update - Exciting Upcoming Projects And More!
**Created on April 19th, 2020**

Hey everyone,

I just wanted to provide an update on the technical side of GFL. We've been making a lot of progress recently and have exciting projects coming up! I will be covering these topics today.

## ACL Rules Removal From New Hosting Provider
We've recently purchased a dedicated server from a new hosting provider. I've been wanting to go with this hosting provider for quite some time and finally decided to do so when our GS2990wx machine's hard drive got [corrupted](https://gflclan.com/forums/topic/54921-gs2990wx-hard-drive-failure-ordered-new-machine/) last weekend.

The one thing that I've been waiting on is hearing back from the hosting provider's DC on whether they can remove ACL rules that can allow us to spoof traffic out as our Anycast IPv4 range. As of right now, all game server outbound traffic from the machine is flowing through the Chicago POP server. This results in bandwidth overage fees, additional latency, and more. When we get the ACL rules removed, we'd be able to use the TC BPF [program](https://gflclan.com/forums/topic/54419-c-ipip-direct-via-tc-egress-hookfilter/) I made and send outgoing traffic back to the client directly by spoofing the source IP as the Anycast network (along with removing the outer IP header, etc). For a full list of benefits from this TC BPF program, please read [this](https://gflclan.com/forums/topic/54459-recent-performance-issues-created-a-fix-for-one/) post here.

A couple days ago I was told this is possible from the DC. However, there is an one-time $35 fee for this and I need to provide the following:

* The IPv4 range (**92.119.148.0/24** in this case, obviously).
* An explanation on why I need these ACL rules removed for our IPv4 range.
* A signed authorization letter.

Considering this will be saving us possibly **$400+/m** of bandwidth overage fees in the future, I feel the **$35.00** one-time fee is worth it. I will be compiling all this information once I hear back from the hosting provider regarding the requirements for the signed authorization letter.

## Expansion With New Hosting Provider
So far, our new hosting provider has been awesome! I haven't seen any reports of our Rust Modded servers suffering from performance issues since moving to the new hosting provider. The machine has an Intel i9-9900K processor with 64 GBs of RAM and 1 TB NVMe storage and is turbo-boosting to **4.6 - 4.8 GHz** (all cores).

One thing that is unfortunate with the new hosting provider is when the processor gets near dangerous CPU temperatures, it will need to down-clock the clock speeds. I haven't seen this become an issue so far and it seems this is something they'll be improving on as well (getting better cooling and so on). The minimum speeds I've seen are 4.6 GHz on all cores under high load which is still pretty decent. It appears to be handling our servers without any issues anyways.

If things continue to go well, I plan to keep expanding with this hosting provider. If we do this, we will be dropping our machines with [Nexril](https://nexril.net/). Nexril has been a great hosting provider during our time with them and I've recommended them to many other server owners. The CEO (James) is very cool as well! However, their Intel i9-9900K machines are **$249.00/m** compared to our new hosting provider at **$139.99/m**.

I also plan to order the following two additional machines after proven stability and the ACL rules are removed allowing us to spoof out as our Anycast network:

1. Intel i9-9900K with 64 GBs of RAM and 1 TB NVMe storage -> This will host the rest of our game servers on Nexril at the moment.
1. Intel i7-9700K with 32 GBs of RAM and 500 GBs NVMe storage (custom build) -> This will host our intense game servers such as our high-slot Rust servers. The Intel i7-9700K won't include hyper-threading. Therefore, it will run slightly cooler and may allow higher turbo-boost speeds (i.e. higher single-threaded performance which is important for our intense servers).

Once I have an update on this topic specifically, I will let you know!

## GS2990wx And GS3900X Machines
[Dagreek](https://gflclan.com/profile/35558-dagreek/) ordered a new 1 TB SSD for the GS2990wx machine which recently suffered a hard drive corruption/failure. These machines weren't meeting the performance expectations needed and we had to move our game servers over to our new hosting provider. However, I am not counting these machines out! I still think the machine's hardware (processor, RAM, and storage) are great and if we can find out what was causing the performance issues (more than likely network-related), we can use these machines for some of our game servers still. Any that want to be located in NYC at least. I forgot to mention, we aren't paying for these machines as well!

I will be launching an investigation regarding these performance issues. I will start by creating C software that can pen-test the machine's NIC to see if that is the bottleneck. I will also be doing other testing as well. I want to see if the issue also exists on the GS3900x machine as well since I've only seen reports from servers on the GS2990wx machine (e.g. I wonder if it was the previously corrupted hard drive that may have been causing issues). Now that we don't have any active game servers running, I won't need to worry about debugging that might result in the game servers going down!

Once I have an update on this, I will let everybody know!

## Packet Loss On Machine With New Hosting Provider
One thing I've also noticed while playing on our CS:S 24/7 Dust2 server which is being ran on the machine with the new hosting provider is I see **1 - 4%** packet loss during busy times. This is obviously a concern, but I haven't seen any **noticeable** performance issues while this occurs. I had other users check as well and they were receiving 1 - 4% packet loss at the time. It appears my route from my city to the game servers is fine. My suspicion is the packets being sent back to the Chicago POP from the game server machine are not making it back in time. There appears to be packet loss at the POP when performing an MTR on the game server machine to the POP server. I've been doing a lot of debugging and the POP's load was only at ~50% when I saw these issues. I also see packet loss when the POP load was only around 30 - 35%. The POP also has two cores and after using the `perf top -a --sort cpu` command, I can see that one CPU was at **65%** CPU and the other was at around **33%**. [Compressor](https://github.com/Dreae/compressor) should be multi-threaded and using all RX queues. The fact that both CPUs had load should prove that statement.

I think this might be with our POP hosting provider's edge. I plan to make a ticket at some point regarding this.

Something else that will probably fix this is issue is running the TC BPF program I made. I noticed similar symptoms on the NYC machines weeks ago and after running the TC BPF program, the symptoms went away (e.g. 0% packet loss at busy times).

I will continue to do investigation on this to see if I can narrow down the issue. Unfortunately, it's quite difficult to debug since XDP isn't great at that. Once the ACL rules are removed, we'll be running the TC BPF program anyways.

## Anycast Infrastructure
I asked [Ben](https://gflclan.com/profile/13-ben/) to start searching for new hosting providers that support BGP sessions based off of [this](https://docs.google.com/spreadsheets/d/1abmV_mXWWCsVxHLfouSivyS7ch-PcUww8S6ksY66c5o/) list. I want to see if we can get 2 - 3 new hosting providers ready for when we expand our Anycast infrastructure (after we make Compressor V2, read below). If we can get the infrastructure expansion plan down now, it'll be a lot less work to do after creating Compressor V2.

One thing I also plan to work on is getting XDP-native support working with our POP servers. This will result in being able to mitigate much larger (D)DoS attacks since we'll be able to drop probably around 8 - 10x more packets per second than XDP-generic. This will be really helpful with Compressor V2 (read below).

Ben plans to do this in the next month. I can also do it if he doesn't get around to doing so, but it'll be a lot of communication work. Though, I had to do this quite a bit before when emailing 30+ hosting providers last time, lol.

## Compressor V1 Issues
I am currently making modifications to the current Compressor [project](https://github.com/Dreae/compressor) created by [Dreae](https://github.com/dreae). I created a GitHub fork [here](https://github.com/gamemann/compressor) that already has two commits. One commit spreads traffic redirected to the **AF_XDP** socket over all RX queues on the NIC. This resolved an issue back when we tried using [Hivelocity](https://hivelocity.net/) as a POP server in NYC (in our case, **A2S_INFO** caching wasn't working properly). The other commit simply passes non-forwarded ICMP replies to the network stack for debugging a POP server in our case.

The next issue I plan to address is TCP connections not working properly. I'm not sure what exactly is wrong with it and the code appears to be fine. TCP connections over Compressor has been an issue since the start and results in slow speeds, timeouts, and more for TCP. I've done debugging and confirmed requests are being received and sent back by Compressor. Rate limiting and connection limits are also not the issue since I tried increasing these on my local environment and had the same issue.

Once I find a fix for this, I plan to correct my existing pull [request](https://github.com/Dreae/compressor/pull/1) that I completely screwed up last night (sorry [Dreae](https://github.com/dreae), I'm new to Git pull requests!).

## Compressor V2
Compressor V2 is a **MAJOR** project [Dreae](https://github.com/dreae) and I will be working on. It will be a huge improvement compared to Compressor V1 and I want this created before expanding our Anycast infrastructure. We are still designing the layout of it all (well, mostly Dreae is since he's far ahead of me in regards to programming/networking). I am currently learning [Elixir](https://elixir-lang.org/) which will be used for the backbone and more so I can help contribute to the project. With that said, I plan to help with the design as well if needed.

Compressor V2 will have the following advantages over Compressor V1:

* All POPs will be controlled by a backbone. This makes it easy to add POP servers, game servers, and so on. This will also make it so we don't have to restart Compressor each time we need to update the config.
* Will be capable of filtering large (D)DoS attacks based off of addable filtering rules on the backbone.
* Malicious traffic will be dropped via XDP (see a comparison [here](https://blog.cloudflare.com/how-to-drop-10-million-packets/)).
* Forwarding and so on will occur within the user space allowing us to debug traffic via `tcpdump` and catch attacks (currently not available on Compressor V1 since XDP forwards packets before the network stack).
* TCP connections will work without any issues.
* Will be performing plain NAT to each POP server and vice versa. This will be more reliable than the current method and better performing in most cases as well. This will remove the need of the TC BPF program I made and give us more control over routing.

The project will be using a mixture of C, XDP/TC/BPF, IPTables, Elixir, Rust or GoLang, and more!

This is definitely something I'm very excited about as well. This is the type of project a team of high-paid engineers would be assigned to do. Therefore, the fact that we're making this without being paid is pretty neat :)

Once I have more details on this project and an official game plan, I will let you all know!

## Conclusion
That's basically it. As you can see, I am keeping myself busy! I am pretty excited to start troubleshooting some of these issues and work on the new projects such as Compressor V2.

If you have any questions, please let me know!

Thank you for your time.

**[Original Source](https://gflclan.com/forums/topic/55289-tech-update-exciting-upcoming-projects-and-more/?do=findComment&comment=252283)**

## Update
**Updated on April 20th, 2020**

I just wanted to provide a small update, Compressor V2 planning has started and the planning can be found here for those interested:

https://gitlab.com/srcds-compressor/compressor/-/issues

This is something I'm very excited about! I also wanted to say thank you to [Dreae](https://github.com/dreae) for all the hard work he is putting into this!

Thanks!

**[Original Source](https://gflclan.com/forums/topic/55289-tech-update-exciting-upcoming-projects-and-more/?do=findComment&comment=252507)**