# MTR/Trace Route Tutorial/Guide + Explanation
**Created on July 17th, 2020**

Hey everyone,

I decided to make a guide on running an MTR or trace route to troubleshoot general networking issues. This guide is mostly focused on running these tools against GFL's infrastructure (e.g. our CS:GO ZE server). However, these tips can easily translate to the Internet as a whole.

These tools are needed when a player is having network-related issues on our servers and website.

In this guide, we're primarily going to focus on Windows since that's what most of our users use. However, in the video and at the bottom of the written guide, I also provide some information for Linux (which can also be used for Mac users as well since Mac is Unix-based).

## Video
[Here's](https://www.youtube.com/watch?v=AE54FpHC8c0) a video I attempted to make on explaining an MTR and trace route. I made this quickly since I was in a rush, but I believe it goes over a lot of valuable information.

Since some users don't want to follow along, don't like long videos, or generally understand written guides better than videos (like myself), I made a written guide below as well.

## What Is A Trace Route?
A trace route is a tool/command you use against a specific host name or IP address to see what route you take to get to the destination. One thing to note is when you connect to anything on the Internet, you're going through "hops" to get there. These hops are routers that basically forward each packet/frame to the the next hop until you've reached your destination.

## What Is An MTR?
An MTR stands for "My Trace Route". It was formerly known as "Matt's Trace Route" a long time ago. An MTR is basically a trace route tool, but offers the following changes compared to a trace route:

1. Continuously sends replies until the tool is stopped.

1. Shows packet loss at each hop.

1. Doesn't wait for three replies from each hop. Therefore, the route populates nearly immediately after executing the command.

The three changes I mentioned above I consider all pros. Therefore, I highly recommend using an MTR instead of a trace route when troubleshooting networking issues.

## Running A Trace Route

Running a trace route on Windows is fairly simple. To start out, I'd recommend searching for "Command Prompt" in Windows and opening a cmd.exe window (Command Prompt). Alternatively, you may hit the **Windows Key + R** to display the "Run" box and enter `cmd` to open a Command Prompt window. From here, you'll want to run `tracert <Hostname/IP>` where `<Hostname/IP>` is either the host name of the destination or IP address.

In this guide, we're going to be using CS:GO ZE's host name which is goze.gflclan.com. You may also use the IP address which is 216.52.148.47. Here's an example:

```
tracert goze.gflclan.com
```

One thing to note is you will not want to provide a port in the IP/host name (e.g. `tracert goze.gflclan.com:27015`). This is because by default, trace routes use the ICMP protocol which doesn't include a port non-like the UDP/TCP protocols. There are trace route/MTR tools that allow you to use the TCP/UDP protocols which uses a source/destination port (the `mtr` command on Linux comes with these options built-in which is discussed at the bottom of this post a bit). However, that is outside the scope of this guide.

After running, depending on the route you take to the destination, it will take some time to complete. It takes time to complete because it waits for three ICMP replies from each hop. This isn't what an MTR does (which is one of the features listed above for it). If a hop times out, it's going to take even longer since there's a specific timeout for it. 

Here are the results from my trace route:

![1](https://g.gflclan.com/2998-07-17-2020-sy3K9uni.png)

Now the information here may be overwhelming to some users. Therefore, I will try to break things down the best I can.

```
Tracing route to goze.gflclan.com [216.52.148.47] over a maximum of 30 hops:
```

This line is basically for visibility/debug. If a host name is specified, it'll attempt to resolve to the IP address associated with the host name and output it in brackets after the host name as seen above. With that said, this also tells us the maximum hop count that is specified in this trace route (which is 30 by default). This means if there are more than 30 hops the route needs to take, it will not include any hops over the 30 mark in our results. Typically, routes should never exceed 30 hops and in most cases where they do, this is usually due to routing loops, etc.

**Note** - Usually the more hops you take, the further away your destination is from you. However, there are many sub-optimal routes on the Internet. Therefore, this isn't **always** true.

Now, let's go over the results themselves which are the following:

```
  1    <1 ms    <1 ms    <1 ms  10.1.0.1
  2    22 ms    18 ms    14 ms  cpe-173-174-128-1.satx.res.rr.com [173.174.128.1]
  3    32 ms    33 ms    37 ms  tge0-0-4.lvoktxad02h.texas.rr.com [24.28.133.245]
  4    20 ms     9 ms    18 ms  agg20.lvoktxad02r.texas.rr.com [24.175.33.28]
  5    12 ms    16 ms    12 ms  agg21.snantxvy01r.texas.rr.com [24.175.32.152]
  6    35 ms    28 ms    31 ms  agg23.dllatxl301r.texas.rr.com [24.175.32.146]
  7    25 ms    27 ms    21 ms  66.109.1.216
  8    42 ms    23 ms    24 ms  66.109.5.121
  9    25 ms    27 ms    26 ms  dls-b21-link.telia.net [62.115.156.208]
 10   166 ms    39 ms    32 ms  kanc-b1-link.telia.net [213.155.130.179]
 11   146 ms    83 ms    51 ms  chi-b2-link.telia.net [213.155.130.176]
 12    39 ms   193 ms   128 ms  chi-b2-link.telia.net [62.115.122.195]
 13    43 ms    43 ms    48 ms  telia-2.e10.router2.chicago.nfoservers.com [64.74.97.253]
 14    49 ms    45 ms    46 ms  c-216-52-148-47.managed-ded.premium-chicago.nfoservers.com [216.52.148.47]
```

As you can see, we have five columns here. I will explain each column below:

1. The first column indicates the hop number. This is auto-incremented on each hop and starts from one. I'll be using this number to indicate hops below (e.g. **hop #x**).

1. This is the latency (in milliseconds) of the first ICMP response received back. Basically the timing from when you've sent the packet to when you've received it. Typically, the lower the latency is, the better. If there was no response (e.g. a timeout), it will output an asterisk (*) instead of the latency.

1. Similar to the second column, this is the latency of the second ICMP response received back.

1. Similar to the second column, this is the latency of the third ICMP response received back.

1. This shows the IP address and/or the host name of the hop. I believe the trace route tool performs a rDNS lookup on the IP address to see if there's a host name associated with it on record. If there is, it will display the host name and then the IP address in brackets.
 
One thing to note is when you look at the host names, usually they give an indication of where the hop is located. For example, in my route we see hop #4 that has the host name **agg20.lvoktxad02r.texas.rr.com**. I believe this host name indicates the hop is located in Live Oak, TX, which is the current city I'm living in (which is technically inside of San Antonio, TX). The next hop (#5) has a host name of **agg21.snantxvy01r.texas.rr.com** and I believe this indicates the hop is in San Antonio, TX. I know for sure hops #6 to #9 are all located in Dallas, TX. You can usually confirm by looking at the latency you get to the hop as well (e.g. I'm only getting 12ms latency to the San Antonio hop which would make sense since I'm located in San Antonio, TX).

Anyways, we can see towards the end that we start routing to Telia in Chicago based off of the host name (e.g. **telia-2.e10.router2.chicago.nfoservers.com**) . NFO has a router with Telia which is hop #13 and hop #14 is our actual destination (the NFO machine that hosts our CS:GO ZE server).

If NFO or Internap (the data center NFO has their machines hosted in) blocks your IP, you will start seeing timeouts after hop #12 more than likely since you start getting blocked when trying to route into the NFO network. It will output an asterisk (*) in-place for the latency when there's a timeout. This is why we ask users who aren't able to connect to our CS:GO ZE server for trace routes to the server. In cases like these, this is usually due to the player's network performing port scans against NFO's network. Unfortunately, this usually indicates the player's computer or another device on the network is compromised (most likely) or the router is configured to perform port scans against networks (least likely).

I believe that's all you need to know for a trace route. Our Technical Team will be able to assist you with the results if you need any clarification or help on them.

## Running An MTR
Now it's time to learn about running an MTR. As mentioned before, an MTR is similar to a trace route, but includes some pros in my opinion. To my knowledge, Windows doesn't include an MTR tool by default. Therefore, I'd recommend using a third-party tool named [WinMTR](https://sourceforge.net/projects/winmtr/). This tool comes with a GUI making it more user-friendly.

After installing, you may run either the 32-bit or 64-bit versions (I'd suggest just using 64-bit since you're most likely running 64-bit nowadays). Just like a trace route, you may specify a host name or IP address under the "Host" field. Afterwards, feel free to hit "Start".

Here are my results:

![2](https://g.gflclan.com/2999-07-17-2020-cQZCuulq.png)

You'll notice that the route populates nearly instantly non-like a trace route. This is because it's not waiting for three replies from each hop. With that said, it also is continuously sending ICMP requests until you hit "Stop".

There are a few new columns when using this tool compared to a trace route. The most important column is "Lost %" which indicates how much packet loss we're getting to that specific hop. Now, the closer we are to 0%, the better. The percentage indicates how many requests we've sent that didn't get a reply back.

One big thing to note about packet loss to each hop is some hops do rate-limit ICMP responses or have them turned off entirely (meaning you'll have 100% loss to that specific hop). So if you see packet loss on a hop, but all hops after that still display 0% packet loss, this is most likely the reasoning and this is also nothing to worry about. If you see a hop with packet loss and after that hop, each other hop going all the way down to the destination has packet loss, that's when you know you're experiencing actual packet loss and the first hop experiencing the packet loss is probably the one dropping the packets (this isn't always true though).

In the above example, I have packet loss on some hops due to rate-limiting. However, you can see hop #14 (the destination), doesn't have any packet loss indicating there aren't any dropped packets to the destination itself. This was actually the results from what I did in the video above where I sent requests every 0.2 seconds instead of every second (I didn't have any packet loss when sending a request each second, but due to rate-limiting, I did have packet loss on some hops when sending a request every 0.2 seconds).

Other than that, the rest of the new columns are related to latency to each hop. Since we are continuously sending requests and measuring the latency for each response, this allowed for more columns such as "Best" (which is the lowest latency to the hop), "Avrg" (which is the average latency to the hop), "Worst" (which is the worst latency to the hop), and "Last" (which is the latency of the latest request sent out and received to the hop).

The "Sent" and "Recv" columns indicate how many ICMP requests we've sent to the hop and received. The packet loss column's ratio is **(Packets Sent - Packets Received) / Packets Sent**.

For outputting MTR results, you can either take a screenshot, use the "Copy Text to clipboard" button, or "Copy HTML to clipboard" buttons. You may also use the export buttons as well to output to a file. Typically, I'd suggest just using "Copy text to clipboard" and pasting the results.

Additionally, you may also hit the "Options" button and it'll show a box like this:

![3](https://g.gflclan.com/3000-07-17-2020-25djSekr.png)

The interval indicates the time in-between sending ICMP requests. The lower this number is, the more requests you'll send obviously. However, as explained above, lower values will lead to more rate-limiting on certain hops, etc. Usually there's no reason to change this from one second.

The ping size is the ICMP packet length in bytes. This is something you won't typically need to change, but if you want to send bigger packets, you may change this to anything under your MTU limit (I don't believe this would support fragmentation, so you'll need to have it under your MTU limit).

I believe LRU (least-recently-used) indicates how many hosts it can store in one route (similar to the max hop count in trace routes). I'd suggest just leaving this to default (128). And the "Resolve names" box indicates whether to do rDNS lookups on the IP address to get a host name for the specific hop. In WinMTR, when a host name is found via rDNS lookup and "Resolve names" is checked, it does not display the IP like a trace route does. Sometimes rDNS lookups are inaccurate. Therefore, this would be a good reason to disable resolving host names in certain cases where needed.

I believe that's about all you need to know for an MTR.

## Linux/Mac
You can execute an trace route or MTR on Linux/Unix-based systems (e.g. Mac) usually by installing the correct packages. I know for Debian-based systems such as Ubuntu, you can install these using `apt` (e.g. `apt install mtr`). On most distros, these tools are included by default. However, for minimal installations, you may need to install these packages manually.

I personally like using MTR on Linux due to all the options it comes with along with it just being a lot better than WinMTR. I'd suggest executing `man mtr` on your Linux OS to see what options you have. For example, take a look at [this](https://manpages.ubuntu.com/manpages/bionic/man8/mtr.8.html) for Ubuntu (or run `man mtr` in your Linux terminal). You'll see many more options such as being able to perform MTRs over the UDP/TCP protocols (e.g. `mtr --udp <host>` for UDP-based MTRs or `mtr --tcp <host>` for TCP-based MTRs). Sometimes this is useful when a hop completely disables ICMP replies and you want to see if you can get a response from the hop using the UDP/TCP protocols instead.

## Conclusion
I understand this is a pretty long post and it's pretty in-depth as well, but I hope this does educate some of you on how to use a trace route or MTR along with what they're good for.

As always, if you need any help with running these tools or inspecting the results, you may reach out to anybody on our Technical Team including myself.

If you see anything inaccurate in this post, please let me know! I'm always willing to learn new things and while I'm pretty sure everything is pretty accurate, there's a chance I missed something.

Thank you for reading!

**[Original Source](https://gflclan.com/forums/topic/59581-mtrtrace-route-tutorial-explanation/?tab=comments#comment-267292)**