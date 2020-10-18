# Added New POP To Our Anycast Network
**Created on May 20th, 2020**

Hey everyone,

I just wanted to let everyone know I've added a new POP to the Anycast network in Seoul, South Korea. I've also made a pull request to Compressor V1 that adds support for XDP-native [here](https://github.com/Dreae/compressor/commit/517eb8486bf5ebc1d66ef8d809ed700bed62aa9e) and the POP is currently running with XDP-native. The added POP and XDP-native support should add more capacity to our network to handle (D)DoS attacks, etc. With that said, I plan on adding filtering rules once I figure out which traffic I want to whitelist on the network.

[Dreae](https://github.com/dreae) and I are still working on [Compressor V2](https://gitlab.com/srcds-compressor/compressor/-/issues) and I recently completed the FOU Wrapping/Unwrapping TC BPF programs [here](https://github.com/gamemann/Compressor-V2-FOU-Wrap-Unwrapper) which were needed for outbound server traffic and reporting to the Steam Master Server. We're making great progress so far :) Once this is finished, we'll be highly expanding our network.

If you experience any issues, please let me know!

Thanks!

**[Original Source](https://gflclan.com/forums/topic/56791-added-new-pop-to-our-anycast-network/?do=findComment&comment=257682)**

## Update
**Updated on May 22nd, 2020**

It turns out XDP-native breaks the AF_XDP side of Compressor V1 (A2S_INFO caching in our case). I made a thread on the XDP Newbies mailing list with the following:

![1](https://g.gflclan.com/2765-05-22-2020-sFYIDgL8.png)

I made a test AF_XDP project [here](https://github.com/gamemann/AF_XDP-Test) to see if it made any difference using the latest recommended AF_XDP code from [XDP-Tutorial](https://github.com/xdp-project/xdp-tutorial/tree/master/advanced03-AF_XDP) (Compressor V1 uses outdated AF_XDP code and an older version of LibBPF). This is the biggest issue as of right now, but even when Compressor V1 does have RX queue #1 loaded with AF_XDP, the program isn't sending back A2S_INFO packets to the client. I know it's receiving traffic on the socket from the XDP redirect map function. I think the current AF_XDP functions used to send data back out are outdated. Therefore, I will work to update this.

I've been working hard on this for the last day or so and hope I can get this resolved by the end of this weekend (assuming this isn't an issue with Vultr's NIC drivers).

Thanks!

**[Original Source](https://gflclan.com/forums/topic/56791-added-new-pop-to-our-anycast-network/?do=findComment&comment=257923)**