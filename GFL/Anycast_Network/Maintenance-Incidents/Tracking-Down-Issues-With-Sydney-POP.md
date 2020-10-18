# Tracking Down One More Issue With New Filters On Sydney POP
**Created on June 3rd, 2020 at 9:02 PM CST**

Hey everyone,

As some of you may know, our Sydney POP has new filters I've applied to the packet processing software we're using. I've hard-coded these filters into the software as a temporary solution to mitigate (D)DoS attacks we've been receiving recently. Dreae and I are working on Compressor V2 (or whatever name we go with) as a permanent solution. However, this will take time to create.

The past few days I've been monitoring traffic on the Sydney POP and there have been a few issues that I've mostly fixed. However, I'm currently trying to track down one last issue. The side effects of this issue appear to be when you connect to the server, you'll time out around 20 or so seconds after the initial connection (the validation map timeout). I received a report regarding this and tried connecting using the forwarding server I have in Sydney and actually witnessed it myself. Unfortunately, I didn't have logging enabled at the time and no matter what I do, I can't reproduce the issue this time around after enabling logging for validations and handshakes. I've been monitoring the log all day and haven't seen many much occurrences of this since this rarely happens.

I do have a suspicion on what the issue might be. However, it appears to be a bug with BPF or something because it's reporting the hits member (indicating how many packets the client has sent during validation period) on the validation map is 0 even though there is never a time it should be set to 0.

I'm making this thread to make users aware of this issue if they're routing through the Sydney POP (anybody closer to Sydney, AUS more than likely). You can confirm you route through this POP by performing a trace route or MTR to any IP on the Anycast network (92.119.148.0/24).

If you experience the side effects from this issue, please post on this thread with the time it happened and private message me your public IP address (you can get it from [here](http://ipcurl.com/)).

This issue is the only thing preventing me from deploying these filters to more POPs at the moment. Unfortunately, the only work-around for the players right now is to keep reconnecting until you puts you on the handshake map. This usually works after the first or second retry, at least for me.

Thank you.

**[Original Source](https://gflclan.com/forums/topic/57370-tracking-down-one-more-issue-with-new-filters-on-sydney-pop/?do=findComment&comment=260037)**

## Update
**Updated on June 3rd, 2020 at 10:02 PM CST**

After talking to [Dreae](https://github.com/dreae) and reviewing the XDP program's code, Dreae told me the validation map being a LRU per CPU map would cause issues when reading values from the map in the XDP program itself since that is unsupported at the moment (you can only read per CPU maps within the user space like I am doing with the handshake LRU per CPU map). Therefore, I've converted the validation map to a single map and used a different function Dreae told me about to update the hits count. This was most likely the issue and why the hits count was 0 (it was **reading** the value from the wrong CPU). Since the hits and expiration time values were at 0, this was most likely causing the timeout issues after 20 seconds of loading in or so. I used per CPU maps for performance advantages. However, the validation map doesn't perform many reads/updates. Therefore, converting this map to a single map wouldn't be a big deal in my opinion.

The whitelist map will remain a LRU per CPU map for performance advantages (this is the map that needs to be spread out to multiple CPUs anyways). We only modify values on this map within the XDP program and read values within the user space which is fine.

I've applied this update to the Sydney POP. I will continue to monitor the logs to ensure I see nothing else suspicious. This issue should be **resolved** and thank Dreae for all the help :)

**[Original Source](https://gflclan.com/forums/topic/57370-tracking-down-one-more-issue-with-new-filters-on-sydney-pop/?do=findComment&comment=260041)**