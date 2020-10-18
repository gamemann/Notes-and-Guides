# NTT Peer Disabled On NA POPs
**Created on October 11th, 2020**

Hey everyone,

This morning I stopped announcing NTT to all of our NA (North America) POPs. This is so if one of our servers get (D)DoS'd, GSK will be able to perform GTT offload which will now result in all traffic going into our NA POPs to go through GSK's network and their upstreams.

We had an attack from a couple nights ago that I was informed on where our NA POPs were forwarding over 1 gbps malicious traffic to our game server machines. This is because our NA POPs do not have my newest filters non-like our Europe/Asia POPs. This is because something needs to be implemented into the new filters that I feel isn't worth implementing at this moment (I feel I'm spending way too much time on a "temporary" solution). You can read more on this [here](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Anycast_Network/Updates/Update-With-New-Filters-On-Compressor.MD). Our Asia/Europe POPs haven't forwarded any malicious traffic yet due to my filters which is a good sign. With this change, we should be mostly protected on the network.

There's a chance this may alter some player's routes who were routing to our NA POPs via NTT before. If you see any negative effects from this (e.g. sub-optimal routes, higher latency, etc), please reach out to me.

If you have any questions, please feel free to reply!

Thank you.

### EDIT

I also want to note that only announcing to GTT as a direct peer on our NA POPs is a temporary solution. Once the network is fully expanded out regarding the infrastructure and Bifrost is completed, I will be introducing a blend of carriers/peers such as Telia ([AS1299](https://bgp.he.net/AS1299)), Hurricane Electric ([AS6939](https://bgp.he.net/AS6939)), and more which will result in better routing as long as we tune our BGP properly (which is the plan, of course).

**[Original Source](https://gflclan.com/forums/topic/64232-ntt-peer-disabled-on-na-pops/?do=findComment&comment=310320)**

## Update
**Updated on October 13th, 2020**

I started announcing NTT again. This is because we started seeing sub-optimal routes since we still need NTT announced to some of our Europe/Asia POPs (GTT isn't available in these specific locations). Basically, if an ISP in NA peers with NTT directly, but not GTT, it would route them to the closest POP that uses NTT which would be overseas. This is obviously a big no-no.

If we get attacked, we can withdraw NTT temporarily until the attacks stop.

Thank you.

**[Original Source](https://gflclan.com/forums/topic/64232-ntt-peer-disabled-on-na-pops/?do=findComment&comment=310662)**