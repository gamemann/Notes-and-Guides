# Issue With Rust Players Timing Out
**Created on July 12th, 2020**

Hi,

I'm making this thread for documentation purposes.

Last night when setting up the new game server machine, I decided to add new IP allocations and sync the POP configs (which included the latest config from POPs running the new filters). The new filters config would work with POPs not running the new filters. However, the one thing I forgot about is POPs running the old version didn't have the Rust rate limit multiplier which was needed (since we had a relatively low standard rate limit which worked fine for Source Engine servers, but not for Rust servers).

This resulted in players easily being rate limited and causing timeouts when routing to POPs not running the new filters.

I've pushed a new update to all POPs raising the standard rate limit for the time being. When all POPs are running the new filters, I will be lowering the standard rate limit, but Rust servers will have a multiplier of four which will be fine.

This issue should be resolved. I haven't received any complaints since.

I just wanted to apologize for the inconvenience. It was something I overlooked and since I was in a rush to try to get the new machine running and tested last night (along with Xy), I didn't catch it while updating the POPs last night.

Thank you for understanding.

**[Original Source](https://gflclan.com/forums/topic/59144-issue-with-rust-players-timing-out/?do=findComment&comment=265966)**