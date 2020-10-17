# Two Network Incidents From October 12th, 2020
**Created on October 14th, 2020**

Hey everyone,

I'm writing this thread for documentation purposes and I'm a bit late to writing it (I'm sorry, I've been all over the place recently, lol).

On **October 12th**, we experienced two network-related issues (one somewhat related to the other).

## Redis Server Outage
Around **9:42 PM CST**, Vultr (one of our hosting providers) had an outage in their Atlanta location:

![1](https://g.gflclan.com/3431-10-14-2020-SDbWt0s7.png)

Our Redis server for A2S_INFO caching is hosted at this DC and I wasn't able to route to it (there seemed to be a BGP issue with one of their upstreams since I didn't make it out of my ISP when running an MTR to the server's IP). This caused A2S_INFO responses to break temporarily until Compressor stopped using the Redis server (which I believe took around 10 - 15 minutes to kick in). During this time, players wouldn't be able to see the servers in the server browser. However, since we don't have any actual POPs in Vultr's Atlanta location, everybody should have been able to route to our game servers successfully (you'd need to connect directly via console if you wanted to play, though).

This lasted around an hour.

## Miami POP Outage
Our Miami POP experienced an outage at around **10:45 PM CST**.

When the Redis server came back online, I restarted [Compressor](https://github.com/Dreae/compressor) on all of our POP servers so they'd be able to establish a valid connection with the Redis server again for A2S_INFO caching. However, Compressor failed to restart on our Miami POP. Since our IPv4 block was still being announced via BGP (from BIRD), any clients routing through this POP would have lost access to our game servers.

Compressor failed to start because there wasn't enough memory available on the VM. Some of our POPs are still running with 1 GB of RAM and while adding SWAP has helped with this Compressor issue, sometimes our POPs fail to restart Compressor due to being out of memory still. A simple full VM restart resolves this issue and that's what I did.

This issue will be fully resolved once we finish expanding our Anycast infrastructure and all of our POPs run with more than 1 GB of RAM. With that said, I believe I can add a dependency to BIRD's systemd configuration so when Compressor's service is down, it will also take down BIRD which will stop announcing our IPv4 block via BGP. This won't fix the main issue, but at least if Compressor is found offline, BIRD will stop and clients will stop routing to the affected POP.

@Alexis This was the issue we talked about two days ago :)

If you have any questions, please let me know.

Thank you!

**[Original Source](https://gflclan.com/forums/topic/64374-two-network-incidents-from-october-12th/?tab=comments#comment-310847)**