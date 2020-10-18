# Preparation And Plan For New Filter Rules On Anycast Network
**Created on June 7th, 2020**

Hey everyone,

As some of you may know, I've been working to hard-code filters into our current packet processing software ([Compressor V1](https://github.com/Dreae/compressor)). This is a temporary solution until Compressor V2 is completed and its main purpose is to mitigate recent (D)DoS attacks aimed towards our network. You can read more about this [here](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Anycast_Network/Notes/Anycast-Filtering-Notes-DDoS-Attacks-Filtering-And-More.MD) which explains why I'm doing this.

So far, these new filters are deployed on our Sydney POP and after correcting a couple [issues](https://github.com/gamemann/Notes-and-Guides/blob/master/GFL/Anycast_Network/Maintenance-Incidents/Tracking-Down-Issues-With-Sydney-POP.md), it appears game server traffic is flowing through this POP smoothly (I haven't heard of any complaints in five or so days). Therefore, I believe we're ready to expand these filters to POPs in other locations.

This morning, I setup an application I made recently to handle all the services we need whitelisted. Each POP will grab the whitelisted services from this back-bone every x seconds. I plan on giving Technical Administrators and Directors access to modify these service lists along with appropriate training so I'm not the only one making these changes since it will be a burden.

I do want to prepare everybody for the deployment. Please read below.

## Deployment Plan
The deployment plan is pretty simple. I'm going to add these filters to the rest of our Asia POPs (Tokyo and Singapore) along with giving it one - two days to ensure there are no reported issues. Afterwards, I will be deploying these new filters to our Europe POPs (Frankfurt, Paris, London, and Amsterdam). Finally, we'll start deploying it to North America after the Europe POPs are proven stable.

I may give it an additional one - two days to try to capture more outbound traffic and make adjustments to our service whitelists. With that said, I need to figure out how to support Rust+ with the new filters since that's important for Rust servers now, I guess. This will likely add some delays.

## Things May Break
Since we're whitelisting all outbound connections/services made by the game server, there's a high chance we've missed some things initially. I have all global/GFL-specific services whitelisted. However, there's a chance the game server is making outbound connections to a specific service in that game or game mode that isn't on the whitelist yet. If you notice something in the server breaking, please report it to the Server Manager and they'll go up the chain if it's related to networking.

I'm doing the best I can to monitor outbound game server traffic and whitelist the correct services. However, monitoring outbound traffic from 20+ game servers is very time-consuming and there's a high chance I've missed some things.

That's basically it. I just wanted to thank you for your understanding and patience regarding this matter. I understand some of you may be frustrated if a specific service breaks. However, this is the only way we can fully secure our network.

If you have any questions, please feel free to reply.

Thank you.

**[Original Source](https://gflclan.com/forums/topic/57525-preparation-and-plan-for-new-filter-rules-on-anycast-network/?do=findComment&comment=260612)**

## Update
**Updated on June 10th, 2020**

The last couple of days I've been trying to whitelist the Rust+ service. Unfortunately, this was harder than I expected since it uses the client's source IP to establish the TCP connection. However, I believe I got it pretty much working.

With that said, I'm setting up scripts to automatically update our whitelisted services and this is nearly finished.

I've updated the Sydney's POP with the up-to-date filters and just want to make sure things run stable for a day or so. Afterwards, we will start expanding the filters to new POPs.

Thank you.

**[Original Source](https://gflclan.com/forums/topic/57525-preparation-and-plan-for-new-filter-rules-on-anycast-network/?do=findComment&comment=261111)**