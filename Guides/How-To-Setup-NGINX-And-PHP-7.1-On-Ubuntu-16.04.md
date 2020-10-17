# How To Set Up Nginx & PHP 7.1 FPM On Ubuntu 16.04 Server
**Created on January 29th, 2018**

Hello, I will be showing you how to install Nginx and PHP 7.1 on an Ubuntu 16.04 server. I made this tutorial while using a stock image of Ubuntu 16.04 LTS server.

## Prerequisites
* Shell access to the Ubuntu 16.04 server.

* Basic knowledge of Linux.

* A user with root access (either the root user itself or running commands with sudo).

* A domain pointing at the Ubuntu 16.04 server (e.g. [gflclan.com](https://gflclan.com/)).
 
## Installing PHP 7.1
First, you need to install the `software-properties-common` package via apt:

```
sudo apt-get install software-properties-common
```

This software provides an abstraction of the used apt repositories. It allows you to easily manage your distribution and independent software vendor software sources.

Next, you need to add the PPA that includes PHP 7.1 packages via apt:

```
sudo add-apt-repository ppa:ondrej/php
```

Now you need to update apt to add the PHP 7.1 packages:

```
sudo apt-get update
```

Next, install the common PHP packages along with PHP FPM via apt:

```
sudo apt-get install php7.1-common php7.1 php7.1-cli php7.1-fpm php7.1-json php7.1-opcache php7.1-readline -y
```

Keep in mind, this includes the stock packages. You may list additional packages you would like installed by executing `sudo apt-cache search php7.1` and running `sudo apt-get install <packageName> -y` to install them.

Lastly, let’s enable the **php7.1-fpm** service by executing the following command:

```
sudo systemctl enable php7.1-fpm
```

This will make it so the PHP 7.1 FPM service automatically starts up on bootup.

There you have it, PHP 7.1 should be installed! You should be able to use the same process with installing updated versions of PHP. For example, instead of using 7.1, you can use 7.2 and so on.

## Configuring PHP 7.1 + FPM
Configuring PHP 7.1 is quite simple, but there are a few ways you can do it. I’m going to show you the way I normally configure PHP 7.1.

### Make A New User
I normally run multiple websites off of one installation. Therefore, I have PHP FPM run under its own user/group for each website (this is typically more secure as well). In this case, we’re going to create a group named **site1** and a user named **site1**.

You can add the group with the following command:

```
sudo groupadd site1
```

You can add the user with the following command (feel free to alter if you have enough knowledge):

```
sudo useradd -m -d /var/www/site1 -g site1 -s /bin/false site1
```

This will automatically create a directory named site1 in `/var/www` (Nginx’s default root directory), you will need to put your site files in this folder.

### Create A New PHP 7.1 FPM Pool
PHP FPM pools are configured in `/etc/php/7.1/fpm/pool.d/`. I normally copy `www.conf` for each pool/website I make and configure it to my needs. 

Let’s start by changing our current directory to `/etc/php/7.1/fpm/pool.d/` to make files easier to copy (we won’t need to enter the full paths). You can do this by executing the following command:

```
cd /etc/php/7.1/fpm/pool.d/
```

You shouldn’t need root access for this since the directory should be readable and executable by all users.

Next, let’s copy the `www.conf` file for our site1 website:

```
sudo cp www.conf site1.conf
```

Afterwards, edit the new `site1.conf` file using the text editor of your choice. I personally prefer Vim:

```
sudo vim site1.conf
```

If you’re new to Vim, please read the manual to learn how to use it. You can read this tutorial [here](https://www.linux.com/learn/vim-101-beginners-guide-vim) if you’d like.

You will see something like this:

```
; Start a new pool named 'www'.
; the variable $pool can be used in any directive and will be replaced by the
; pool name ('www' here)
[www]

; Per pool prefix
; It only applies on the following directives:
; - 'access.log'
; - 'slowlog'
; - 'listen' (unixsocket)
; - 'chroot'
; - 'chdir'
; - 'php_values'
; - 'php_admin_values'
; When not set, the global prefix (or /usr) applies instead.
; Note: This directive can also be relative to the global prefix.
; Default Value: none
;prefix = /path/to/pools/$pool

; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
user = www-data
group = www-data

; The address on which to accept FastCGI requests.
; Valid syntaxes are:
;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific IPv4 address on
;                            a specific port;
;   '[ip:6:addr:ess]:port' - to listen on a TCP socket to a specific IPv6 address on
;                            a specific port;
;   'port'                 - to listen on a TCP socket to all addresses
;                            (IPv6 and IPv4-mapped) on a specific port;
;   '/path/to/unix/socket' - to listen on a unix socket.
; Note: This value is mandatory.
listen = /run/php/php7.1-fpm.sock

; Set listen(2) backlog.
; Default Value: 511 (-1 on FreeBSD and OpenBSD)
;listen.backlog = 511
...
```

First, we want to change the first uncommented (line without a semicolon at the beginning) from `[www]` to `[site1]`.
 
#### From
```
[www]
```

#### To
```
[site1]
```

Second, find the lines starting with `user =` and `group =`. We want to change the values after the equal sign to the new user and group we made for the website. Change those lines to this:

```
user = site1
group = site1
```

#### From
```
user = www-data
group = www-data
```

#### To
```
user = site1
group = site1
```

Lastly, we want to find the first uncommented line that starts with listen. By default, it will listen to `/run/php/php7.1-fpm.sock`. However, I like to change this to something easier to remember. In this case, let’s change it to `/run/php/site1.sock`.

```
listen = /run/php/site1.sock
```

#### From
```
listen = /run/php/php7.1-fpm.sock
```

#### To
```
listen = /run/php/site1.sock
```

You are done configuring the PHP 7.1 FPM pool! Everything else should be set up correctly. Feel free to look into configuring PHP FPM further by reading here.

### Disable Harmful Functions In PHP
Securing PHP is very important and there are a few useful guides I will post at the end of the tutorial (or through the comments). The first step I usually take to secure PHP is disabling harmful functions (e.g. functions that will execute commands through the shell).

You will need to open the PHP configuration file using a text editor of your choice. The PHP configuration file will be named php.ini and should be located in `/etc/php/7.1/fpm/`. I will edit the file using Vim by executing the following command:

```
sudo vim /etc/php/7.1/fpm/php.ini
```

We will need to find the line starting with `disable_functions`. In Vim, you can hit SHIFT + Colon (:), type `/disable_functions`, and hit enter. It should then go to the line with `disable_functions=`. Personally, I like using the following:

```
disable_function = apache_child_terminate,apache_setenv,define_syslog_variables,escapeshellarg,escapeshellcmd,eval,fp,fput,ftp_connect,ftp_exec,ftp_get,ftp_login,ftp_nb_fput,ftp_put,ftp_raw,ftp_rawlist,highlight_file,ini_alter,ini_get_all,ini_restore,inject_code,mysql_pconnect,openlog,passthru,php_uname,phpAds_remoteInfo,phpAds_XmlRpc,phpAds_xmlrpcDecode,phpAds_xmlrpcEncode,popen,posix_getpwuid,posix_kill,posix_mkfifo,posix_setpgid,posix_setsid,posix_setuid,posix_setuid,posix_uname,proc_close,proc_get_status,proc_nice,proc_open,proc_terminate,shell_exec,syslog,system,xmlrpc_entity_decode,parse_ini_file,show_source,exec,pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_get_handler,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,pcntl_async_signals,
```

#### From
```
disable_functions=...
```

#### To 
```
disable_function = apache_child_terminate,apache_setenv,define_syslog_variables,escapeshellarg,escapeshellcmd,eval,fp,fput,ftp_connect,ftp_exec,ftp_get,ftp_login,ftp_nb_fput,ftp_put,ftp_raw,ftp_rawlist,highlight_file,ini_alter,ini_get_all,ini_restore,inject_code,mysql_pconnect,openlog,passthru,php_uname,phpAds_remoteInfo,phpAds_XmlRpc,phpAds_xmlrpcDecode,phpAds_xmlrpcEncode,popen,posix_getpwuid,posix_kill,posix_mkfifo,posix_setpgid,posix_setsid,posix_setuid,posix_setuid,posix_uname,proc_close,proc_get_status,proc_nice,proc_open,proc_terminate,shell_exec,syslog,system,xmlrpc_entity_decode,parse_ini_file,show_source,exec,pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_get_handler,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,pcntl_async_signals,
```

Lastly, restart the PHP 7.1 FPM service for these changes to take effect:

```
sudo systemctl restart php7.1-fpm
```

There you have it! Configuring PHP 7.1 + FPM should be complete! There is additional configuration you can do to PHP such as performance tweaking and so on. I will link useful guides at the end of the tutorial and will most likely make some of my own in the future.

## Installing Nginx
Installing Nginx is simple through the APT package manager. You can execute the following command:

```
sudo apt-get install nginx -y
```

I’d recommend downloading and installing the recommended dependencies if asked. After it is installed, let’s enable the service by executing the following command:

```
sudo systemctl enable nginx
```

This will make it so the Nginx automatically starts up on bootup.

 

It’s that simple, Nginx is now installed!

## Configuring Nginx
Configuring Nginx is simple since most of the settings are tuned correctly from the installation.

We will make a new configuration file for site1. Site configs are handled in the `/etc/nginx/sites-enabled/` directory. However, what I like doing is configuring the site file in the `/etc/nginx/sites-available/` directory and creating a symbolic link from `/etc/nginx/sites-available/<siteFile>` to `/etc/nginx/sites-enabled/<siteFile>`. 

First, let’s change our current directory to `/etc/nginx/sites-available/`:

```
cd /etc/nginx/sites-available
```

You will not need root access for this since the directory should be readable and executable by all users. 

Next, let’s copy the default file for our site1 website. You can do this by executing the following command:

```
sudo cp default site1
```

Some users may prefer to use `site1.conf` which is fine since that makes things look cleaner.

We will need to edit the file using a text editor of your choice. I will open it with Vim:

```
sudo vim site1
```

The file will look like this:

```
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# http://wiki.nginx.org/Pitfalls
# http://wiki.nginx.org/QuickStart
# http://wiki.nginx.org/Configuration
#
# Generally, you will want to move this file somewhere, and start with a clean
# file but keep this around for reference. Or just disable in sites-enabled.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php7.0-cgi alone:
        #       fastcgi_pass 127.0.0.1:9000;
        #       # With php7.0-fpm:
        #       fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}

# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#       listen 80;
#       listen [::]:80;
#
#       server_name example.com;
#
#       root /var/www/example.com;
#       index index.html;
#
#       location / {
#               try_files $uri $uri/ =404;
#       }
#}
```

The first lines we will look for are the ones starting with listen. You should see two:

```
listen 80 default_server;
listen [::]:80 default_server;
```

We want to remove the default_server from the first line and comment out the second line by adding a `#` at the beginning because we aren’t using IPv6. It should look like this:

```
listen 80 default_server;
#listen [::]:80 default_server;
```

#### From
```
listen 80 default_server;
listen [::]:80 default_server;
```

#### To
```
listen 80;
#listen [::]:80 default_server;
```

The next line you want to look for starts with root. Change this from root `/var/www/html` to the directory we want to store our website files in. In this case, I want to use the home directory of the new user I created for the website which is `/var/www/site1`. The line should now look like this:

```
root /var/www/site1;
```

#### From
```
root /var/www/html;
```

#### To
```
root /var/www/site1;
```

You will then want to find the line starting with index and append `index.php` after `index`. I also remove useless index files such as `index.nginx-debian.html`. The line should look like this:

```
index index.php index.html index.htm;
```

#### From
```
index index.html index.htm index.nginx-debian.html;
```

#### To
```
index index.php index.html index.htm;
```

The next line should be right under the last line. It should start with `server_name`. You would put the domains of your website. In this case, my website’s domain is gflclan.com. This is how it should look:

```
server_name gflclan.com www.gflclan.com;
```

#### From
```
server_name _;
```

#### To
```
server_name gflclan.com www.gflclan.com;
```

Additionally and optionally, you can add access and error logging for the specific website by adding the following two lines:

```
access_log /var/log/nginx/site1_access.log;
error_log /var/log/nginx/site1_error.log;
```

You can change the log files if you’d like. You can also comment one out if you do not want to use it. I usually disable the access log on large websites since the file would grow to be very large and keep the error log.

Next, we want to enable handling of the PHP files on the website. If you scroll down, you’ll see a bunch of lines commented out that have to do with PHP:

```
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
#
#location ~ \.php$ {
#       include snippets/fastcgi-php.conf;
#
#       # With php7.1-cgi alone:
#       fastcgi_pass 127.0.0.1:9000;
#       # With php7.1-fpm:
#       fastcgi_pass unix:/run/php/php7.1-fpm.sock;
#}
```

Remove the first comment (`#`) of each line so it looks like this:

```
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
#
location ~ \.php$ {
       include snippets/fastcgi-php.conf;

       # With php7.1-cgi alone:
       fastcgi_pass 127.0.0.1:9000;
       # With php7.1-fpm:
       fastcgi_pass unix:/run/php/php7.1-fpm.sock;
}
```

Let’s comment out the following line since we’re listening on a socket instead of an address/port:

```
#fastcgi_pass 127.0.0.1:9000;
```

#### From
```
fastcgi_pass 127.0.0.1:9000;
```

#### To
```
#fastcgi_pass 127.0.0.1:9000;
```

We need to change the socket path on the next `fastcgi_pass` variable to the socket we made back when configuring PHP 7.1 FPM. The line should now look like this:

```
fastcgi_pass unix:/run/php/php/site1.sock;
```

#### From
```
fastcgi_pass unix:/run/php/php7.1-fpm.sock;
```

#### To
```
fastcgi_pass unix:/run/php/site1.sock;
```

Now save the file and exit.

We will need to create a symbolic link from `/etc/nginx/sites-available/site1` to `/etc/nginx/sites-enabled/site1`. We can do this by executing the following command:

```
ln -s /etc/nginx/sites-available/site1 /etc/nginx/sites-enabled/site1
```

Files in the `/etc/nginx/sites-enabled/` directory are included in the Nginx config.

Finally, let’s restart Nginx by executing the following command:

```
sudo systemctl restart nginx
```

You could also use `sudo systemctl reload nginx` if you don’t want to restart the Nginx server.

That’s it! Nginx and PHP 7.1 FPM should be fully configured to work with each other on the specific website you made! Try accessing the website by going to your domain on port 80 (default).

## Useful Guides

* [PHP security](https://php.net/manual/en/security.php).

* [PHP performance tuning](https://blog.appdynamics.com/engineering/php-performance-crash-course-part-1-the-basics/).

* [PHP FPM performance tuning](http://www.zend.com/en/products/server/nginx-performance).

* [Nginx performance tuning](https://www.nginx.com/blog/tuning-nginx/).

## Conclusion
You should have an Nginx server configured to run with PHP 7.1 and FPM through site1. Additional configuration shouldn’t be required but you can always tweak PHP, FPM, and Nginx settings to make your website perform faster and better. Please ensure any files you put in the `/var/www/site1/` directory are owned by the `site1` user and group. With that said, most website files will only need permissions of *644* while directories should have *755*.

I will be making more guides in the future including how to set up SSL for your website along with FTP users using [VSFTPD](https://security.appspot.com/vsftpd.html). This is my first guide regarding Nginx and PHP. If you see anything I can improve on, please let me know! I tried my best to make the guide clean and easy to read.

This guide was written by Christian Deacon. If you have any questions, please post a comment!

Thanks for reading!

**[Original Source](https://gflclan.com/guides/how-to-set-up-nginx-php-71-fpm-on-ubuntu-1604-server-r4/)**