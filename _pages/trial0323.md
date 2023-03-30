---
permalink: /trial0323/
title: "Evio trial accounts"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# Introduction

The EdgeVPN.io team is hosting an XMPP+TURN service in the cloud to help new users get started using the software. 
With this service, we provide prospective new users with configuration files that allow a plug-and-play deployment of Evio nodes using Docker containers for Linux on x86 (amd64) and arm64.

To support ramping-up many new users, the trial accounts are limited to five nodes and expire in a 2-week period; these may change depending on user demand. 


## Step 1: request accounts and configuration file

[Click here enter your request](https://forms.gle/2TTrt9nV32pFAHbb9); we will review and contact you by email to provide you with configuration files. The information we ask for is:

* Contact email
* Project name
* Brief project summary

## Step 2: Deploy nodes

Once you receive the configuration files, the easiest way to get started is to run Evio Docker containers, using the configuration files you received.
In the example below, we assume that:
1) you already have Docker installed on two computers, 
2) you are using an account that can run Docker containers in --privileged mode,
3) you received trial account configuration files configured with Evio IP addresses 10.10.100.1 and 10.10.100.2 and
4) you repeat the steps below in both computers, each

On both computers 1 and 2:
You can pull the Evio Docker image using:
```
docker pull edgevpnio/evio-node:latest
```

On computer 1:
Then, copy the trial account configuration file config-001.json you have received to ~/.evio/config.json and start the container: 
```
cp config-001.json ~/.evio/config.json
docker run -d -v /home/$USER/.evio/config.json:/etc/opt/evio/config.json -v /var/log/evio/:/var/log/evio/ --restart always --privileged --name evio-node --network host edgevpnio/evio-node:latest
```

On computer 2:
Copy the trial account configuration file config-002.json you have received to ~/.evio/config.json and start the container: 
```
cp config-002.json ~/.evio/config.json
docker run -d -v /home/$USER/.evio/config.json:/etc/opt/evio/config.json -v /var/log/evio/:/var/log/evio/ --restart always --privileged --name evio-node --network host edgevpnio/evio-node:latest
```

## Step 3: Test it out

Now try simple ping test from computer 1 to computer 2:

```
# ping 10.10.100.2
```


