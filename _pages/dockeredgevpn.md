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

First, let's create a configuration file for a container that uses XMPP user test1. You may change the name of the user below.

Copy and save this as /home/$USER/evio/config/config-001.json (the directory you created in the previous step) - *make sure you replace A.B.C.D with the IP address of your XMPP host:*

```json
{
  "CFx": {
    "Overlays": [ "MyTest" ]
  },
  "Logger": {
    "LogLevel": "DEBUG",
    "Directory": "/var/log/evio/"
  },
  "Signal": {
    "Overlays": {
      "MyTest": {
        "HostAddress": "A.B.C.D",
        "Port": "5222",
        "Username": "test1@openfire.local",
        "Password": "password_test1",
        "AuthenticationMethod": "PASSWORD"
      }
    }
  },
  "Topology": {
    "Overlays": {
      "MyTest": {
        "MaxSuccessors": 2,
        "MaxOnDemandEdges": 3,
        "Role": "Switch"
      }
    }
  },
  "LinkManager": {
    "Stun": [ "stun.l.google.com:19302", "stun1.l.google.com:19302" ],
    "Overlays": {
      "MyTest": {
        "Type": "TUNNEL",
        "TapName": "tnl-"
      }
    }
  },
  "UsageReport": {
    "Enabled": true,
    "TimerInterval": 3600,
    "WebService": "https://qdscz6pg37.execute-api.us-west-2.amazonaws.com/default/EvioUsageReport"
  },
  "BridgeController": {
    "BoundedFlood": {
      "OverlayId": "MyTest",
      "LogDir": "/var/log/evio/",
      "LogFilename": "bf.log",
      "LogLevel": "DEBUG",
      "BridgeName": "evio",
      "DemandThreshold": "100M",
      "FlowIdleTimeout": 60,
      "FlowHardTimeout": 60,
      "MulticastBroadcastInterval": 60,
      "MaxBytes": 10000000,
      "BackupCount": 2,
      "ProxyListenAddress": "",
      "ProxyListenPort": 5802,
      "MonitorInterval": 60,
      "MaxOnDemandEdges": 0
    },
    "Overlays": {
      "MyTest": {
        "NetDevice": {
          "AutoDelete": true,
          "Type": "OVS",
          "SwitchProtocol": "BF",
          "NamePrefix": "evio",
          "MTU": 1410,
          "AppBridge": {
            "AutoDelete": true,
            "Type": "OVS",
            "NamePrefix": "appbr",
            "IP4": "10.10.10.21",
            "PrefixLen": 24,
            "MTU": 1410
          }
        },
        "SDNController": {
          "ConnectionType": "tcp",
          "HostName": "127.0.0.1",
          "Port": "6633"
        }
      }
    }
  }
}
```

To configure the second container, copy config-001.json to config-002.json, **and replace the following entry in the json file** to set up another EdgeVPN.io IP address:

```
        "IP4": "10.10.10.22",
```

## Start the containers

Now you will run two containers, named evio001 and evio002, mapping the different configuration file and the log directories to different mount points. **Note:** the examples below use the dkrnet network and Docker NAT. This requires TURN if you connect multiple hosts. If you plan to run a single container in your host, it's advisable you use the host's network instead, by replacing _dkrnet_ by _host_ below.

### Instructions for Evio 20.12.1 and above:

*NOTE* Evio versions 20.12.0+ have moved the configuration file location to /etc/opt/evio:

```
docker run -d -v /home/$USER/evio/config/config-001.json:/etc/opt/evio/config.json -v /home/$USER/evio/logs/001:/var/log/evio/ --rm --privileged --name evio001 --network dkrnet edgevpnio/evio-node:20.12.2 /sbin/init

docker run -d -v /home/$USER/evio/config/config-002.json:/etc/opt/evio/config.json -v /home/$USER/evio/logs/002:/var/log/evio/ --rm --privileged --name evio002 --network dkrnet edgevpnio/evio-node:20.12.2 /sbin/init
```


## Test your connection

You can open a shell into the container evio001 (virtual IP address 10.10.10.21), and ping the evio002 node (virtual IP 10.10.10.22):

```
docker exec -it evio001 /bin/bash
# ping 10.10.10.22
```

Or, the other way around:

```
docker exec -it evio002 /bin/bash
# ping 10.10.10.21
```



