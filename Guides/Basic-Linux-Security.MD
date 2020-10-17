# Basic Linux Security 
**Created on March 3rd, 2019**
Hello,

In this guide, I will be going over how to secure a stock Linux machine. With that said, these are methods I use on Ubuntu/Debian-based systems. Therefore, some commands may need to be altered depending on the Linux distro you’re using.

## Keep Your System Up-To-Date
When logging into the server for the first time, ensure to keep the system up-to-date with kernel updates, etc. Try to keep your machines up-to-date as much as possible but before upgrading, ensure it isn’t during a live period because it will require a restart more than likely. With that said, having a recovery option ready is recommended just in-case it locks you out of the system somehow after a reboot.

You can run the following commands on Ubuntu/Debian as root to update and upgrade the system:

```
apt-get update
apt-get upgrade
apt-get dist-upgrade
```

## Disable Root Login
When you’ve made your own user with root and gave it sudo access (you can add sudo users by running `visudo` as root in the terminal), I’d strongly suggest disabling the root login in general. You can do so by editing `/etc/ssh/sshd_config` and setting **PermitRootLogin** to **no**.

Afterwards, ensure to restart the SSH server by running the following command as root:

```
systemctl restart sshd
```

## Change SSH Port
Although hackers can scan your IP for open ports, changing your SSH port usually adds an extra step for a hacker. Changing the SSH’s port is easy to do. Edit the `/etc/ssh/sshd_config` file and set **Port** to whatever you want the SSH port to be (e.g. 4444).

Afterwards, ensure to restart the SSH server by running the following command as root:

```
systemctl restart sshd
```

Please remember when you next SSH into the server, you will need to specify the new port when connecting. You can do this by specifying the `-p` flag followed by the port you want to use to connect. For example:

```
ssh christian@myhost.internal -p <portNum>
```

## Use SSH Keys
SSH keys are easy to generate and generally more secure. They also allow you to log into multiple servers easier if you’re using one key to log into all servers. For example, you’d only need to know the key’s passphrase to log into each server.

By default, RSA SSH keys are 2048 bits. In my opinion, 2048-bit RSA keys are not strong enough. Therefore, if you want to increase the bits, you can specify the `-b` flag and follow it with the number of bits you want the key to be. Personally, I’d suggest generating SSH keys with 4096 bits.

You can generate an SSH key using RSA with the following:

```
ssh-keygen -t rsa -C “Just a comment.”
```

If you want to generate an SSH key with 4096 bits, you can use the following:

```
ssh-keygen -t rsa -b 4096 -C “Just a securer key.”
```

In recent years, many have started recommending using `ed25519` keys. I personally recommend this method. You cannot use the `-b` flag with this. However, I've heard these are typically more secure than RSA keys anyways. If you want to create an SSH key using `ed25519`, please use the following:

```
ssh-keygen -t ed25519 -C "Just a comment."
```

After generating the key on your local terminal (or the server you’re using to connect to your remote server), you will want to copy the contents of the public key (most likely with `cat ~/.ssh/id_rsa.pub`). On the remote server, you’ll want to create a directory named `.ssh/` in the user’s home directory. Give this directory a permission of 700 (only user has read, write, and execute). Afterwards, create a file named `authorized_keys` in the `.ssh/` directory. Paste the contents of the public SSH key into this file. You can add multiple SSH keys to this file (one per line). Afterwards, give the file a permission of 600 (only user has read and write access). Ensure both the directory and file is owned by the appropriate user.

Here are the commands I normally use to create the `.ssh/` directory and `authorized_keys` file and assign them the appropriate permissions along with ownership.

```
mkdir .ssh
touch .ssh/authorized_keys
chmod 700 .ssh/
chmod 600 .ssh/authorized_keys
chown -R user:usergroup .ssh/
```

Finally, I recommend editing the `/etc/ssh/sshd_config` file and enabling public key authentication through there. In most systems, I believe it’s enabled by default but I like uncommenting out the lines anyways. Here’s an example:

```
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
```

Afterwards, restart the SSH server by executing the following command as root:

```
systemctl restart sshd
```

Try logging into the server. If it still prompts you for a password instead of using the SSH key authentication, I’d recommend adding the `-v` flag to the SSH command to get a verbose output and see what the issue is. For example:

```
ssh -v christian@myhost.internal
```

Additionally, you can use multiple private keys on one local terminal. However, you’ll have to specify which key to use when either connecting with the SSH command or with a config file. In the SSH command, you can specify the `-i` flag and follow it with the path to the private key you want to use. For example:

```
ssh -i /home/christian/.ssh/secondkey christian@myhost.internal
```

Or you can specify which key to use for certain IPs/domains in a config file. Make a config file in your local terminal’s `.ssh/` directory (`~/.ssh/config`). Afterwards, you can use the following format to specify a certain private key per host:

```
Host <IP/Domain>
    IdentityFile /path/to/keyfile
```

For example:

```
Host myhost.internal
    IdentityFile /home/christian/.ssh/secondkey
```

## Disable Password Authentication
In my opinion, if you’re using SSH keys, you should just disable password authentication overall. You can do this by editing the `/etc/ssh/sshd_config` file and changing **PasswordAuthentication** to **no**. Please ensure your SSH key authentication works first (preferably with sudo access to at least one user) before doing this. If not, you may get locked out if your SSH key doesn’t work and you’ll have to boot the machine into recovery mode to enable password authentication again or fix your SSH key issue unless if you have KVM access, of course.

Afterwards, restart the SSH server by executing the following command as root:

```
systemctl restart sshd
```

If disabling password authentication overall is not an option (e.g. if you need at least one user to connect to the server via SSH and password authentication), just ensure to delete passwords on users not needed. I would highly suggest doing so to at least users with sudo access. You can delete user’s passwords by executing the following command as root.

```
passwd -d <user>
```

## Allowed Users
I would also suggest only allowing certain users to login via SSH. You can do this by editing the `/etc/ssh/sshd_config` file and specifying the **AllowUsers** config option followed by a list of users allowed to login (one user per space). For example, **User1 User2 User3** and so on. Full example here:

```
AllowUsers “christian user1 user3”
```

Afterwards, restart the SSH server by executing the following command as root:

```
systemctl restart sshd
```

## IPTables
The next big step in securing your machine is using IPTables. On Ubuntu, in order to have IPTables save, I have to install the `iptables-persistent` package. You can do so by executing the following command as root and choosing your options:

```
apt-get install iptables-persistent
```

I won’t get into the specifics of IPTables (going to leave that for another guide), but I would strongly recommend only white-listing the services the server will use and dropping all other traffic. After white-listing the necessary ports and IPs, set the chain’s policy to DROP (which will drop all traffic by default if no rules are matched against the packet). 

Afterwards, you can save IPTables by executing the following command as root:

```
netfilter-persistent save
```

After you’re done configuring IPTables, I’d strongly recommend making a backup as well which can be done by the following command as root:

```
iptables-save > /path/to/file
```

Additionally, you will most likely be white-listing the SSH port (which is 22 by default). With SSH keys, the security is already good but if you want to take it one step further, you can white-list only certain IPs for SSH access. This obviously comes with penalties as well (e.g. you will need to have the white-listed IP to access SSH). With that said, if you go with this option, ensure you have a host (e.g. a VPS or a dedicated machine) that has a static IP. In the case that your home IP address changes, you’ll be able to SSH as this host and white-list your new home IP.

## [O] WAFs/OpenVPN
Having your machines under a WAF (Web Application Firewall) such as [pfSense](https://www.pfsense.org/), [Sophos](https://www.sophos.com/en-us.aspx), or [IPFire](https://www.ipfire.org/) will certainly help the security and give you more flexibility with firewall/NAT rules. Keep in mind, this can add some network overhead to your machines and if the firewall goes down, all the machines under it will likely go down. For example, with game servers, I don’t recommend having a WAF because it’ll add possible latency overhead (latency is very important for game servers). For websites, some latency overhead won’t be a big deal and the extra security features is definitely worth it.

Some WAFs also come with a built-in OpenVPN server (or another type of VPN server) which can further secure machines under the firewall (e.g. only white-list VPN IPs to certain services on the machines under the WAF such as SSH).

## [O] IDS/IPS
IPS stands for Intrusion Protection System and IDS stands for Intrusion Detection System. You can read the differences [here](https://www.varonis.com/blog/ids-vs-ips/):

Most WAFs come with an IPS such as Snort. This will help prevent malicious attacks by inspecting the packets that come through the WAF. IDSs are installed onto the machines themselves and they don’t necessarily prevent malicious attacks. However, they will log any suspicious activity to system admins. They monitor logs and configs.

## Conclusion
These are things I keep in mind when setting up my own machines and firewalls. As I learn new things involving security, I will add them to this article. I would like to state that most breaches and security leaks are an inside job. Therefore, make sure to watch who you give access to. With that said, keep in mind that the security of the software you run is equally as important as the above. For example, if you’re running a web server with PHP, ensure you harden the security there as well.

If you have any feedback or anything I can add/do better, please let me know! I’m still learning a lot but I just wanted to share things that I already know.

Thank you for reading!

**[Original Source](https://gflclan.com/guides/basic-linux-security-r19/)**