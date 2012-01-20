---
layout: post
title: "Continuous Integration with Jenkins"
date: 2012-01-19 10:55
comments: true
categories: testing, jenkins
---

Jenkins is a opens source continuous integration server.

This blog post will show you how

- to set up Jenkins in an Ubuntu 11.04 Virtual Box
- create a simple Git controlled Project with Jenkins
- run the build job

<!--more-->

##Build-Up

We are going to put Jenkins into a virtual Ubuntu 11.04 (Natty) Box.
Therefore we use vagrant with our self made base box

``` bash
$ mkdir ~/develop/jenkins
$ cd ~/develop/jenkins
$ git init
$ vagrant init 'myubuntubox'
$ git add Vagrantfile
```

**Note**
The *myubuntubox* is a self created base box made with [VeeWee](https://github.com/jedi4ever/veewee).
For a complete instruction, how to quickly build a [Vagrant](http://vagrantup.com/) base box, see
[Stefan's Blogpost about VeeWee](http://seletz.github.com/blog/2012/01/17/creating-vagrant-base-boxes-with-veewee/)

We need to customize the Vagrantfile a little bit for our needs:

``` ruby
# -*- mode: ruby -*-

Vagrant::Config.run do |config|

    config.vm.define :jenkins do |jenkins|
        jenkins.vm.box = "myubuntubox"
        jenkins.vm.network "192.168.50.10"

        # jenkins port
        jenkins.vm.forward_port("jenkins", 8080, 8080)

        # postinstall script
        jenkins.vm.provision :shell, :path => "postinstall.sh"

    end

end

# vim: set ft=ruby :
```

Since we configured a `postinstall.sh` script provisioner, we need to create it
as well:

``` bash
$ cat postinstall.sh
#!/bin/bash

echo "postinstall script ran!" > /vagrant/postinstall.log

$ git add postinstall.sh
```

And finally start the virtual box:

``` bash
$ vagrant up
[jenkins] Importing base box 'myubuntubox'...
...
[jenkins] Running provisioner: Vagrant::Provisioners::Shell...
[jenkins] stdin: is not a tty
```

Check if our provisioner run:

``` bash
$ cat postinstall.log
postinstall script ran!
```


## Installation

Next we are going to install Jenkins into our Ubuntu box.  Therefore we are
going to add the following lines into the `postinstall.sh`

``` bash
#!/bin/bash

log=/vagrant/postinstall.log

echo "postinstall script started!" > $log


function info {
    echo $(date +%D\ %T) $1 >> $log
}

info "Adding convenient packages..."
apt-get install -q -y vim | tee -a $log

# Jenkins Installation
info "Adding Jenkins key to system..."
wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | apt-key add - | tee -a $log

info "Adding Jenkins source to /etc/apt/sources.list..."
if grep "jenkins" /etc/apt/sources.list
then
    info "Jenkins source already in source.list"
else
    echo "deb http://pkg.jenkins-ci.org/debian binary/" >> /etc/apt/sources.list
fi

info "Updating apt source..."
apt-get update | tee -a $log

info "Installing Jenkins..."
apt-get install -q -y jenkins | tee -a $log
```

This Script runs now every time on:

```bash
$ vagrant up
```

or, if the box is already running:

```bash
$ vagrant provision
```

### Recap -- What happened so far

- We created a new [Ubuntu 11.04 (Natty) Server](http://www.ubuntu.com/download/server/download) Box with [Vagrant](http://vagrantup.com)
- We created a [Vagrant Shell Provisioner](http://vagrantup.com/docs/provisioners/shell.html)
- We installed [Jenkins](https://wiki.jenkins-ci.org) from the provided Ubuntu Package Repository
- Jenkins will be launched as a daemon `/etc/init.d/jenkins {start|stop|status|restart|force-reload}`
- Logs will be located in `/var/log/jenkins/jenkins.log`
- The config is in `/etc/default/jenkins`
- Jenkins listens on port 8080

If everything went good so far, we should be able to open up our favorite
browser and go to [http://localhost:8080](http://localhost:8080).

Congratulations, Jenkins is at your command!

If something went wrong, check the `postinstall.log` and login to the virtual
box with:

```bash
$ vagrant ssh
```


## Setting up your first Project

Our first project will be on GitHub, thus, we need to install the Git Plugin first.
Go to the [Jenkins Plugin Manager](http://localhost:8080/pluginManager/available)
and select the `Git Plugin` and the `Github Plugin`.

Click on "Download now and Install after restart".
On the Installation Site, click the checkbox to restart Jenkins after Installation.

Now we are ready to start our first Project.

Click on [New Job](http://localhost:8080/view/All/newJob) and select *Build a
free-style software project*. Set the Job Name to *MyJob*

Set the following Values on the [Job configuration Site](http://localhost:8080/job/MyJob/configure)

- Projectname: MyJob
- Description: A simple test Job
- GitHub Project: https://github.com/ramonski/jenkins-server
- Git Repository Url: https://github.com/ramonski/jenkins-server.git
- **Important** Check "Skip internal tag" Option, since we make anonymous checkouts
- Build trigger: Build when a change is pushed to GitHub and activate the
  checkbox to check the source code management system periodically. Add ther a
  line with `* * * * *` which basically means to check every minute.
- Select "Shell Script" for build

### The Build Script

You can customize how to build your Project.
Therefore you can use [Environment Variables](http://localhost:8080/env-vars.html)

For testing, we echo the values of the build number and the build id into a file:

```
cd $WORKSPACE
echo "I'm build number $BUILD_NUMBER with the ID $BUILD_ID" > buildinfo
```


### First build

Ok, lets start our first build!
Click on [build now](http://localhost:8080/job/MyJob/build?delay=0sec)

The Build should be successful.

You should be able to view the [buildinfo](http://localhost:8080/job/MyJob/ws/buildinfo/*view*/)
inside your workspace.


#### Troubleshooting

If your build failed, check the [build log](http://localhost:8080/job/MyJob/1/console)

If you get the following error:

```
Permission denied (publickey).
fatal: The remote end hung up unexpectedly
```

you probably set the Read+Write Repository URL in the Project Config. Log out
from GitHub and take then the (Read Only) Repository URL.

Another error could tell you this:

```
Command "git tag -a -f -m Jenkins Build #3 jenkins-MyJob-3" returned status code 128: 
*** Please tell me who you are.
```

then you probably forgot to set the "Skip internal tag" on the
[Job configuration Site](http://localhost:8080/job/MyJob/configure)


## Next Steps

Now it's time to prepare Jenkins for the real world.
You probably don't want that everyone can maintain Jenkins through the Web.
So you need to create some Users and give them the propert rights, or you can
configure Jenkins to talk to your companies LDAP.

If you plan to hook the Vagrant Box into your local Network, you can enable
[Bridged Networking in your Vagrant Config](http://vagrantup.com/docs/bridged_networking.html).

There are also a lot of other cool Plugins available for Jenkins, so check out your the
[Plugin Manager](http://localhost:8080/pluginManager/).

A collection of some plugins we use:

- [Jenkins IRC Plugin](http://wiki.jenkins-ci.org/display/JENKINS/IRC+Plugin)
  *This plugin is an IRC bot that can publish build results to IRC channels.*

- [ChuckNorris Plugin](https://wiki.jenkins-ci.org/display/JENKINS/ChuckNorris+Plugin)
  *Displays a picture of Chuck Norris (instead of Jenkins the butler) and a
  random Chuck Norris 'The Programmer' fact on each build page.*

- [Python Plugin](http://wiki.jenkins-ci.org/display/JENKINS/Python+Plugin)
  *Adds the ability to execute python scripts as build steps.*

- [Jython Plugin](https://hudson.dev.java.net/plugin/jython/)
  *Adds the ability to execute Jython script as a build step in free-style
  software projects from within the JVM.*

- [many more...](http://updates.jenkins-ci.org/download/plugins/)
