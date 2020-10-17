# "500 No space left on device" On Ubuntu 16.04 LTS Server Under Sophos UTM 9
**Created on January 31st, 2018**

Hello, I am writing this guide in hopes it’ll help others who come across this strange and complicated issue.

## The Problem
Two of the VMs I have under [Sophos](https://www.sophos.com/en-us.aspx) (UTM 9) returns a “500  No space left on device” error while trying to run `apt-get update` even though the device is not out of space. Updating the packages takes a very long time to complete and eventually just returns that error. Here were the results from my latest attempt at executing `apt-get update`:

```
cdeacon@ubuntu:~$ sudo apt-get update
Hit:1 http://us.archive.ubuntu.com/ubuntu xenial InRelease
Get:2 http://us.archive.ubuntu.com/ubuntu xenial-updates InRelease [102 kB]
Get:3 http://us.archive.ubuntu.com/ubuntu xenial-backports InRelease [102 kB]
Get:4 http://security.ubuntu.com/ubuntu xenial-security InRelease [102 kB]
Ign:5 http://us.archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages
Ign:6 http://security.ubuntu.com/ubuntu xenial-security/main amd64 Packages
Ign:7 http://us.archive.ubuntu.com/ubuntu xenial-updates/main i386 Packages
Ign:8 http://security.ubuntu.com/ubuntu xenial-security/main i386 Packages
Ign:9 http://us.archive.ubuntu.com/ubuntu xenial-updates/main Translation-en
Get:10 http://security.ubuntu.com/ubuntu xenial-security/main Translation-en [189 kB]
Get:11 http://security.ubuntu.com/ubuntu xenial-security/universe amd64 Packages [200 kB]
Ign:12 http://security.ubuntu.com/ubuntu xenial-security/universe i386 Packages
Ign:13 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages
Ign:6 http://security.ubuntu.com/ubuntu xenial-security/main amd64 Packages
Ign:14 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe i386 Packages
Ign:8 http://security.ubuntu.com/ubuntu xenial-security/main i386 Packages
Ign:15 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe Translation-en
Get:12 http://security.ubuntu.com/ubuntu xenial-security/universe i386 Packages [162 kB]
Ign:6 http://security.ubuntu.com/ubuntu xenial-security/main amd64 Packages
Ign:5 http://us.archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages
Ign:8 http://security.ubuntu.com/ubuntu xenial-security/main i386 Packages
Ign:7 http://us.archive.ubuntu.com/ubuntu xenial-updates/main i386 Packages
Err:6 http://security.ubuntu.com/ubuntu xenial-security/main amd64 Packages
  500  No space left on device [IP: 91.189.88.162 80]
Ign:8 http://security.ubuntu.com/ubuntu xenial-security/main i386 Packages
Ign:9 http://us.archive.ubuntu.com/ubuntu xenial-updates/main Translation-en
Ign:13 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages
Ign:14 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe i386 Packages
Ign:15 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe Translation-en
Ign:5 http://us.archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages
Ign:7 http://us.archive.ubuntu.com/ubuntu xenial-updates/main i386 Packages
Ign:9 http://us.archive.ubuntu.com/ubuntu xenial-updates/main Translation-en
Ign:13 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages
Ign:14 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe i386 Packages
Ign:15 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe Translation-en
Err:5 http://us.archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages
  500  No space left on device [IP: 91.189.91.23 80]
Ign:7 http://us.archive.ubuntu.com/ubuntu xenial-updates/main i386 Packages
Ign:9 http://us.archive.ubuntu.com/ubuntu xenial-updates/main Translation-en
Ign:13 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages
Ign:14 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe i386 Packages
Ign:15 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe Translation-en
Fetched 858 kB in 19min 19s (739 B/s)
Reading package lists... Done
E: Failed to fetch http://us.archive.ubuntu.com/ubuntu/dists/xenial-updates/main/binary-amd64/Packages  500  No space left on device [IP: 91.189.91.23 80]
E: Failed to fetch http://security.ubuntu.com/ubuntu/dists/xenial-security/main/binary-amd64/Packages  500  No space left on device [IP: 91.189.88.162 80]
E: Some index files failed to download. They have been ignored, or old ones used instead.
```

## My Setup
I have three Ubuntu 16.04 LTS VMs (same images) running under a Sophos UTM 9 machine. I used all of these VMs as web servers.

The two VMs experiencing issues runs under the default internal network for Sophos (in this case, 192.168.12.0/24). The other VM runs under an additional interface added to Sophos (in this case, 192.168.13.0/24).

## Results From The VMs Experiencing Issues
When running into an error that contains “no space left on device”, the first thing you would typically look at is the amount of space used on the machine. Here’s an output from `df -h` on one of the VMs experiencing issues:

```
cdeacon@ubuntu:~$ sudo df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            977M     0  977M   0% /dev
tmpfs           200M  6.9M  193M   4% /run
/dev/vda1        16G  1.8G   14G  12% /
tmpfs           996M     0  996M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           996M     0  996M   0% /sys/fs/cgroup
tmpfs           200M     0  200M   0% /run/user/1000
```

None of the filesystems were even close to using 100%. While running `apt-get update`, I ran `watch --interval=0 df -h` to see if it somehow temporarily filled up, but it never did.

After investigating further, there was a possibility I could be out of inodes. I ran the command `df -i` to check and here were the results:

```
cdeacon@ubuntu:~$ sudo df -i
Filesystem      Inodes IUsed  IFree IUse% Mounted on
udev            249860   397 249463    1% /dev
tmpfs           254881  1339 253542    1% /run
/dev/vda1      1048576 98280 950296   10% /
tmpfs           254881     1 254880    1% /dev/shm
tmpfs           254881     3 254878    1% /run/lock
tmpfs           254881    16 254865    1% /sys/fs/cgroup
tmpfs           254881     4 254877    1% /run/user/1000
```

Once again, none of the filesystems were even close to 100%.

At this point, I was very confused. With that said, I could download large files (~200 MBs) from my external server fine on this VM. The only issue I was having was running anything with `apt`.

Finally, I found [this](https://raspberrypi.stackexchange.com/questions/71635/apt-get-update-error-500-no-space-left-on-device/71697) Stack Exchange question that appeared to be running into similar issues. In the thread, the answer states that this is an issue with Sophos. I decided to confirm if this was true by adding an additional NIC to the machine that didn’t run through Sophos. Surprisingly, the public NIC could download files fine.

I basically used the `wget` command to download the file listed in the Stack Exchange thread “http://mirror.us.leaseweb.net/raspbian/raspbian/dists/stretch/main/debian-installer/binary-armhf/Packages”. I used `--bind-address` to download under a specific address/device (e.g. `eth0` or `eth1`).

Here are the results from the Sophos NIC:

```
cdeacon@ubuntu:~$ sudo wget --bind-address=192.168.12.5 http://mirror.us.leaseweb.net/raspbian/raspbian/dists/stretch/main/debian-installer/binary-armhf/Packages
--2018-01-31 15:16:32--  http://mirror.us.leaseweb.net/raspbian/raspbian/dists/stretch/main/debian-installer/binary-armhf/Packages
Resolving mirror.us.leaseweb.net (mirror.us.leaseweb.net)... 207.244.94.80, 2604:9a00:2010:a0b8::5
Connecting to mirror.us.leaseweb.net (mirror.us.leaseweb.net)|207.244.94.80|:80... connected.
HTTP request sent, awaiting response... 500 No space left on device
2018-01-31 15:16:32 ERROR 500: No space left on device.
```

Here are the results from the temporary public NIC:

```
cdeacon@ubuntu:~$ sudo wget --bind-address=192.206.141.95 http://mirror.us.leaseweb.net/raspbian/raspbian/dists/stretch/main/debian-installer/binary-armhf/Packages
--2018-01-31 15:19:39--  http://mirror.us.leaseweb.net/raspbian/raspbian/dists/stretch/main/debian-installer/binary-armhf/Packages
Resolving mirror.us.leaseweb.net (mirror.us.leaseweb.net)... 207.244.94.80, 2604:9a00:2010:a0b8::5
Connecting to mirror.us.leaseweb.net (mirror.us.leaseweb.net)|207.244.94.80|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 384152 (375K) [application/octet-stream]
Saving to: 'Packages.2'

Packages.2                    100%[===============================================>] 375.15K  --.-KB/s    in 0.04s

2018-01-31 15:19:40 (8.57 MB/s) - 'Packages.2' saved [384152/384152]
```

This confirmed it was Sophos causing the issue. But why? What also didn’t make sense is the other VM under a different network for Sophos didn’t run into this issue. It seemed to only affect VMs under the default internal network for Sophos in my case. I’m still not sure why this issue only affected the default internal network (192.168.12.0/24).

I ran the `df -h` command on the Sophos machine and this was the result:

```
cdeacon:/root # df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda6       5.2G  3.4G  1.6G  69% /
udev           1002M   84K 1001M   1% /dev
tmpfs          1002M     0 1002M   0% /dev/shm
/dev/vda1       331M   16M  295M   5% /boot
/dev/vda5       703M  688M     0 100% /var/storage
/dev/vda7       949M   40M  841M   5% /var/log
/dev/vda8       495M  6.3M  453M   2% /tmp
tmpfs          1002M     0 1002M   0% /var/sec/chroot-httpd/dev/shm
tmpfs          1002M     0 1002M   0% /var/storage/chroot-reverseproxy/dev/shm
tmpfs          1002M  4.0K 1002M   1% /var/storage/chroot-smtp/tmp/ram
tmpfs          1002M     0 1002M   0% /var/storage/chroot-http/tmp
```

The `/dev/vda5` filesystem for `/var/storage` was completely filled. This was indeed the issue. It somewhat makes sense but I'm not sure why it was filling up.

## The Solution
The problem for me was that `/var/storage` was completely filled up. I haven’t found anything useful on Google related to this issue (mostly just useless threads from 2005). What I did to fix this issue temporarily was remove some useless files located in `/var/storage/core` and everything started working fine with the VMs that experienced issues. Some threads recommended clearing out the HTTP proxy cache and email spool. You can find out what’s taking up space by executing the following command in the `/var/storage` directory:

```
du -sh *
```

Additionally, you could add more space to the partition with the `/var/storage` mount point (in my case, `vda5`). I also tweaked the log settings to delete logs after three days. Though, I’m not sure if any log files would be stored in `/var/storage`.

## Conclusion
This is a very strange issue and can be complicated if you haven’t pinned it down being Sophos yet. While this solution isn’t a permanent fix yet (I am on the lookout for a permanent fix and when I find one, I will update this guide), it should set you on the right path.

If you found a permanent fix or a reason why it would only occur on one internal network, please feel free to post a comment so I can update the guide (you will be credited as well).

This guide was written by Christian Deacon.

Thank you for reading!

**[Original Source](https://gflclan.com/guides/500-no-space-left-on-device-on-ubuntu-1604-lts-server-under-sophos-utm-9-r7/)**