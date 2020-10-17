# GFL's Anycast Network
## Overview
GFL's Anycast network was created in March of 2019 by [Christian Deacon](https://github.com/gamemann) along with [Dreae](https://github.com/Dreae), who made the initial/current packet processing software, [Compressor](https://github.com/Dreae/compressor) and provided guidance on the network infrastructure. We currently operate [AS398129](bgp.he.net/AS398129) and manage `92.119.148.0/24`.

GFL's game servers are behind this network on `92.119.148.0/24` and we also host a few of [HellsGamers](https://hellsgamers.com/) servers.

The Anycast network is responsible for forwarding legitimate game server traffic to our PoP servers (Point of Presence) and dropping malicious.

## Current Setup
As of **October 17th, 2020**, we currently use GameServerKings ([AS26863](https://bgp.he.net/AS26863)) and Vultr ([AS20473](https://bgp.he.net/AS20473)) as our upstreams.

With that said, we've used BGP communities offered by both providers to allow us to primarily use GTT ([AS3257](https://bgp.he.net/AS3257)) and NTT ([AS2914](https://bgp.he.net/AS2914)). This is because we found a lot of sub-optimal routes we couldn't resolve when using other carriers with Vultr. This is why we decided to stick to one - two primary peers for now.

## Future Plans
Our plans for the future are to continue to expand with GSK as they've been able to mitigate larger (D)DoS attacks aimed at our network nearly daily and find hosting providers with better hardware.

With that said, Dreae and Christian are working on a new firewall called [Bifrost](https://github.com/BifrostTeam) which will support many features such as fast packet processing/forwarding, BGP Flowspec support for mitigating larger (D)DoS attacks at the upstream level, and hooks for the fastest packet processing paths including [XDP](https://www.iovisor.org/technology/xdp), the [DPDK](https://www.dpdk.org/), TC (Traffic Control), and netfilter.