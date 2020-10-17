# Big Game Server Machine Upgrades And More!
**Created on August 22nd, 2020**

Hey everyone,

I am creating this thread to let everybody know about some big changes that will be going into effect the next couple of weeks or so regarding our game server machines assuming things go smoothly. These changes will result in better performance for our game servers and will also cut down on expenses! Xy and I have been talking about this for the last few days and we both believe this is the best approach to go with. I truthfully believe we've found our niche with this move.

Before continuing, you may have noticed there have been **a lot** of threads like this in the past (regarding machine upgrades that is). To be honest with you, we've been trying to find the perfect hosting provider for our game server machines that comes with the best performance possible at a decent price. Due to the nature of our Anycast network, we're allowed to experiment with different hosting providers around the world without changing IPs since we own the /24 IPv4 block on our Anycast network (which forwards traffic to these game server machines). With that said, we've been rapidly expanding the past year in regards to new servers. Therefore, we've been needing more and more dedicated machines to help with the load on our existing machines.

I will now be going into detail on our old setup and why it's really messy. Afterwards, I will dive into the new upgrades and explain why we will benefit a lot from them.

## Our Current Setup
As of right now, our game server machines are distributed between three hosting providers in different locations. Our main hosting provider (Nexril) is located in Dallas, TX. We've been using Nexril since the start of the Anycast network. I would like to note that Nexril has been GREAT so far and I'd definitely recommend them to others. However, due to the new upgrades, we do plan on cutting these machines to save money. We also have a dedicated machine in Chicago with Ready2Frag (GS12). And finally, we have one last machine in NYC with Dedicated.com. However, this machine specifically doesn't cost us anything because dagreek is allowing us to use his machine he has colocated with them for free (thank you so much again!).

I will now list all machines we have currently along with their specs and price.

### GS08 (Nexril)
* Intel i7-7700K @ 4.2 GHz (4 cores/8 threads).
* 64 GBs of DDR4 RAM.
* 500 GBs SSD.
* Located in Dallas, TX.
* $120.00/m.

### GS09 (Nexril)
* Intel Xeon E3-1271v3 @ 3.6 GHz (4 cores/8 threads).
* 32 GBs of DDR4 RAM.
* 500 GBs SSD.
* Located in Dallas, TX.
* $80.00/m.

### GS11 (Nexril)
* Intel Xeon E3-1240v5 @ 3.5 GHz (4 cores/8 threads).
* 64 GBs of DDR4 RAM.
* 250 GBs SSD.
* Located in Dallas, TX.
* $110.00/m.

**Note** - This was a replacement for our GS10 machine that had hardware failures. We needed something very quickly at that time and this was the only machine available.

### GS12 (Ready2Frag)
* Intel i9-9900K @ 3.6 GHz (8 cores/16 threads).
* 64 GBs of DDR4 RAM.
* 1 TB NVMe.
* Located in Chicago, IL.
* $139.99/m.

**Note** - R2F hasn't been able to lift our ACLs that is preventing us from sending traffic out directly as our Anycast network. This isn't their fault directly, but it is a fault with their current DC (they've been taking months to get this change implemented sadly). This results in traffic going through the Chicago POP and bandwidth overage fees for that specific POP. Though, I do believe they'll have a solution in-place in the next few months or so. By that time, I'd assume we'll be with our new hosting provider, though.

### GS13 (Nexril)
* Intel i9-9900K @ 3.6 GHz (8 cores/16 threads).
* 128 GBs of DDR4 RAM.
* 1 TB NVMe.
* Located in Dallas, TX.
* $199.00/m.

### GS3900x (Dedicated.com & With Dagreek)
* AMD Ryzen 9 3900x @ 3.8 GHz (12 cores/24 threads).
* 32 GBs of DDR4 RAM.
* 2 TBs NVMe.
* Located in New York City, NY.
* Free.

**Note** - We did also use a machine that included the AMD Ryzen 2990wx Threadripper (32 cores/64 threads) with Dagreek for free. However, there were issues with the network on this machine causing performance issues and it later on experienced a hard drive failure. From what I can tell, the GS3900x machine hasn't had these same issues which is why we're still using it.

I'd like to also note that the current machines we have with the Intel i9-9900K for some reason appear to be limited to 4.2 - 4.3 GHz clock speeds on all cores under 50 - 60% load which honestly isn't good. For our GS13 machine, this doesn't appear to be a thermal limitation since the CEO confirmed this on his side. Please keep this in mind when I discuss our new upgrades below.

As of right now, we're spending **$648.99/m** with the above. With that said, we have a total of 40 cores (80 threads) to play with (only 28 cores if we exclude the GS3900x machine). However, we have quite a few machines that have older processors as well.

Due to upgrades I'd like to make to our Anycast network, I'd really like to spend $600/m or less on our game server machines until we expand more (which'll result in more income, obviously). Thankfully, this will be possible with the new expansion :)

## The Upgrades And New Setup!
So this is where the fun talk begins!

As some of you know, we've recently introduced [Gameserverkings](https://www.gameserverkings.com/) (GSK) to our Anycast network in our Dallas location. You can read [this](https://gflclan.com/forums/topic/60352-anycast-update-new-pop-hosting-provider-upgrades-and-more/) update for more information. I've been talking to the GSK owner, Renual, for the past month and I have to say he is honestly one of the smartest people I've talked to regarding networking, (D)DoS protection/mitigation, BGP, hardware, and more. He has been very helpful on advising us on what we should do with the Anycast network along with smoothly mitigating (D)DoS attacks against our network. He has went far and beyond on setting up policies that even allows him to mitigate (D)DoS attacks that would normally go through our POPs with other hosting providers such as Vultr who doesn't have as much (D)DoS protection as GSK and also advising us on packet processing/filtering software I plan to create with [Dreae](https://github.com/dreae) (Bifrost) along with personal projects of mine (e.g. my XDP firewalls and so on). He even allowed us to do a trial-run with our current GSK POP in Dallas. We've been using our POP with GSK the past couple of weeks without paying anything (this will be changing soon, though). Anyways, it's safe to say that I'm very impressed with GSK and Renual.

After talking to Renual about available hardware with GSK along with features, I instantly became interested in using GSK as our game server machine hosting provider.

The following are the three new machines I'd like to introduce to our setup in the next few weeks after discussing with Renual and Xy.

### GS14 (GSK)
* Intel i9-9900K @ 3.6 GHz (8 cores/16 threads).
* 64 GBs of DDR4 RAM.
* 500 GBs NVMe.
* Located in Dallas, TX.
* $149.99/m.

### GS15 (GSK)
* Intel i9-10900K @ 3.7 GHz (10 cores/20 threads).
* 64 GBs of DDR4 RAM.
* 1 TB NVMe.
* Located in Dallas, TX.
* $187.99/m.

### GS16 (GSK)
* Intel i9-10900K @ 3.7 GHz (10 cores/20 threads).
* 64 GBs of DDR4 RAM.
* 1 TB NVMe.
* Located in Dallas, TX.
* $187.99/m.

We are already in the process of setting up GS14 (we haven't paid for it yet, but we will likely be doing so after we set it up which will be sometime this weekend). As of right now, GSK doesn't have the Intel i9-10900K available. However, Renual stated he should have some for us in the next couple of weeks. Therefore, we'll need to wait until then to setup GS15 and GS16.

This new setup costs us **$525.97/m** and since we plan on keeping the GS3900x for less populated servers, we will still have 40 cores to play with just like our current setup. With that said, the processors above will be a HUGE improvement compared to the older processors we use with Nexril at the moment. The Intel i9-10900K is one of the fastest processors out at the moment in regards to single-threaded performance which is what matters the most with game servers. The Intel i9-9900K isn't far behind on that as well.

Each machine with GSK are also liquid-cooled. Renual has stated that we can overclock these machines if we'd like as well. However, I don't think it's worth it for these processors since they can get very hot and I'd prefer stability over a small increase in performance by overclocking. Though, it's nice knowing we have the option just in case which most hosting providers do not provide.

On Thursday, I ran CPU stress tests on the GS14 machine and the Intel i9-9900K was able to clock to **4.7 GHz** on all cores under 100% load. I had the test running for two hours with the same results. As mentioned before, our current machines weren't able to get above 4.2 GHz (all cores) on the Intel i9-9900K at 50 - 60% load. I'm still not sure what the reasoning to that is, but I do know that GS14 will be able to get to 4.7 GHz (all cores) without running into thermal limitations which is a great sign. With that said, the Intel i9-10900K will be able to clock at **4.8 GHz** on all ten cores under heavy load once we introduce GS15 and GS16.

One thing I'd like to note is we won't have a KVM attached 24/7 to these machines (which is something we do have with our current hosting providers). However, Renual stated they have 24/7 support who are able to attach a KVM to the machine at any time and should be ready within minutes. We also have the option of purchasing the KVM hardware ourselves and having Renual attach it to our machines so we can use it 24/7. This will be a one-time fee (one time per machine), but I don't really see a need for it right now if we can get fast support and a KVM attached within minutes. I'm not expecting to run into issues with these machines as well, but it's always great having a backup plan since we've definitely benefited from KVM access in the past (e.g. dealing with kernel panics, network connectivity lost, and more).

This new setup with GSK will be completely replacing our current setup. This will save us **$123.02/m** and each game server will see a big performance increase once moved to the new machines in regards to hardware/clock speeds, at least.

In conclusion, with the expenses decrease, big increase in performance for all game servers under our Anycast network, and the strong connection we have with GSK and Renual, I believe this is literally a win-win move for us. Given how well GSK has been doing for our Anycast network and the fact that the company itself has SO much potential, I believe this will be our long-term solution.

## NFO Machine Upgrade
I just wanted to briefly discuss a potential NFO upgrade in the future. I've been notified by our Server Managers running servers on this machine that they'd like a machine upgrade and I do agree with them, but it's hard since we don't have many servers running on NFO currently. As of right now, we have one machine with NFO running the Intel Xeon E3-1275v6 @ 3.8 GHz (4 cores/8 threads) with 64 GBs of RAM and a 1 TB SSD. This is costing us **$164.99/m** at the moment. We still have this machine because it hosts game servers we've had for a very long time including our CS:GO Zombie Escape server and we don't want to move them to the Anycast network due to IP changes along with the fact that the network isn't fully completed yet (we'll be likely getting everything completed in the next six - twelve months or so).

Other servers running on this machine other than CS:GO Zombie Escape include:

* CS:S Surf RPG DM (this is likely moving to the Anycast network soon, though).
* CS:S BHop (I'd also like to move this server to the Anycast network, but I still need to talk to Server Managers about it).
* GMod Hide and Seek.
* New/future servers that require Windows and won't work with Wine on Linux (e.g. our future ARK server).
* A bunch of test servers.

As of right now, the only machines available where we'd see a single-threaded performance improvement in would include six and eight cores. The machine with six cores would include the Intel Xeon E-2286G @ 3.8 GHz with 64 GBs of RAM and a 960 GBs SSD. This would cost us $219.99/m with a $99 one-time setup fee. The turbo-boost on the Intel Xeon E-1286G is 4.7 GHz while the turbo-boost on the Intel Xeon E3-1275v6 (current) is 4.2 GHz. I'm not sure what the all-core turbo boost speeds are on each CPU. I'll need to do more research on that. Lastly, the machine with eight cores (the Intel Xeon E-2286G) has a turbo-boost of 4.9 GHz, but would cost us $234.99/m and has an one-time setup fee of $99.

So obviously, we'd see a big improvement on the single-threaded performance most likely (I just need to confirm the all-core turbo-boost speeds of each CPU first along with the cooling solutions in-place on NFO to make sure we don't run into thermal limitations). However, I really would like to put the machine to more use because running CS:GO Zombie Escape and GMod Hide and Seek on a machine with six or eight cores costing us $200+/m just isn't worth it in my opinion. Perhaps we could look into setting up Battlefield servers again (we used to host very popular BF3 and BF4 servers years ago). Although, there will be slot fees for those servers (thanks, EA...). With that said, our future ARK server will also be hosted on this machine since it requires Windows and will not work on Wine for Linux due to the custom plugins needing a Windows library.

Anyways, this is something I will bringing up to our staff team in the next few weeks and we'll see what we can do. I'll also try asking NFO if we can get a custom-built machine. Currently, they've disabled hyper-threading on our current CPU due to security reasons. I believe this has impacted performance a bit as well, but I'm not sure.

I'll make sure to keep everybody updated on this in the future!

## Conclusion
Overall, these are pretty exciting changes we have coming up! We will be introducing three new **POWERFUL** machines that'll replace our current setup. The new setup will cost less monthly and each game server under our Anycast network will also see a big performance improvement regarding the hardware/clock speeds they're running on. The new machines will come with top of the line hardware regarding single-threaded performance which is what matters most for our game servers. We'll also benefit from the strong connection we have with our new hosting provider (GSK)!

Lastly, I am also looking into upgrading our NFO machine that hosts a few of our older servers. This should help a lot with performance on our CS:GO Zombie Escape and GMod Hide and Seek servers. Though, nothing is solid yet and I still need to do more research.

Xy and I will be making more announcements when we're ready to move servers to our new setup.

If you have any questions or concerns, please feel free to reply!

Thank you for reading.

**[Original Source](https://gflclan.com/forums/topic/61638-big-game-server-machine-upgrades-and-more/?tab=comments#comment-274096)**

## Update #1
**Updated on August 23rd, 2020 at 6:51 PM CST**

For those curious, I've performed another stress test using the Ubuntu package on our GS14 machine for 15+ minutes. GS14, as explained in the original post, includes the Intel i9-9900K. Here are the results!

### 100% Load
![1](https://g.gflclan.com/3187-08-23-2020-x7tuSdCL.png)

![2](https://g.gflclan.com/3188-08-23-2020-LuTO4O8h.png)

![3](https://g.gflclan.com/3189-08-23-2020-MqjsX66q.png)

We saw **~4.7 GHz** consistently for 15+ minutes at 100% load on all cores :) This would be the biggest test and it stayed around 90C. The dip you see on the CPU frequency graph was one core going from 4.7 GHz to 4.69996 GHz.

Additionally, if the temps stayed around 90C for 15+ minutes, it will more than likely not get any hotter. However, we did run it for a while last night just to be sure and it stayed around these temperatures.

Command executed to initialize stress test:

```
stress --cpu 16
```

### 50% Load
![4](https://g.gflclan.com/3190-08-23-2020-OLdc9f51.png)

![5](https://g.gflclan.com/3191-08-23-2020-wI1HqjeG.png)

![6](https://g.gflclan.com/3192-08-23-2020-L8OV8j00.png)

Command executed to initialize stress test:

```
stress --cpu 8
```

**Note** - I believe the reason the temperatures stayed high is because we were putting load on all cores technically still. When executing the above command, I believe the first 1 - 8 are the physical cores themselves. 9 - 16 would be the threads. Regardless, it still performed great, as expected.

I'm actually now really curious how the Intel i9-10900K will perform at 100% load on all cores with our current liquid cooler. It will be interesting to see, but keep in mind that our machines will likely only see 50 - 60% load (during peak) assuming we don't overload them (which is the plan, of course).

Thanks!

**[Original Source](https://gflclan.com/forums/topic/61638-big-game-server-machine-upgrades-and-more/?do=findComment&comment=274518)**

## Update #2
**Updated on August 23rd, 2020 at 7:31 PM CST**

And finally, I also made a CS:S test server and spawned in 63 bots with a DM plugin enabled [here](https://www.youtube.com/watch?v=jYLCmcVGqEw).

It worked out quite well. At some points there were small FPS dips, but I believe this was due to collisions (block was enabled for players/bots) and defuse kits being left behind by the CTs (when they collide, it can impact server performance).

The CPU cores were running at around **4.7 - 4.9 GHz** at this time!

**[Original Source](https://gflclan.com/forums/topic/61638-big-game-server-machine-upgrades-and-more/?do=findComment&comment=274533)**