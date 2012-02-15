---
layout: post
title: "JavaScriptCore on Mac"
date: 2012-02-15 09:48
comments: true
categories: [JavaScript Mac Shell]
---

[JavaScriptCore](http://trac.webkit.org/wiki/JavaScriptCore) is the built-in JavaScript engine for WebKit


You can find it here:

``` bash
$ /System/Library/Frameworks/JavaScriptCore.framework/Versions/A/Resources/jsc
```
<!--more-->

You can symlink it somewhere to your $PATH for convenience:


``` bash
$ sudo ln -s /System/Library/Frameworks/JavaScriptCore.framework/Versions/A/Resources/jsc /usr/bin/jsc
```

Time to test:


``` bash
$ jsc --help
Usage: jsc [options] [files] [-- arguments]
  -d         Dumps bytecode (debug builds only)
  -e         Evaluate argument as script code
  -f         Specifies a source file (deprecated)
  -h|--help  Prints this help message
  -i         Enables interactive mode (default if no files are specified)
  -s         Installs signal handlers that exit on a crash (Unix platforms only)

$ jsc
> var foo = "bar";
undefined
> typeof(foo);
string
```

## Changenotes

- 2012-02-15 created
