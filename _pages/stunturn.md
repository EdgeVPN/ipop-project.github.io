---
permalink: /stunturn/
title: "STUN and TURN configuration"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# Introduction

EdgeVPN.io supports NAT traversal using two mechanisms: [STUN](https://en.wikipedia.org/wiki/STUN) (Session Traversal Utilities for NAT), and [TURN](https://en.wikipedia.org/wiki/Traversal_Using_Relays_around_NAT) (Traversal Using Relays around NAT). 

STUN works for the majority of deployed NATs: the [cone-type NATs, including full-cone, port-restricted cone, and address-restricted cone](https://en.wikipedia.org/wiki/Network_address_translation). If your deployment only has cone-type NATs, STUN is sufficient. However, if your nodes are behind symmetric NATs, you will need also TURN.

This document describes a couple of possible ways you can deploy your own STUN and TURN servers - they assume you are using Ubuntu Linux 14.04 or later. You may also use existing Internet services - free STUN servers are available (such as the Google STUN servers EdgeVPN.io is configured with by default), and pay-per-use TURN services also exist, e.g. [Xirsys](http://www.xirsys.com).

Open-source XMPP servers [eJabberd](https://www.ejabberd.im/) and [Openfire](https://www.igniterealtime.org/projects/openfire/) both offer STUN services, facilitating deployment if you are using either of these as your XMPP servers. eJabberd also offers a TURN service.

# Deploying STUN and TURN in eJabberd

eJabberd allows you to enable STUN and/or TURN through its configuration file. [Please refer to the eJabberd documentation for up-to-date instructions](https://docs.ejabberd.im/admin/configuration/listen/#ejabberd-stun-1)

# Deploying STUN in Openfire

Openfire supports STUN by adding a server plug-in. Currently, we are not aware of a TURN plug-in for Openfire. [Please refer to the Openfire documentation for up-to-date instructions](https://www.igniterealtime.org/projects/openfire/plugins/1.2.2/stunserver/readme.html)

# Deploying your own TURN server

There is an open-source package [coturn](https://github.com/coturn/coturn) you can use to install your own TURN server

Please refer to the coturn documentation to find out how to run it on the system(s) you choose to be TURN servers. 

## Running coturn on Amazon EC2

If you'd like to run coturn on an Amazon EC2 instance, [it's relatively straightforward to get started](https://medium.com/@omidborjian/setup-your-own-turn-stun-signal-relay-server-on-aws-ec2-78a8bfcb71c3). In particular, if you run an Ubuntu 18.04 AMI:

1. Setup security group incoming rules for your instance:

```
80 : TCP # if you need to setup with SSL
443 : TCP # if you need to setup with SSL
3478 : UDP
3478 : TCP
10000–20000 : UDP
```

2. Install coturn

```
sudo apt-get install coturn
```

3. Setup your configuration file

Edit /etc/turnserver.conf to configure as follows (note: set verbose for debugging only; the example sets test1 and password_test1 for a TURN user):

```
realm=yourdomain.com
fingerprint
external-ip=<ec2-public-ip-address>
listening-port=3478
min-port=10000
max-port=20000
log-file=/var/log/turnserver.log
verbose                    
user=test1:password_test1
```

3. Setup coturn service

Edit /etc/default/coturn and uncomment the following line:

```
TURNSERVER_ENABLED=1
```

4. Start the service

```bash
sudo service coturn start
```

5. Verify that turnserver is running; by default it runs on port 3478, which can be changed in the turnserver.conf file

```bash
netstat -aupn | grep 3478
```
  
6. Configure

You should now be able to [configure your EVio nodes](/configfile) to use the TURN service, by replacing the <public-ip-of-turnserver>, <username> and <password> parameters with your own (without the <, > characters). See below for an example of the LinkManager configuration snippet:

```
  "LinkManager": {
    "Dependencies": [
      "Logger",
      "TincanInterface",
      "Signal"
    ],
    "Stun": [
      "stun.l.google.com:19302",
      "stun1.l.google.com:19302"
    ],
    "Turn": [{
      "Address": "<public-ip-of-turnserver>:3478",
      "User": "<username>",
      "Password": "<password>"
     }],
    "Overlays": {
      "101000F": {
        "Type": "TUNNEL",
        "TapName": "tnl-"
      }
    }
  },
```

7. Managing users and other TURN parameters

coturn provides a rich set of parameters you can configure for your system, and several ways to manage users, including time-limited credentials. It is beyond the scope of this tutorial to cover all of these, as they will vary depending on your setup/goals - please refer to the coturn documentation

