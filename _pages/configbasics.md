---
permalink: /configbasics/
title: "EdgeVPN basic configuration"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# Introduction

The EdgeVPN.io package is released with a sample configuration file that serves as a good starting point for many use cases. This document describes basic configuration parameters that you need to configure for your deployment, as well as the typical parameters you might want to tweak for your deployment.

A simple way to get started with your first configuration is to [request an EdgeVPN trial account](/trial). You will receive a set of automatically-generated, working configuration files that you can use to test your first deployment, and use as a template moving forward.

[Please refer to this document for a full description of configuration parameteres](/configfile)

# A basic configuration file template

Before jumping into details, here is a configuration file template that is sufficient to get started with EdgeVPN.io - just replace XMPP _HostAddress_, _Username_ and _Password_ in the _Signal_ module, and _IP4_ and _NetworkAddress_ in the _BridgeController_ module: 


```
{
  "CFx": {
    "Model": "Default",
    "Overlays": [ "MyTest" ]
  },
  "Logger": {
    "LogLevel": "INFO",
    "Device": "File",
    "Directory": "/var/log/evio/",
    "CtrlLogFileName": "ctrl.log",
    "TincanLogFileName": "tincan_log",
    "MaxFileSize": 10000000,
    "MaxArchives": 2
  },
  "Signal": {
    "Overlays": {
      "MyTest": {
        "HostAddress": "A.B.C.D",
        "AuthenticationMethod": "PASSWORD",
        "Port": "5222",
        "Username": "test1@openfire.local",
        "Password": "password_test1"
      }
    }
  },
  "Topology": {
    "PeerDiscoveryCoalesce": 1,
    "Overlays": {
      "MyTest": {
        "MaxSuccessors": 2,
        "MaxOnDemandEdges": 3,
        "Role": "Switch"
      }
    }
  },
  "LinkManager": {
    "Stun": [
      "stun.l.google.com:19302",
      "stun1.l.google.com:19302"
    ],
    "Overlays": {
      "MyTest": {
        "Type": "TUNNEL",
        "TapNamePrefix": "tnl"
      }
    }
  },
  "BridgeController": {
    "BoundedFlood": {
      "LogDir": "/var/log/evio/",
      "LogFilename": "bf.log",
      "LogLevel": "INFO",
      "MaxBytes": 10000000,
      "BackupCount": 10,
      "TrafficAnalysisInterval": 10,
      "ProxyListenAddress": "",
      "ProxyListenPort": 5802,
      "Overlays": {
        "MyTest": {
          "DemandThreshold": "10M",
          "FlowIdleTimeout": 60,
          "FlowHardTimeout": 60,
          "MaxOnDemandEdges": 3
        }
      }
    },
    "Overlays": {
      "MyTest": {
        "NetDevice": {
          "AutoDelete": true,
          "Type": "OVS",
          "SwitchProtocol": "BF",
          "NamePrefix": "evi",
          "MTU": 1410,
          "AppBridge": {
            "AutoDelete": true,
            "Type": "OVS",
            "NamePrefix": "app",
            "NetworkAddress": "10.10.100.0/24",
            "MTU": 1370,
            "IP4": "10.10.100.1",
            "PrefixLen": 24
          }
        },
        "SDNController": {
          "ConnectionType": "tcp",
          "HostName": "127.0.0.1",
          "Port": "6633"
        }
      }
    }
  },
  "UsageReport": {
    "Enabled": true,
    "TimerInterval": 3600,
    "WebService": "https://qdscz6pg37.execute-api.us-west-2.amazonaws.com/default/EvioUsageReport"
  }
}
```


# Configure your XMPP server endpoint and user credentials

This is a required configuration for your deployment - you must setup every node to connect to an XMPP server. This is part of the _Signal_ module, and includes the IP address and port (typically 5222) of the server. The simplest approach uses password-based authentication, where you must add the username and password:


```
  "Signal": {
    "Overlays": {
      "MyTest": {
        "HostAddress": "A.B.C.D",
        "Port": "5222",
        "Username": "user@xmppsite.com",
        "Password": "password",
        "AuthenticationMethod": "PASSWORD"
      }
    }
  },
```

Certificate-based authentication requires additional steps to [create a CA, sign certificates, and configure the server](/openfireconfig). As far as the configuration goes, in the _Signal_ module you must specify the user's certificate and private key files. Note that the typical port for certificate-based authentication is 5223, rathar than 5222:

```
  "Signal": {
    "Overlays": {
      "MyTest": {
        "HostAddress": "A.B.C.D",
        "Port": "5223",
        "AuthenticationMethod": "x509",
        "CertDirectory": "/home/user/edgevpn/cacerts/",
        "CertFile": "edgevpn.crt",
        "KeyFile": "edgevpn.key"
      }
    }
  },
```

# Configure your bridge 

When you deploy a node, it creates an SDN Open vSwitch (OVS) in your computer; ports of this OVS are like ports of an Ethernet switch, and WebRTC TinCan links are like virtual cables that terminate on these ports through a tap interface. There are two types of deployment you may consider. 

## Bridge with IP address

The first type of deployment "patches" the OVS switch to a Linux bridge that is set up with its own IPv4 address. This is useful, for instance, when the node is a single node (physical machine, VM, container) that is added to the network. The configuration is as follows:

An example of the console output of _ip address_ of how such a deployment would look like in a node is pasted below. In this example:

* lo is the loopback interface
* eth0 is the main network interface
* ovs-system is the Open vSwitch
* eviMyTest is the EdgeVPN.io bridge (the name is a concatenation of the prefix and overlay ID from the configuration)
* appMyTest is the bridge configured with an IP address
* tnl* are the tap devices from which EdgeVPN.io packets are picked/injected from/into the virtual network

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether a2:a0:50:bf:79:d0 brd ff:ff:ff:ff:ff:ff
3: eviMyTest: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether d2:35:42:50:e9:42 brd ff:ff:ff:ff:ff:ff
4: appMyTest: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether de:40:a2:aa:aa:4a brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/16 scope global brl101000F
       valid_lft forever preferred_lft forever
5: tnla100011fffff: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc fq_codel master ovs-system state UNKNOWN group default qlen 1000
    link/ether b6:39:18:2e:60:87 brd ff:ff:ff:ff:ff:ff
8: tnla100012fffff: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc fq_codel master ovs-system state UNKNOWN group default qlen 1000
    link/ether 9e:68:ea:74:8a:ec brd ff:ff:ff:ff:ff:ff
22: eth0@if23: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

For this deployment, you need to configure the IP4 address of the node, and PrefixLen to match the subnet your virtual network uses. In the example below, we configure a 24-bit subnet, and IP4 address 10.10.100.1 for Linux bridge _app_. Additional parameters under "BoundedFlood" configure various timeouts used by the node - please refer to the complete configuration documentation for more information.

```
  "BridgeController": {
    "BoundedFlood": {
      "LogDir": "/var/log/evio/",
      "LogFilename": "bf.log",
      "LogLevel": "INFO",
      "MaxBytes": 10000000,
      "BackupCount": 10,
      "TrafficAnalysisInterval": 10,
      "ProxyListenAddress": "",
      "ProxyListenPort": 5802,
      "Overlays": {
        "MyTest": {
          "DemandThreshold": "10M",
          "FlowIdleTimeout": 60,
          "FlowHardTimeout": 60,
          "MaxOnDemandEdges": 3
        }
      }
    },
    "Overlays": {
      "MyTest": {
        "NetDevice": {
          "AutoDelete": true,
          "Type": "OVS",
          "SwitchProtocol": "BF",
          "NamePrefix": "evi",
          "MTU": 1410,
          "AppBridge": {
            "AutoDelete": true,
            "Type": "OVS",
            "NamePrefix": "app",
            "NetworkAddress": "10.10.100.0/24",
            "MTU": 1370,
            "IP4": "10.10.100.1",
            "PrefixLen": 24
          }
        },
        "SDNController": {
          "ConnectionType": "tcp",
          "HostName": "127.0.0.1",
          "Port": "6633"
        }
      }
    }
  },
```
  

# Configure NAT traversal

EdgerVPN requires at least one STUN server in order to traverse the most common types of [NATs - the "cone" type (full, address-, or port-restricted)](https://en.wikipedia.org/wiki/Network_address_translation). Unless you can be sure that all devices will be behind "cone" NATs, you will need to also use TURN server(s).

If you use STUN only, you will be able to use existing, freely-available STUN servers on the Internet (see example below), or deploy your own STUN server(s). If you plan to use TURN as well, you need to either deploy and manage your own TURN server (e.g. on a cloud provider such as Amazon EC2), or use a TURN service. [This document provides information on how to deploy open-source STUN/TURN services](/stunturn), including on EC2. Commercial TURN-as-a-service options are also an option; for example, the software has been tested and works with [Xirsys](http://www.xirsys.com).

The setup in the configuration file is simple. An example with STUN only, configured with a list of two freely-available Google STUN servers (you may add your own STUN servers to this list if you wish):

```
  "LinkManager": {
    "Stun": [
      "stun.l.google.com:19302",
      "stun1.l.google.com:19302"
    ],
    "Overlays": {
      "MyTest": {
        "Type": "TUNNEL",
        "TapNamePrefix": "tnl"
      }
    }
  },
```

An example with a TURN server added (substitute _Address_, _User_ and _Password_ appropriately for your server):

```
  "LinkManager": {
    "Stun": [
      "stun.l.google.com:19302",
      "stun1.l.google.com:19302"
    ],
    "Turn": [{
      "Address": "A.B.C.D:3478",
      "User": "turnuser",
      "Password": "turnpassword"
     }],
    "Overlays": {
      "MyTest": {
        "Type": "TUNNEL",
        "TapNamePrefix": "tnl"
      }
    }
  },
```

# Configure P2P Topology

The structured peer-to-peer topology used in your deployment can be configured under "Topology", The key parameters you may modify are MaxSuccessors (maximum number of immediate successors a node has in the ring) and MaxOnDemandEdges (maximum number of on-demand edges that can be created in response to large flows measured between two nodes). 

_Note_ MaxOnDemandEdges needs to match in both Topology and BridgeController-BoundedFlood. Setting the value to 0 disables on-demand


