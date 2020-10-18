# Routing Issues For Players In Asia From Two Days Ago
**Created on October 11th, 2020**

Hey everyone,

I'm creating this thread for documentation purposes and this is mostly a copy and paste from the announcement I made on our Discord servers two days ago.

Recently, our newer hosting provider, GSK, started peering with Hurricane Electric. This resulted in many routes from players within our Asia region to route to our GSK POP through Hurricane Electric. This is bad because they should be routing through our POPs in Asia instead. With that said, it seems this tripped an uRPF filter on GSK's end which resulted in the player's IPs being blocked on GSK's behalf (since it's a really strange route going from Asia to GSK through Hurricane Electric). If one of our game servers got (D)DoS attacked, GSK would enable GTT offload which results in our traffic going through GSK's network for (D)DoS mitigation (for the servers that are attacked, single /32 IPs). Therefore, this would block a lot of our players who tripped the uRPF filter. Any game servers on our GSK machines would also block these players since we send traffic back to the client directly from these machines via GSK's network.

I've talked to GSK and as a quick solution, they created a BGP community that withdraws Hurricane Electric from our routes which I've applied two days ago. This will allow the affected players to route to our servers again assuming GTT offload isn't enabled and the game server isn't on one of our GSK machines. We've unblocked the player IPs who were blocked that we were aware of. We're working with GSK and looking into a better long-term solution for this issue.

If you experience any issues connecting to our game servers, please reach out to Liloz01, Aurora, or I since we all have a group DM regarding this with GSK.

I do apologize for the inconvenience and if you have any questions, please let me know.

Thank you for your time.

**[Original Source](https://gflclan.com/forums/topic/64230-routing-issues-for-players-in-asia-from-two-days-ago/?do=findComment&comment=310317)**