# Two New POPs Added And Other Changes
**Created on October 13th, 2019**

Hey all,

I just wanted to let everyone know we've added two new PoP servers to our Anycast network. One is located in Atlanta and the other is located in Toronto. This is to help more with the routing within the network.

Unfortunately, the Atlanta PoP is getting very poor routing at the moment (little to no traffic is flowing through it). I've submitted a ticket to Vultr with the following contents (besides the BIRD config for security reasons):

> Hello,<br><br>
> I've setup two more PoP servers for our Anycast network on Vultr (one in Atlanta while the other is in Toronto). Our Atlanta PoP is receiving very little traffic when BIRD is listening. We only want to announce to GTT (AS3257) and NTT (AS2914) since we found these peers to be the most stable. We are doing this globally via our Anycast network.<br><br>
> As of right now, Vultr's Atlanta location is the only host I've seen actually route to the Atlanta Anycast PoP we have (I tested this by using Vultr's Atlanta Looking Glass to our Anycast network).<br><br>
> When I remove all filters from the BIRD config, it does start receiving more traffic. However, as I stated above, I'd prefer only announcing to GTT and NTT. With that said, there were still hosting providers located in Atlanta that wouldn't route to the Atlanta PoP regardless of empty filters.<br><br>
> As it stands, here is my BIRD config with the current filters:<br><br>
> ```<br>
> <stripped for security><br>
> ```<br><br>
> Here is one hosting provider's Looking Glass in Atlanta not routing to our current PoP:<br><br>
> http://lg.atl.colocrossing.com/ <br><br>
> ```<br>
> traceroute to 92.119.148.4 (92.119.148.4), 30 hops max, 60 byte packets<br>
> 1  192.168.104.2 (192.168.104.2)  0.678 ms  0.636 ms  0.611 ms<br>
> 2  192-3-16-129-host.colocrossing.com (192.3.16.129)  1.310 ms  1.963 ms  1.941 ms<br>
> 3  10.4.1.189 (10.4.1.189)  1.245 ms  1.243 ms 10.4.1.133 (10.4.1.133)  1.191 ms<br>
> 4  78.152.59.152 (78.152.59.152)  1.032 ms  1.011 ms  1.088 ms<br>
> 5  xe-0-0-3.cr1-chi1.ip4.gtt.net (89.149.141.165)  16.121 ms  16.100 ms  16.076 ms<br>
> 6  ip4.gtt.net (173.205.42.134)  27.873 ms  27.723 ms  27.689 ms<br>
> 7  10.67.0.182 (10.67.0.182)  39.223 ms  38.415 ms  38.368 ms<br>
> 8  * * *<br>
> 9  * * *<br>
> -- Traceroute timed out --<br>
> ```<br><br>
> Here is another:<br><br>
> https://atl-lg.hostus.us/ <br><br>
> ```<br>
> traceroute to 92.119.148.5 (92.119.148.5), 30 hops max, 60 byte packets<br>
> 1  * * *<br>
> 2  unassigned.quadranet.com (104.223.91.255)  0.418 ms  0.410 ms  0.430 ms<br>
> 3  atl-b22-link.telia.net (62.115.183.188)  0.343 ms  0.323 ms  0.337 ms<br>
> 4  atl-b24-link.telia.net (62.115.113.25)  0.425 ms  0.746 ms  0.726 ms<br>
> 5  ae-2.r04.atlnga05.us.bb.gin.ntt.net (129.250.9.21)  0.567 ms  0.582 ms  0.522 ms<br>
> 6  ae-4.r21.atlnga05.us.bb.gin.ntt.net (129.250.5.202)  2.983 ms  0.854 ms  0.812 ms<br>
> 7  ae-6.r23.dllstx09.us.bb.gin.ntt.net (129.250.4.116)  18.913 ms  18.943 ms  21.497 ms<br>
> 8  ae-5.r11.dllstx09.us.bb.gin.ntt.net (129.250.5.24)  21.514 ms  21.490 ms  17.968 ms<br>
> 9  ae-0.a01.dllstx09.us.bb.gin.ntt.net (129.250.3.244)  17.707 ms  21.232 ms  21.258 ms<br>
> 10  ae-0.choopa.dllstx09.us.bb.gin.ntt.net (128.241.219.54)  61.014 ms  60.734 ms  60.696 ms<br>
> 11  * * *<br>
> 12  * * *<br>
> -- Traceroute timed out --<br>
> ```<br><br>
> Here is one from https://ping.pe/ (55 Marietta/RamNode):<br><br>
> ```<br>
> 0.  US 192.184.82.0     Loss%  Snt   Last  Avg   Best  Wrst  StDev AS Name                         PTR<br>
> 1.     ???              100.0  20    0.0   0.0   0.0   0.0   0.0   -<br>
> 2.  US 129.250.203.221  0.0%   20    1.1   1.0   0.3   7.0   1.4   2914  US NTT                    xe-0-0-25-1.a01.atlnga05.us.bb<br>
> 3.  US 129.250.5.57     0.0%   20    19.8  19.8  19.7  20.0  0.1   2914  US NTT                    ae-11.r03.atlnga05.us.bb.gin.n<br>
> 4.  US 129.250.5.211    0.0%   20    0.6   3.0   0.6   24.9  6.9   2914  US NTT                    ae-3.r21.atlnga05.us.bb.gin.nt<br>
> 5.  US 129.250.4.116    0.0%   20    19.4  19.6  19.4  20.2  0.3   2914  US NTT                    ae-6.r23.dllstx09.us.bb.gin.nt<br>
> 6.  US 129.250.5.24     0.0%   20    22.9  23.0  22.9  23.4  0.2   2914  US NTT                    ae-5.r11.dllstx09.us.bb.gin.nt<br>
> 7.  US 129.250.3.30     0.0%   20    22.9  23.9  22.7  35.2  2.7   2914  US NTT                    ae-1.a01.dllstx09.us.bb.gin.nt<br>
> 8.  US 128.241.219.54   15.0%  20    21.1  23.2  21.0  40.5  4.9   2914  US NTT                    ae-0.choopa.dllstx09.us.bb.gin<br>
> 9.     ???              100.0  20    0.0   0.0   0.0   0.0   0.0   -<br>
> 10.    ???              100.0  20    0.0   0.0   0.0   0.0   0.0   -<br>
> 11.    92.119.148.5     0.0%   20    21.4  21.5  21.4  23.8  0.5   20473 US CHOOPA                 server5.us.gflclan.com<br>
> ```<br><br>
> Is GTT & NTT direct peers with Vultr in the Atlanta location? If so, can somebody look into this issue with routing? If not, can somebody give me a list of all direct peers in this location specifically that I can use?<br><br>
> I put in a ticket a couple months ago regarding poor routing and requested a full list of direct peers at each data center Vultr is present in. I was told a list was being compiled, but the ticket was closed after receiving an update stating a list was still being put together. I'd appreciate if we could actually get this list so we can tune each location's BGP communities.<br><br>
> Thank you for your time.

Hopefully Vultr will look into this issue or at least provide a full list of direct peers.

With that said, I've added SWAP to all of our PoPs (thanks for the help, Xy). This will help with crashes Compressor receives when being restarted after some time. I am also looking into switching all of our PoP servers to a new minimal OS which will help with RAM usage, etc. I am also looking into adding one more location.

If you have any questions or concerns, please let me know.

Thanks.

**[Original Source](https://gflclan.com/forums/topic/47317-two-new-pops-atlanta-and-toronto-other-pop-changes/?tab=comments#comment-226696)**

## Update
**Updated on October 18th, 2019**

I've removed the Atlanta PoP since GTT & NTT aren't direct peers there. I tried it with Telia, but it messed up the routing in many locations (e.g. the West Coast was routing to the Atlanta PoP instead of the West Coast PoPs). I've tried prepending 1 - 3 hops as well to the Atlanta PoP, but it didn't make much of a difference.

Thanks.

**[Original Source](https://gflclan.com/forums/topic/47317-two-new-pops-atlanta-and-toronto-other-pop-changes/?tab=comments#comment-227258)**