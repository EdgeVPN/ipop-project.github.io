---
permalink: /trial/
title: "Evio trial accounts"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# Introduction

The EdgeVPN.io team is hosting an XMPP+TURN service in the cloud to help new users get started using the software. With this service, we provide prospective new users with configuration files that allow a plug-and-play deployment of Evio nodes using Docker containers for x86/Linux.

To support ramping-up many new users, the trial accounts are limited to five nodes and expire in a 2-week period; these may change depending on user demand. 


## Step 1: request accounts and configuration file

[Click here enter your request](https://forms.gle/2TTrt9nV32pFAHbb9); we will review and contact you by email to provide you with configuration files. The information we ask for is:

* Contact email
* Project name
* Brief project summary

## Step 2: Deploy nodes

Once you receive the configuration files, the easiest way to get started is to run Evio Docker containers in an x86/Linux computer, using the configuration files you received.

To install Docker:

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt-get update -y
sudo apt-get install -y openvswitch-switch \
                        python3 python3-pip python3-venv \
                        apt-transport-https \
                        ca-certificates \
                        curl git \
                        software-properties-common \
                        containerd.io \
                        docker-ce-cli \
                        docker-ce 
sudo groupadd -f docker
sudo usermod -a -G docker $USER
sudo docker network create dkrnet
```

Logout and login so that group changes are in effect.

**Note: If you are using Ubuntu 20.04, start OVS manually:

```
sudo systemctl start ovs-vswitchd.service
```

If you already have Docker installed, you just need to create a Docker network, and use the -v Docker option to mount the configuration file and log directory.

You can start the container and bind to the dkrnet network as follows. Alternatively, you can replace _dkrnet_ with _host_ below to use Docker's host networking - if you only run a single container in your host.

```
docker run -d -v /path_to/config-001.json:/etc/opt/evio/config.json -v /path_to/logs/001:/var/log/edge-vpnio/ --rm --privileged --name evio001 --network dkrnet edgevpnio/evio-node:20.12.0 /sbin/init
```

## Step 3: Test it out

By default, the files configure node addresses in the 10.10.100.0/24 subnet, and 10.10.100.1 - 10.10.100.5 range. A simple ping test:

```
docker exec -it evio001 /bin/bash
# ping 10.10.100.2
```


