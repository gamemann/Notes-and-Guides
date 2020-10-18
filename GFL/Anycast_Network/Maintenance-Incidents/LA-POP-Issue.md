# LA POP Issue
**Created on June 17th, 2019**

I'm making this thread for documentation purposes.

Tonight at around **7:30 PM CST**, our LA POP ran into an issue where it stopped routing to our game server machines with one of our primary hosting providers. The route wasn't leaving the DC which was odd. Since ICMP replies were disabled on the LA POP, I couldn't confirm this until I stopped announcing the POP to the network and disabled Compressor. After doing this, I confirmed there was a routing issue and the players who were affected before were able to connect to the servers again.

As of **8:15 PM CST**, the POP is able to route to the game server machines again (confirmed via MTR). However, I still haven't announced it to the network again since I want to monitor and ensure it doesn't go down in the next hour or so.

Thanks!

**[Original Source](https://gflclan.com/forums/topic/57918-6-17-20-la-pop-issue/?do=findComment&comment=262120)**