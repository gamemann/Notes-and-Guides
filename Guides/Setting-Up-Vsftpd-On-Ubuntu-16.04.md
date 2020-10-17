# Setting Up Vsftpd With Ubuntu 16.04 LTS Server Under A Web Server Environment
**Created on January 30th, 2018**

Hello, I will be showing you how to set up [vsftpd](https://security.appspot.com/vsftpd.html) with Ubuntu 16.04 LTS server under a web server environment.

## Introduction To Vsftpd
Vsftpd is a FTP server for UNIX systems including Linux. Vsftpd is stable, secure, fast, and very easy to install on Ubuntu 16.04. 

A question that may be asked is:

“Why use FTP over SFTP?” 

One reason a user would prefer FTP over SFTP is due to FTP being normally faster. Although, keep in mind there are downfalls to using FTP over SFTP. SFTP is typically more secure than FTP.

You can read more about vsftpd [here](https://security.appspot.com/vsftpd.html#about) and learn more about SFTP vs FTP [here](https://southrivertech.com/whats-difference-ftp-sftp-ftps/) and [here](https://blog.ftptoday.com/sftp-vs.-ftp-understanding-the-difference).

## The Situation
So I am running a website with [Nginx + PHP 7.1 FPM](https://github.com/gamemann/Notes-and-Guides/blob/master/Guides/How-To-Setup-NGINX-And-PHP-7.1-On-Ubuntu-16.04.md). The website runs under its own user (in this case, site1). The website user’s home directory is `/var/www/site1/` and the website’s root path is `/var/www/site1/public_html/`. I want to have it so I can log in through FTP (port 21) for the site1 user and create/edit files and directories for the website. We can easily achieve this with vsftpd.

## Prerequisites
* Shell access to the Ubuntu 16.04 server.

* Basic knowledge of Linux.

* A user with root access (either the root user itself or running commands with sudo).

* A user we can use to log in through with vsftpd (e.g. a website user).

## Before Continuing
Ensure the user we are using vsftpd for has a password set through Linux. You can achieve this by executing the following command and entering a new password:

```
sudo passwd <user>
```

You will be prompted to enter a new password after entering this command.

### Example
```
sudo passwd site1
```

## Installing Vsftpd
You can install Vsftpd through the APT package manager. You can do this by executing the following command:

```
sudo apt-get install vsftpd -y
```

I’d suggest downloading and installing any recommended dependencies if there is any.

Finally, just enable the `vsftpd` service by executing the following command:

```
sudo systemctl enable vsftpd
```

This will make the vsftpd service automatically start on bootup.


That’s it for installing vsftpd! Read how to configure the vsftpd file to suit our situation’s needs below.

## Configuring Vsftpd
Configuring vsftpd is quite easy. In Ubuntu 16.04 LTS, most of the configuration is done in one file. The file is `/etc/vsftpd.conf`. Let’s begin configuring!

Open the file using the text editor of your choice. I personally like to use Vim. Here’s the command I use to open the file via Vim:

```
sudo vim /etc/vsftpd.conf
```

If you’re new to Vim, please read the manual to learn how to use it. You can read this tutorial [here](https://www.linux.com/learn/vim-101-beginners-guide-vim) if you’d like.

Fortunately, with Ubuntu 16.04’s vsftpd package the configuration file shouldn’t need many things altered to have things suit the situation’s needs. I remember installing vsftpd on another Linux distro which came with a different configuration and I had to alter many more things to suit my needs.

### Have Vsftpd Listen
The first line we want to change is listen=NO. We want to change this to YES. This will make it so the vsftpd service itself listens on port 21 (FTP).

```
listen=YES
```

#### From
```
listen=NO
```

#### To
```
listen=YES
```

Additionally, you can set `listen_ipv6` to `NO` for good practice since we won’t be using IPv6. It shouldn’t make a difference whether it is on or off, though.

```
listen_ipv6=NO
```

#### From
```
listen_ipv6=YES
```

#### To
```
listen_ipv6=NO
```

### Enabling Writing
The next thing we want to do is enable writing with vsftpd. We need to find the commented out line that contains `write_enable=YES` and uncomment it by removing the pound character (`#`) at the beginning of the line so it looks like this:

```
write_enable=YES
```

#### From
```
#write_enable=YES
```

#### To
```
write_enable=YES
```

### Enabling Umask
The next thing we want to do is enable umask. The umask has to do with file permissions and it subtracts from the base mask (*666* for files and *777* for directories). For example, having a umask of *022* will make files created through FTP have *644* permissions and directories created through FTP have *755* permissions. You can use an umask calculator here and learn what umask is here.

Find the commented out line that contains `local_umask=022` and uncomment it by removing the pound character (`#`) at the beginning of the line.

```
local_umask=022
```

#### From
```
#local_umask=022
```

#### To
```
local_umask=022
```

Keep in mind, the umask value of *022* is probably the best for website FTP users which is why we’re keeping it.

### Chrooting Users
Enabling chrooting will restrict each FTP user to their home directory. Without chroot, the user would be able to explore directories and files outside of the home directory as long as they had permission to do so. In our situation, I do not want users to be able to explore outside of the home directory.

To achieve this, we need to uncomment the first line that contains `chroot_local_user=YES`. Uncomment the line by removing the pound character (`#`) at the beginning of the line.

```
chroot_local_user=YES
```

#### From
```
#chroot_local_user=YES
```

#### To
```
chroot_local_user=YES
```

You are done configuring vsftpd! Save the file and exit.

 

Additionally, if you do not want specific users to use vsftpd, you can edit the `/etc/ftpusers` file and add the user(s) there.

Finally, restart the vsftpd service by executing the following command:

```
sudo systemctl restart vsftpd
```

You should be able to login through FTP by connecting to the server’s IP address on port 21 (default) with the username and password of the Linux user you want to connect with. On Windows, I’d recommend using a FTP client such as [Filezilla](https://filezilla-project.org/download.php).

## Frequently Asked Questions
### Question
I have everything configured and a password set for the user but I receive a “530 Login Incorrect” error. What do I do?

### Answer
This could be related to this [issue](https://askubuntu.com/questions/617370/why-vsftpd-doesnt-work-when-pam-service-name-vsftpd).

Basically, ensure the user’s shell is listed in the `/etc/shells` file. Additionally, you can add the shell your user is using to the list (e.g. `/bin/false` or `/bin/nologin`).

There is another workaround to this issue as well but it is **not** recommended. You can edit the `/etc/vsftpd.conf` file, find the line starting with `pam_service_name`, and change the value to something other than `vsftpd`. For example, you could change it to `vsftp`.

```
pam_service_name=vsftp
```

#### From
```
pam_service_name=vsftpd
```

#### To
```
pam_service_name=vsftp
```

After doing this, vsftpd won’t be using the vsftpd PAM service (used for authentication and more). Therefore, users specified in the `/etc/ftpusers` list will be able to login through FTP since the vsftpd PAM service is what enables the exclusion list.

### Question
I receive the error “500 OOPS: vsftpd: refusing to run with writable root inside chroot()”. What do I do?

### Answer
Vsftpd requires the user’s home directory to only be readable and/or executable. It **cannot** be writable. You can remove the write permission from the directory by executing the following command:

```
sudo chmod a-w /path/to/home/directory
```

In our situation, we would do:

```
sudo chmod a-w /var/www/site1
```

If you run into additional issues after this, you can try setting the file permissions specifically by running the following command:

```
sudo chmod 555 /path/to/home/directory
```

### Question
I am trying to connect but the connection is timing out. What do I do?

### Answer
Firstly, ensure the vsftpd service is running by executing the following command:

```
sudo systemctl status vsftpd
```

If you see “Active: active (running)” in the output, the service is running. Otherwise, it either failed to start or is generally down. If it failed to start, you would need to look at the log files or receive more details by adding the `-l` flag to `systemctl`.

Secondly, ensure port 21 isn’t being blocked by your **firewall**. You may need to whitelist the port in [IPTables](https://wiki.archlinux.org/index.php/iptables) if that is enabled.

## Useful Guides
* [Setting up vsftpd for a User’s Directory on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-vsftpd-for-a-user-s-directory-on-ubuntu-16-04).

## Conclusion
That’s it! You should have vsftpd running! If you find something incorrect about this guide, please let me know! I am trying to improve in every area I can.

This guide was written by Christian Deacon. If you have any questions, please post a comment!

Thank you for reading!

**[Original Source](https://gflclan.com/guides/setting-up-vsftpd-with-ubuntu-1604-lts-server-under-a-web-server-environment-r6/)**