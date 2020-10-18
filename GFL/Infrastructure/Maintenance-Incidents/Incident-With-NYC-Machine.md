# Incident With NYC Machine
**Created on June 20th, 2020 at 12:07 PM CST**

Hey everyone,

I'm writing this thread for documentation purposes.

Yesterday, our game server(s) on our NYC machine (e.g. CS:GO Surf Timer #4) stopped receiving traffic temporarily. This is because our hosting provider appeared to have added ACLs back to prevent us from spoofing as our Anycast network (92.119.148.0/24). I made a ticket months ago to have them remove these ACLs. Unfortunately, they were added back.

I've disabled the IPIP Direct [program](https://github.com/gamemann/IPIPDirect-TC) I made so the outbound traffic is now going through the NYC POP (this allowed the servers to come back up at least). I've created another ticket with our hosting provider asking them to look into this and find out why these ACLs were added back.

Once I have an update, I will let you know.

Thank you.

**[Original Source](https://gflclan.com/forums/topic/58022-6-19-20incident-with-nyc-machine/?do=findComment&comment=262465)**

## Update
**Updated on June 20th, 2020 at 12:14 PM CST**

Received the following from our hosting provider:

> I do believe we put functionality in place to stop spoofing due to some local network attacks that were being emitted. I'll put this in queue for Matt to review and see if he can allow it for your IP range.

Once I have another update, I'll let everybody know.

Thanks.

**[Original Source](https://gflclan.com/forums/topic/58022-6-19-20incident-with-nyc-machine/?do=findComment&comment=262466)**

## Update
**Updated on June 20th, 2020 at 6:23 PM CST**

Our hosting provider removed the filters that were preventing us from sending out as our Anycast network. The IPIP Direct program is enabled on the NYC machine again and working properly :)

Thanks!

**[Original Source](https://gflclan.com/forums/topic/58022-6-19-20incident-with-nyc-machine/?do=findComment&comment=262494)**