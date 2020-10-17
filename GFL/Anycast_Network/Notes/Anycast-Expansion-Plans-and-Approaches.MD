# Anycast Expansion Plans And Approches
**Created on July 19th, 2020**

Hey everyone,

I decided to make this thread to show people who are interested what plans I have for our Anycast network expansion along with the paths we can go down. This post will also include the challenges we have to face with expanding the network.

You may also be interested in [Dreae](https://github.com/dreae) and I's (Christian) plans for BiFrost (formerly known as Compressor V2) which are detailed [here](https://gflclan.com/forums/topic/57006-anycast-filtering-notes-ddos-attacks-plans-to-mitigate-and-more/). This includes what we plan on doing for application layer attacks.

This thread will be more focused on the infrastructure of the network.

## What Is Anycast?
Before going into our plans, I just wanted to briefly go over what an Anycast network is. Basically, it's a network we fully own that is put in-front of our game servers. We have multiple POPs (Point of Presences) which are servers scattered around the world all announcing the same IPv4 block (92.119.148.0/24). All of our game servers have a public IP from the mentioned IP block under this network. The client routes/connects to the POP server based off of the AS-PATH and/or BGP hop count. Usually, this is the closest POP server to them physically. From here, the POP server forwards the traffic to our game server machines that hosts the game servers themselves.

This network came with plenty of pros, but it's also **a lot** to maintain since we're fully responsible for the network (e.g. forwarding traffic, upkeep, (D)DoS protection/filtering, and more). This is pretty much the reasoning as to why I'm swamped all the time in GFL. Thankfully, this is something I'm very interested in and also something that has taught me A LOT about programming and networking.

For a more in-depth explanation, I'd recommend reading [this](https://www.cloudflare.com/learning/cdn/glossary/anycast-network/) CloudFlare post.

## BiFrost
I figured I'd start off with explaining what [BiFrost](https://github.com/BifrostTeam) is. BiFrost will be the new open-source packet processing/filtering software that Dreae and I will create and maintain. This was formerly known as Compressor V2 since we're using [Compressor](https://github.com/Dreae/compressor) right now made by Dreae. This software will be responsible for forwarding traffic our clients send to our game servers to the game servers themselves. Additionally, it will also be responsible for the following:

* Filtering and dropping malicious traffic from (D)DoS attacks.

* Caching specific packet types to fight against low-throughput attacks.

* Sending non-client outbound traffic from our game servers.

For more information, I'd recommend taking a look at [this](https://gflclan.com/forums/topic/57006-anycast-filtering-notes-ddos-attacks-plans-to-mitigate-and-more/) thread which goes in-depth on what type of filtering we'll be performing with BiFrost and more.

## Our Current Setup
Before going into detail on our expansion plans, I wanted to give an overview on our current setup.

Currently, we have 14 POPs scattered around the world. Each POP is a VPS from a single hosting provider. We have three POPs in Asia, four POPs in Europe, and seven POPs in North America.

Each POP runs [Compressor](https://github.com/Dreae/compressor) which is packet processing software made by Dreae. I've made changes to this software and added filtering as a temporary solution to mitigate (D)DoS attacks. Currently, I am still expanding these new, temporary filters to our POP servers.

With my new filters (which again, are a temporary solution until BiFrost is completed), most (D)DoS attacks should be dropped at the POP. Therefore, if anything, there may be some POPs that get over-flooded during attacks that affects a portion of our players. However, there's a high chance there will be POPs that do not go down. Therefore, not affecting the players routing through those POPs. We've also mostly eliminated a single-point-of-failure on the network by having the game server machines send traffic back to the client directly via my IPIP Direct [program](https://github.com/gamemann/IPIPDirect-TC) instead of having all outbound traffic from the game servers go back through a POP on the network. 

## Some Notes On Finding Hosting Providers
I just wanted to point out some things on what we need to look for when searching for new hosting providers for the network. I won't mention pricing since that's self explanatory.

Firstly, we need to make sure the hosting provider supports BGP sessions (this allows them to announce our IPv4 block). Some hosting providers do BGP sessions for free while others have either one-time fees or monthly fees. Obviously, I'd prefer finding providers that do these for free. However, if the fees aren't too pricey, I'd consider going with them depending on the quality of their service. Thankfully, there's a helpful Google Spreadsheet [here](https://docs.google.com/spreadsheets/d/1abmV_mXWWCsVxHLfouSivyS7ch-PcUww8S6ksY66c5o/) that lists hosting providers known for offering BGP sessions. There are still some hosting providers I've found that support BGP sessions, but weren't added to this list. So that's something to keep in mind. The list is maintained by the community which is neat, though.

Secondly, I usually try to find hosting providers that support both [GTT](https://bgp.he.net/AS3257) and [NTT](https://bgp.he.net/AS2914) as direct peers. I've found that primarily using these peers on our current network results in better routing along with it being easier to maintain. If we supported a bunch of direct peers, there would be a lot of sub-optimal routes due to the amount of direct peers we have. A lot of hosting providers including our current does support BGP communities ([example}(https://www.vultr.com/docs/as20473-bgp-customer-guide)). This allows us to tune routing easier via setting BGP communities in our BIRD config (the BGP daemon running on our POPs). However, we still don't have full control over the BGP routing. Therefore, there would likely be sub-optimal routes still. This is why I'd suggest just using one - two direct peers for the network.

Lastly, the bandwidth limits. Our game servers consume a lot of bandwidth (probably around ~40 TBs a month or more). Usually VPSs have lower bandwidth limits. Therefore, something to check is the bandwidth overage fees and ensuring those aren't too high in the case we do exceed the bandwidth limits.

## Direct Transit
I also wanted to mention the possibility of us purchasing direct transit in the future. Direct transit would allow us to peer with an upstream directly instead of going through our current hosting provider. For example, here's a BGP route in our current setup:

Our network ([AS398129](https://bgp.he.net/AS398129)) -> Our POP Hosting Provider Exchange (ASxxxx) -> GTT/NTT (Direct Peers) -> Other ISPs/peers to destination.

If we were to purchase direct transit from GTT or NTT, our route would look like this instead:

Our network -> GTT/NTT -> Other ISPs/peers to destination.

This would remove the hop in-between our network and the current direct peers we use to my understanding. Therefore, improving routing and potentially latency.

Unfortunately, direct transit is **VERY** expensive and I don't believe this is something GFL will be able to afford. At least from a [tier 1](https://en.wikipedia.org/wiki/Tier_1_network) ISP. Especially considering bandwidth is the main factor in pricing and our game servers consume a lot of bandwidth. Though, it's something I still plan on looking into. What would be neat is some of these peers we can purchase direct transit from gave us control over blocking traffic at the upstream level. Therefore, we'd be able to leverage their network capacity to block high-volume attacks.

## What Approaches We Can Take

I wanted to go into detail on the two likely approaches we can take with this project. As of right now, I'm honestly not sure which approach we'll be taking since it depends on many factors, in which, we won't know until we ask the hosting providers we want to go with.

### Quantity Over Quality
The quantity over quality approach is where we would be expanding with a high amount of cheaper POP servers. We would likely be renting cheaper VPSs from multiple hosting providers around the world and using them as our POP servers on the network. We'd likely get above 100 POP servers in total if we went with this approach.
 
One of the key reasons this approach is considered and beneficial is because we wouldn't be relying as much on the hosting provider's built-in (D)DoS protection against high-volume attacks. Currently, most VPSs include a one gigabit link. Let's say we had 10 POP servers with one gigabit links, if the hosting provider didn't do anything about high-volume attacks, we'd basically have 10 gigabits total network capacity. Unfortunately, any filtering we'd apply to BiFrost wouldn't be able to block attacks exceeding one gigabit in this case. Therefore, if we decided to buy 100+ cheaper VPSs with one gigabit links, this would potentially give us over 100 gigabits total network capacity (there's obviously a lot of other factors here such as the actual network capacity at each data center, multiple POP servers being on the same node, and so on).

The other pro to this approach is there would be fewer clients routing to each POP. Therefore, if a POP is over-flooded by an attack, less players would be affected.

The one potential con to this approach is the amount of maintenance required. Managing 100 servers is quite a task. We were planning to make as many things automated as possible with BiFrost. Therefore, if we did a good job with BiFrost, I do believe we could maintain it without too much work. The other thing is, since these POPs would be cheaper VPSs with less power, we'd need to ensure we have enough power to support BiFrost since that won't be as light-weight as our current software.

### Quality Over Quantity
The next approach I want to talk about is quality over quantity. This is self explanatory for the most part. Basically, we'd be likely renting out dedicated machines as our POP servers. Instead of having many cheaper POP servers, we'd have fewer very powerful POP servers.

The one pro about this is it'd be easier to maintain. However, there is a very important factor here. Most dedicated machines include a one gigabit NIC and link still. Unfortunately, machines with 10 gigabit NICs or higher are typically expensive. Therefore, we would **NEED** the hosting provider to be able to filter high-volume (D)DoS attacks for us. If the attack exceeds the POP's NIC/link, there is absolutely nothing our packet processing/filtering software (BiFrost) can do to mitigate the attack unless we have access to place filtering rules on the upstream level.

With that said, since there would be fewer POPs, more players would be routing through each POP. Therefore, if a POP was over-saturated or went down, more players would be impacted by this.

One other thing I may consider in the future if this all works out is doing colocation. This means we'd rent racks at multiple data centers, build our own machines, ship them to the DCs, and use them as POP servers. Before doing this, I'd need to learn how to build servers themselves (the hardware) which is something on my to-do list anyways since I want to build two personal home servers to do pen-testing with via XDP, DPDK, etc. With that said, there would be a higher up-front cost with this since we'd have to purchase the hardware to build the machines. However, the monthly cost should be a lot cheaper. Therefore, we'd save money in the long-run. We might also be able to find a data center that sells 10 gigabit links for somewhat cheap and we'd be able to build a machine with a 10+ gpbs NIC giving us the power to mitigate larger attacks on that specific POP.

### A Mix
The last approach would be mixing our POPs with quality and quantity. Therefore, for example, we'd have powerful POPs in locations we see a lot more traffic/attacks in. However, we'd have cheaper POPs in locations that don't have as much traffic.

Since BiFrost will be including POP monitoring and statistics in the future, one thing I considered is having all of our base POPs pretty powerful. However, if BiFrost detects a (D)DoS attack causing a specific POP to go over 70% CPU or network usage, we could automatically spin up cheaper POPs (VPSs) in that location to absorb the attack and bring the load down temporarily.

## The Plans
I'd like to now go over the possible plans for the first two approaches mentioned above.

### Quantity Over Quality
As of right now, if we went with a quantity over quality approach, we'd be looking for three - four solid hosting providers with our POP servers for redundancy and coverage reasons. We would be purchasing cheaper VPSs and each VPS would hopefully be less than $10/m. Each VPS would need 2 GBs of RAM to support BiFrost as well. However, they can include low storage space and one CPU core.

I was hoping to aim for 100+ POP servers at $700 - $900/m or so which isn't too bad considering the protection we'd gain from it.

As stated above, this would definitely be more maintenance. However, we'd have better damage control since less clients would be routing through each POP (so if one goes down, less players would be impacted). One thing we'd need to make sure of is having BiFrost handle everything on each POP automatically. This would result in less maintenance on our part.

### Quality Over Quantity
This is the approach I have a bit more planned out since I've been looking into it the past couple weeks.

Recently, [Hivelocity](https://www.hivelocity.net/) introduced more locations for their bare metal machines (AKA dedicated servers). You can find a list [here](https://www.hivelocity.net/dedicated-servers/#instant-bare-metal-servers-filter). We did use Hivelocity as a POP hosting provider at one point in the beginning of the year in NYC. However, we needed to cancel the machine due to reasons not related to them. When we did have the POP with them, it did work very well, though. The only complaint I had was the machine's NIC driver not supporting XDP-native (supported driver list can be found [here](https://github.com/iovisor/bcc/blob/master/docs/kernel-versions.md#xdp)). In fact, the NIC driver didn't even have enough headroom for XDP-generic causing the XDP program to process packets slower than IPTables which was eventually patched thanks to the XDP [maintainer](https://github.com/netoptimizer) (more information can be found in [this](https://gflclan.com/forums/topic/51387-another-update-on-new-nyc-pop-a-roller-coaster/) thread). I would need to discuss this with them and see if we can get a NIC that does support XDP-native. Either that or work to see if we can get XDP-native support added to the NIC drivers they currently use which is a more complicated process (I may be able to implement support, but that's pretty complex and requires understanding the Linux kernel code).

With that said, we would **NEED** to have Hivelocity filter high-volume attacks for us since these machines only include one gigabit NICs/links.

If everything went well with Hivelocity, I'd like to purchase their $50/m dedicated machines and use them as POP servers in most locations other than our prime locations.

As for our prime locations, I've been in-talks with another hosting provider who has a very nice setup. They claim they can easily mitigate high-volume attacks and they also offer some neat tools such as BGP [Flowspec](https://netcraftsmen.com/bgp-flowspec-step-forward-ddos-mitigation/) which would allow us to add filters to our upstreams via BGP. This would allow us to potentially block high-volume attacks ourselves by passing filters to the upstreams. This would be pretty advanced, but something I'm definitely interested in doing with BiFrost if possible. Unfortunately, they don't have many locations. However, they do have locations in Dallas and London at the moment along with plans to expand into New York City and Chicago. I consider these cities our prime locations and if we went with a quality over quantity approach, I'd like to use this hosting provider for our prime locations. They also offer colocation and custom builds if necessary.

We'd most likely have 10 or so $50.00/m POPs from Hivelocity and 3 - 4 $75.00/m POPs from the hosting provider mentioned above. This should range from $750 - $850/m or so which is my goal for this Anycast expansion.

## Conclusion
I hope this thread interests some people and also helps you understand what thought process I have going into expanding the network. There are obviously a lot of factors that goes into all of this and to be honest, I'm still not sure which plan we'll go with. It's pretty exciting planning all of this out, but I will admit it is a huge project and consumes a lot of time. Especially when you need to write many emails to hosting providers discussing our options, etc.

These expansion plans won't be executed until Dreae and I have completed BiFrost as well.

If you have any questions, please feel free to ask!

Thank you for your time.

**[Original Source](https://gflclan.com/forums/topic/59681-anycast-expansion-plansapproaches/?tab=comments#comment-267591)**