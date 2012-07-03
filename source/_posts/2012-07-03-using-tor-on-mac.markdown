---
layout: post
title: "Using Tor on Mac"
date: 2012-07-03 19:00
comments: true
categories: web, mac
published: false
---

Anonymity Online on Mac with Tor

<!-- more -->

# Prerequisites

You need Homebrew on your Mac

http://mxcl.github.com/homebrew


# Installation

Installation using brew:

``` bash
    $ brew install tor
```

You can start tor automatically on login with:

``` bash
mkdir -p ~/Library/LaunchAgents
cp /usr/local/Cellar/tor/0.2.2.37/homebrew.mxcl.tor.plist ~/Library/LaunchAgents/
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.tor.plist
```

# Running

Starting Tor in the foreground:

``` bash
    $ /usr/local/Cellar/tor/0.2.2.37/bin/tor
```

# Configure SOCKS Proxy

Go to the Network Settings and configure the SOCKS Proxy to 127.0.0.1:9050

{% img /downloads/images/socks-proxy.jpg %}


# Check

Just visit: https://check.torproject.org
