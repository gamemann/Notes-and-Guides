# Read If You're Experiencing "Lag" Recently On Our Game Servers (High Ping or Packet Loss)
**Created on March 28th, 2020**

Hey everyone,

I've been seeing a high number of reports recently regarding users experiencing lag on our game servers. If you see that your ping is higher than normal or you're receiving packet loss, this thread is for you.

If you're seeing these symptoms, please perform a trace route or an MTR (recommended) to the server IP you're playing on.

On Windows, you can perform a trace route by executing the following command in Windows Command Prompt:

```
tracert <server IP>
```

You will need to replace `<server IP>` with the server's IP address you're playing on.

For an MTR, you may download a tool such as [WinMTR](https://sourceforge.net/projects/winmtr/) to achieve this. You will have to fill out the text field at the top of the program with the server IP's address and click start. Please allow the program to send 100 - 200 requests and try to do this while you're experiencing high latency/packet loss on the server. An MTR is **A LOT** better than a trace route in this case since it shows packet loss at each hop along with continuously sends requests to the destination. Therefore, I **HIGHLY** recommend using MTR over trace route for this.

For Linux, you can just install the needed packages for the `traceroute` and `mtr` commands (probably `net-tools`).

Once you have these results, please PM me them.

I've gotten results from one individual (Lurn) from CS:GO Arena so far. According to their results, one of our direct peers (NTT) is experiencing issues and I'm suspecting this is due to the recent traffic spike caused by the coronavirus lockdown. I've been seeing similar issues at my job recently, especially with Microsoft, etc. I'm suspecting other users to be experiencing this same issue and I just want to confirm this. If I am able to retrieve a direct peers list from our POP hosting provider, I may be able to take off NTT from our direct peers list temporarily. However, our POP hosting provider still hasn't given me that list after asking for it three - four times now.

If you're experiencing performance issues on our Rust servers, this may be caused by the issue I explain at the end of [this](https://gflclan.com/forums/topic/53600-tech-update-new-web-machine-what-im-working-on-and-more-3-16-20/) thread. I am working to find a fix for this and feel free to read my latest status [comment](https://gflclan.com/profile/1-roy/?status=11502&type=status) for an update on that.

Thank you for your time.

**[Original Source](https://gflclan.com/forums/topic/64374-two-network-incidents-from-october-12th/?tab=comments#comment-310847)**