---
layout: post
title:  "Debugging Php inside a Docker container"
date:   2016-10-01 12:18:15
categories: php docker
---

I recently moved our Development environment in work to Docker and found that debugging wasn't as straigth forward as before

I had spent half a day trying different things including this solution I came across on [gist.github.com](https://gist.github.com/chadrien/c90927ec2d160ffea9c4)

I had my docker php.ini set up for xdebug:

```ini
[xdebug]
xdebug.remote_enable=1
xdebug.remote_port=9000
xdebug.remote_connect_back=On
xdebug.var_display_max_children = 999
xdebug.var_display_max_data = 999
xdebug.var_display_max_depth = 100
```

I had the xdebug [chrome extension](https://github.com/mac-cain13/xdebug-helper-for-chrome) installed

In my laptop host file I have the following `127.0.0.1 localhost dtest.xxx.com` so I can access my web app (and other docker web apps) through `dtest.xxx.com` which worked fine to run the app

In Phpstorm (version 2016.1.2) In preferences `->` Languages & Frameworks `->` PHP `->` Servers I have :

host = dtest.xxx.com, port=80 , Debugger = Xdebug<br> 
Use path mappings is checked and I have<br>
 - _File/Directory_ set to `/Users/<myname>/projectx/server`<br>
 - _Absolute path_ on the server is set to `/projectx/server`
 
I have tried setting the `xdebug.remote_host` to my macs ip obtained from ifconfig as well as trying the ip in

```
/Users/<myname>/Library/Containers/com.docker.docker/Data/database/com.docker.driver.amd64-linux/slirp/host
```

But I could not hit a breakpoint , I even tried `xdebug_break()` to be sure it wasn't an issue with Phpstorm.

###Workaround and Final Solution
I figured out a workaround by opening an ssh tunnel to the docker container (note the -p 12 is because I had ssh exposed through port 12)
```
ssh -R 9000:localhost:9000 root@dtest.xxx.com -p:12
```

But I wasn't happy with this, it meant I need ssh set up in my container.I eventually figured out that my xdebug settings in docker should be

```
xdebug.enable=1
xdebug.remote_enable = 1
xdebug.idekey="PHPSTORM"
xdebug.remote_port=9000
xdebug.remote_host="192.168.65.1"
xdebug.remote_connect_back=0
```

where 192.168.65.1 was docker host ip found in 

```
~/Library/Containers/com.docker.docker/Data/database/com.docker.driver.amd64-linux/slirp/host
```

I Also had to set the idekey correctly in the chrome extension, in my case PHPSTORM. With this I no longer required the ssh tunnel
