# Network Update (New NYC PoP Fixed)
**Created on January 19th, 2020**

Hey everyone,

I just wanted to provide an update on our Anycast network. Around two weeks ago, I was [talking](https://gflclan.com/forums/topic/50581-anycast-upgrades-another-dedicated-server-for-our-game-servers/) about how we're getting a new hosting provider to replace our NYC PoP. I had everything setup at one point two weeks ago and started announcing the new PoP to the network. However, unfortunately, there was a very strange issue occurring when Compressor tried to send out and receive [A2S_INFO](https://developer.valvesoftware.com/wiki/Server_queries#A2S_INFO) packets. This forced me to stop announcing the new PoP to the network and to switch back to the old PoP until I fixed the issue.

I've been debugging this issue for nearly two weeks now (hours each day) while initially thinking it was an issue with how IPs were routed to the server from our new hosting provider. This issue was pretty complicated, especially given I didn't have advanced knowledge with C and working with XDP + AF_XDP (what Compressor uses to process packets). After talking to Support for a day or so, I found out this was probably not with the IPs routing to the server and instead, had to do with Compressor not listening on all RX queues due to the way the NIC is designed. Up until yesterday I thought this was an issue with the IP routing. Today, I've been doing a lot of debugging and found out how to make Compressor listen on all RX queues and send the necessary traffic to the XDP user space properly (AF_XDP). With that said, since the machine was hyper-threaded and there were only 4 RX queues available, it was set to listen on 8 RX queues which resulted in bind errors. Therefore, I had to change the amount of XSK sockets Compressor created to match the amount of RX queues available.

Although this was a very time-consuming process and I will not lie, I've been extremely exhausted/burnt out recently due to it, I ended up learning more about TX/RX queues, XDP + AF_XDP, C, NIC hardware/drivers, and more to finally fix this issue tonight. I am happy I am learning about all of this as well since this is an area I'm very interested in!

I haven't started announcing the new PoP to the network yet since I don't want to interrupt our servers during peak times (in the case that for whatever reason it doesn't work for Anycast traffic, though this is highly unlikely). I will more than likely try putting the PoP up later tonight when things calm down to see if it works.

I just wanted to thank everyone for their patience in regards to this. Once we have this new PoP in-place, the lag on servers like Rust Modded should stop. With that said, this will allow us to move some servers off of our GS09 machine (which has been overloaded recently) to one of our NYC machines. This issue has been preventing that from moving along, but since that's fixed, we're good to go.

I will also be making a pull request to [Compressor](https://gitlab.com/Dreae/compressor/) with the changes I've implemented that allows Compressor to listen on all RX queues.

Thank you for your time.

**[Original Source](https://gflclan.com/forums/topic/51177-network-update-new-nyc-pop-fixed/?do=findComment&comment=238473)**

## Update
**Updated on January 20th, 2020**

Hey everyone,

The new NYC PoP is now active and everything appears to be working fine! The NYC machines and my VMs are routing to the new PoP without any issues from what I can tell.

If you experience any issues, please let me know!

Thanks.

**[Original Source](https://gflclan.com/forums/topic/51177-network-update-new-nyc-pop-fixed/?do=findComment&comment=238596)**