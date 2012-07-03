---
layout: post
title: "Testing Zabbix 2.0"
date: 2012-04-30 17:21
comments: true
categories:
published: false
---

Today I'm going to test Zabbix 2.0 on a Ubuntu 11.10 VirtualBox.

<!-- more -->


# Steps to go

1. Set up a new VirtualBox Sandbox
2. Install Zabbix 2.0
3. Set up and Configure some Zabbix Tests


# Tools I want to use

1. Ubuntu 11.10 Server
2. Vagrant
3. Veewee
4. Zabbix


# Preparation

It is already some time ago since I used Vagrant and Veewee, thus, I decided to
update my system first and learn the things again from scratch...


## Installing rvm
<https://rvm.io/install>

``` bash
curl -L get.rvm.io | bash -s stable
```


## Upgrading rvm
<https://rvm.io/rvm/upgrading>

``` bash
rvm get stable
```


## Installing VeeWee

*VeeWee is a tool to easily build vagrant base boxes*

``` bash
$ rvm install 1.9.3
$ rvm use 1.9.3
$ rvm gemset create veewee
$ rvm use 1.9.3@veewee
Using /Users/rbartl/.rvm/gems/ruby-1.9.3-p194 with gemset veewee

$ gem install veewee
```


## Creating an Ubuntu 11.04 "basebox"

You can find an detailed how to on this site:
<http://seletz.github.com/blog/2012/01/17/creating-vagrant-base-boxes-with-veewee>


### ... but in short

VeeWee adds a subcommand to `vagrant` called "basebox"

To list the available templates, call `vagrant basebox templates`

To create your own basebox out of one of the templates, follow these steps

``` bash
$ cd $HOME
$ vagrant basebox define "ubuntu-11.10-server" 'ubuntu-11.10-server-amd64'
The basebox 'ubuntu-11.10-server' has been successfully created from the template 'ubuntu-11.10-server-amd64'
You can now edit the definition files stored in definitions/ubuntu-11.10-server
or build the box with:
vagrant basebox build 'ubuntu-11.10-server'
```

A folder called `definitions` has been created.
You can customize these to your needs or leave it to the defaults.

``` bash
$ tree definitions
definitions
└── ubuntu-11.10-server
    ├── definition.rb
    ├── postinstall.sh
    └── preseed.cfg
```

Now you can build the basebox

``` bash
$ vagrant basebox build 'ubuntu-11.10-server'

We did not find an isofile in <currentdir>/iso.

The definition provided the following download information:
- Download url: http://releases.ubuntu.com/11.10/ubuntu-11.10-server-amd64.iso
- Md5 Checksum: f8a0112b7cb5dcd6d564dbe59f18c35f

Download? (Yes/No)  |No|
Yes

[... this will take some time ...]
```

**hint: If you downloaded the iso already, simply create a folder called `iso` in
your home folder and put it there**

Validating and exporting the Basebox
``` bash
$ vagrant basebox validate 'ubuntu-11.10-server'
[...]
7 scenarios (7 passed)
21 steps (21 passed)
0m5.826s

$ vagrant basebox export 'ubuntu-11.10-server'
Vagrant requires the box to be shutdown, before it can export
Sudo also needs to work for user vagrant
Performing a clean shutdown now.
Executing command: sudo shutdown -P now

Broadcast message from vagrant@ubuntu-11
    (/dev/pts/0) at 14:33 ...

The system is going down for power off NOW!
......
Machine ubuntu-11.10-server is powered off cleanly
Executing vagrant voodoo:
vagrant package --base 'ubuntu-11.10-server' --output 'ubuntu-11.10-server.box'

$ vagrant box add 'ubuntu-11.10-server' 'ubuntu-11.10-server.box'
```


## Getting Ubuntu 11.10 up and running

``` bash
$ mkdir -p ~/Vagrant/zabbix
$ cd !$
$ vagrant init 'ubuntu-11.10-server'
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

Adjust the Vagrantfile:

``` bash
$ vim Vagrantfile

# forward port 80 to our local machine
config.vm.forward_port 80, 8080
```

Start the Machine and SSH into it:

``` bash
$ vagrant up
$ vagrant ssh
```


## Installing Zabbix 2.0

Install required Packages:

```bash
$ sudo aptitude install -y build-essential
$ sudo aptitude install -y mysql-server
$ sudo aptitude install -y libmysqlclient16-dev
$ sudo aptitude install -y php5
$ sudo aptitude install -y php5-gd
$ sudo aptitude install -y php5-mysql
$ sudo aptitude install -y snmp
$ sudo aptitude install -y libsnmp-dev
$ sudo aptitude install -y snmpd
$ sudo aptitude install -y libcurl4-openssl-dev
$ sudo aptitude install -y fping
```

Make the zabbix user and group:

```bash
$ sudo adduser zabbix
$ adduser zabbix admin
$ sudo su - zabbix
$ id
uid=1001(zabbix) gid=1002(zabbix) groups=1002(zabbix),111(admin)
```

Download and unpack Zabbix:

```bash
$ wget http://prdownloads.sourceforge.net/zabbix/zabbix-2.0.0rc3.tar.gz
$ tar xzpf zabbix-2.0.0rc3.tar.gz
```

Create the Database:

```bash
$ sudo mysql -e"create database zabbix;" -p
Enter password:
$ sudo mysql -e"grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';" -p
Enter password:
```
**Note: You can omit the `-p` if you didn't set a root password for the database at install time**

Initialize the Database:

``` bash
$ mysql -D zabbix -uzabbix -pzabbix < /home/zabbix/zabbix-2.0.0rc3/database/mysql/schema.sql
$ mysql -D zabbix -uzabbix -pzabbix < /home/zabbix/zabbix-2.0.0rc3/database/mysql/data.sql
$ mysql -D zabbix -uzabbix -pzabbix < /home/zabbix/zabbix-2.0.0rc3/database/mysql/images.sql
```

Configure, compile and install the server:

``` bash
$ cd zabbix-2.0.0rc3
$ ./configure --prefix=/usr --with-mysql --with-net-snmp --with-libcurl --enable-server --enable-agent
$ sudo make install
```

Prepare the rest of the system:

``` bash
$ sudo vim /etc/services
```
and add this at the end of the file:

```
zabbix_agent  10050/tcp  # Zabbix ports
zabbix_trap   10051/tcp
```

Bootstrap configuration Files

``` bash
$ sudo mkdir /etc/zabbix
$ sudo chown -R zabbix.zabbix /etc/zabbix/
$ cp conf/zabbix_*.conf /etc/zabbix/
```

Change the server config:

```
$ vim /etc/zabbix/zabbix_server.conf

# Database user
DBUser=zabbix
# Database password
# Comment this line if no password used
DBPassword=zabbix
```

copy init.d scripts:

``` bash
$ sudo cp misc/init.d/debian/zabbix-server /etc/init.d
$ sudo cp misc/init.d/debian/zabbix-agent /etc/init.d
```

Adjust the scripts

``` bash
$ sudo vim /etc/init.d/zabbix-server
DAEMON=/usr/sbin/${NAME}

$ sudo vim /etc/init.d/zabbix-agent
DAEMON=/usr/sbin/${NAME}
```

Make them executable

``` bash
$ sudo chmod 755 /etc/init.d/zabbix-server
$ sudo /usr/sbin/update-rc.d zabbix-server defaults
$ sudo chmod 755 /etc/init.d/zabbix-agent
$ sudo /usr/sbin/update-rc.d zabbix-agent defaults
```

Starting the Server

``` bash
$ sudo /etc/init.d/zabbix-server start
Starting Zabbix server daemon: zabbix_server
$ sudo /etc/init.d/zabbix-agent start
Starting Zabbix agent daemon: zabbix_agentd
$ ps aux | grep zabbix
zabbix   24390  0.0  0.5  76376  1984 ?        S    16:21   0:00 /usr/sbin/zabbix_server
zabbix   24395  0.0  0.2  66580  1052 ?        S    16:22   0:00 /usr/sbin/zabbix_agentd
zabbix   24396  0.0  0.3  66580  1152 ?        S    16:22   0:00 /usr/sbin/zabbix_agentd
zabbix   24397  0.0  0.2  66580   848 ?        S    16:22   0:00 /usr/sbin/zabbix_agentd
zabbix   24398  0.0  0.2  66580   848 ?        S    16:22   0:00 /usr/sbin/zabbix_agentd
zabbix   24399  0.0  0.2  66580   848 ?        S    16:22   0:00 /usr/sbin/zabbix_agentd
zabbix   24400  0.0  0.2  66580  1084 ?        S    16:22   0:00 /usr/sbin/zabbix_agentd
```

Configure web interface

``` bash
$ mkdir /home/zabbix/public_html
$ cp -R frontends/php/* /home/zabbix/public_html/
```

Configure Apache

``` bash
$ sudo vim /etc/apache2/sites-enabled/000-default
```

add this to the config

```
    Alias /zabbix /home/zabbix/public_html/
    <Directory /home/zabbix/public_html>
      AllowOverride FileInfo AuthConfig Limit Indexes
      Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
      <Limit GET POST OPTIONS PROPFIND>
        Order allow,deny
        Allow from all
      </Limit>
      <LimitExcept GET POST OPTIONS PROPFIND>
        Order deny,allow
        Deny from all
      </LimitExcept>
    </Directory>
```

Make some php.ini adjustments:

``` bash
$ sudo vim /etc/php5/apache2/php.ini

max_execution_time = 300 ; Maximum execution time of each script, in seconds
date.timezone = Europe/Berlin
```

Restart the Server

``` bash
sudo /etc/init.d/apache2 restart
```


## Web Interface Configuration

Open a Browser and navigate to <http://localhost:8080/zabbix>

{% img /downloads/images/zabbix-introduction.jpg %}

Click next until step 3 **Check of pre-requisits**

{% img /downloads/images/zabbix-pre-requisits.jpg %}

I have some failing tests here. We need to fix that in the php


``` bash
$ sudo vim /etc/php5/apache2/php.ini

; Maximum size of POST data that PHP will accept.
; http://php.net/post-max-size
post_max_size = 16

; Maximum amount of time each script may spend parsing request data. It's a good
; idea to limit this time on productions servers in order to eliminate unexpectedly
; long running scripts.
; Note: This directive is hardcoded to -1 for the CLI SAPI
; Default Value: -1 (Unlimited)
; Development Value: 60 (60 seconds)
; Production Value: 60 (60 seconds)
; http://php.net/max-input-time
max_input_time = 300
```

We need to restart Apache after these adjustments:

``` bash
$ sudo /etc/init.d/apache2 restart
 * Restarting web server apache2
 ... waiting    ...done.
```

After a page reload, everything should be green:

{% img /downloads/images/zabbix-pre-requisits-passed.jpg %}

On the next screen, enter the username/password for your MySQL Database as
configured above. A click on the **Test** Button, should return with **OK**

{% img /downloads/images/zabbix-db-connection.jpg %}

On Screen 7 I got another fail

{% img /downloads/images/zabbix-install.jpg %}

I fixed this, by adjusting the permissions on the **conf** directory:

``` bash
$ chmod o+w /home/zabbix/public_html/conf
```

After a click on the **Retry** Button, the Test passed and the configuration was written:


``` bash
$ cat /home/zabbix/public_html/conf/zabbix.conf.php

<?php
// Zabbix GUI configuration file
global $DB;

$DB['TYPE']                     = 'MYSQL';
$DB['SERVER']                   = 'localhost';
$DB['PORT']                     = '0';
$DB['DATABASE']         = 'zabbix';
$DB['USER']                     = 'zabbix';
$DB['PASSWORD']         = 'zabbix';

// SCHEMA is relevant only for IBM_DB2 database
$DB['SCHEMA']                   = '';

$ZBX_SERVER                             = 'localhost';
$ZBX_SERVER_PORT                = '10051';
$ZBX_SERVER_NAME                = '';

$IMAGE_FORMAT_DEFAULT   = IMAGE_FORMAT_PNG;
?>
```

{% img /downloads/images/zabbix-install-passed.jpg %}

Change the permissions back on the **conf** directory

``` bash
$ chmod o-w /home/zabbix/public_html/conf
```

Click **Finish** after the last step. The followin Login screen should now appear:

{% img /downloads/images/zabbix-login.jpg %}

Log in with username: Admin password: zabbix

{% img /downloads/images/zabbix-dashboard.jpg %}


## Configure the first Test
