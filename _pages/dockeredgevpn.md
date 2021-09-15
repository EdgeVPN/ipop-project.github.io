---
permalink: /dockeredgevpn/
title: "Deploy EdgeVPN.io with Docker"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

## Introduction

This tutorial guides you through the process of deploying two EdgeVPN.io nodes using Docker

## Dependences

The tutorial assumes you have an XMPP server up and running, with user accounts and a group setup, [as described in the tutorial on deploying an XMPP server using Docker](/openfiredocker)

## Configure host network and directories

First, you need to install Docker and Open vSwitch and create a network namespace in the Docker host:

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

**Note: Make sure you log out and back in again so the docker group addition is in effect

**Note: If you are using Ubuntu 20.04, start OVS manually:

```
sudo systemctl start ovs-vswitchd.service
```

Now, create directories in the host to hold configuration files and logs for your containers:

```
cd
mkdir evio
cd evio
mkdir config
mkdir logs
mkdir logs/001
mkdir logs/002
```

## Setup configuration files

The simplest way to get started is by [requesting a trial account](/trial) and following the configuration file template that you receive once your request is processed. In the example here, we use config-001.json and config-002.json to deploy two nodes.


## Start the containers

Now you will run two containers, named evio001 and evio002, mapping the different configuration file and the log directories to different mount points. **Note:** the examples below use the dkrnet network and Docker NAT. This requires TURN if you connect multiple hosts. If you plan to run a single container in your host, it's advisable you use the host's network instead, by replacing _dkrnet_ by _host_ below.

### Instructions for Evio 20.12.1 and above:

```
docker run -d -v /home/$USER/evio/config/config-001.json:/etc/opt/evio/config.json -v /home/$USER/evio/logs/001:/var/log/evio/ --rm --privileged --name evio001 --network dkrnet edgevpnio/evio-node:21.9.0 /sbin/init

docker run -d -v /home/$USER/evio/config/config-002.json:/etc/opt/evio/config.json -v /home/$USER/evio/logs/002:/var/log/evio/ --rm --privileged --name evio002 --network dkrnet edgevpnio/evio-node:21.9.0 /sbin/init
```


## Test your connection

You can open a shell into the container evio001 (virtual IP address 10.10.100.1), and ping the evio002 node (virtual IP 10.10.100.2):

```
docker exec -it evio001 /bin/bash
# ping 10.10.100.2
```

Or, the other way around:

```
docker exec -it evio002 /bin/bash
# ping 10.10.100.1
```



