# New POP For NYC
**Created on December 30th, 2019**

Hey everyone,

Just an FYI that I've rebuilt our NYC PoP on our Anycast network. I did this because it appears the current/old PoP has something wrong that isn't allowing our packet processing software (Compressor) to use more than one thread. This was causing performance issues for clients routing to the NYC PoP along with servers located in NYC (including our Rust Modded server, which saw very bad performance at higher player counts due to this).

I made the new PoP with a different kernel + OS.

I will soon be shutting down the old PoP (which will stop BGP, etc). You may see a few lag spikes during this time as the route updates to the newer NYC PoP.

If you have any questions, please let me know.

Thank you for understanding.

**[Original Source](https://gflclan.com/forums/topic/50198-new-pop-for-nyc/?do=findComment&comment=235645)**

## Update
**Updated on January 6th, 2020**

I apologize for the delay on this. I finally shut off the old PoP that the game server machine was routing to last night because we were seeing issues with the route (packet loss). However, the new PoP didn't resolve this issue. It appears Compressor is indeed using more than one core/thread on the new PoP server. Therefore, I believe this may be an issue with our hosting provider instead.

I will be making a thread soon with a plan to resolve this issue + upgrade the Anycast network overall.

Thank you.

**[Original Source](https://gflclan.com/forums/topic/50198-new-pop-for-nyc/?do=findComment&comment=236801)**