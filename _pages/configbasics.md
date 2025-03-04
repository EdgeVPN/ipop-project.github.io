---
permalink: /configbasics/
title: "EdgeVPN basic configuration"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# Introduction

The EdgeVPN.io package is released with a sample configuration file that serves as a good starting point for many use cases. This document describes basic configuration parameters that you need to configure for your deployment, as well as the typical parameters you might want to tweak for your deployment.

Note that the configuration file format has changed starting in the March 2023 release. The changes are relatively minor, but not backwards-compatible with previous versions. If you are updating Evio from a previous version, please update your configuration files accordingly.

A simple way to get started with your first configuration is to [request an EdgeVPN trial account](/trial). You will receive a set of automatically-generated, working configuration files that you can use to test your first deployment, and use as a template moving forward.

[Please refer to this document for a full description of configuration parameteres](/configfile0223)

# A basic configuration file template

Before jumping into details, here is a configuration file template that is sufficient to get started with EdgeVPN.io - just replace XMPP _HostAddress_, _Username_ and _Password_ in the _Signal_ module, and _IP4_ and _NetworkAddress_ in the _BridgeController_ module: 


```
{
    "Broker": {
        "Overlays": [
            "MyTest"
        ],
        "Controllers": {
            "UsageReport": {
                "Enabled": true,
                "Module": "usage_report",
                "Dependencies": [
                    "Topology"
                ],
                "TimerInterval": 3600
            }
        }
    },
    "Signal": {
        "Overlays": {
            "MyTest": {
                "HostAddress": "your.xmppsite.com",
                "Port": "5222",
                "Username": "eviouser_N@your.xmppsite.com",
                "Password": "xmpp_password"
            }
        }
    },
    "LinkManager": {
        "Stun": [
            "stun.l.google.com:19302",
            "stun1.l.google.com:19302"
        ],
        "Turn": [
            {
                "Address": "your.turnsite.com:3478",
                "User": "evioturnuser_N",
                "Password": "turn_password"
            }
        ],
        "IgnoredNetInterfaces": [
                    "ovs-system"
                ]
    },
    "BridgeController": {
        "BoundedFlood": {
            "Overlays": {
                "MyTest": {}
            }
        },
        "Overlays": {
            "MyTest": {
                "NetDevice": {
                    "MTU": 1410,
                    "AppBridge": {
                        "MTU": 1350,
                        "IP4": "10.10.100.15",
                        "PrefixLen": 24
                    }
                }
            }
        }
    }    
}
```


# Configure your XMPP server endpoint and user credentials

This is a required configuration for your deployment - you must setup every node to connect to an XMPP server. 
This is part of the _Signal_ module, and includes the domain name (or IP address) and port (typically 5222) of the XMPP server. 
The simplest approach uses password-based authentication, where you must add the username and password. For information on how to configure x509-based authentication, please refer to the detailed configuration document.
In the example below, the XMPP server listens on port 5222 of server your.xmppsite.com, and the Evio node authenticates to the XMPP server with username eviouser_N a password xmpp_password.



```
    "Signal": {
        "Overlays": {
            "MyTest": {
                "HostAddress": "your.xmppsite.com",
                "Port": "5222",
                "Username": "eviouser_N@your.xmppsite.com",
                "Password": "xmpp_password"
            }
        }
    },
```

# Configure your bridge 

When you deploy a node, it creates an SDN Open vSwitch (OVS) in your computer; ports of this OVS are like ports of an Ethernet switch, and WebRTC TinCan links are like virtual cables that terminate on these ports through a tap interface. 
There are multiple types of deployment you may consider; we'll focus on the most recommended approach, which is to deploy Evio in a Docker container and "patch" the OVS switch to a Linux bridge that is set up with its own IPv4 address.
For this deployment, you need to configure the IP4 address of the node, and PrefixLen to match the subnet your virtual network uses. 
In the example below, we configure a 24-bit subnet, and IP4 address 10.10.100.15 for Linux bridge _app_, and MTU values that we have found to work in most deployments (you may need to tweak them depending on your environment).
Additional parameters under "BoundedFlood" configure various timeouts used by the node - please refer to the complete configuration documentation for more information.

```
    "BridgeController": {
        "BoundedFlood": {
            "Overlays": {
                "MyTest": {}
            }
        },
        "Overlays": {
            "MyTest": {
                "NetDevice": {
                    "MTU": 1410,
                    "AppBridge": {
                        "MTU": 1350,
                        "IP4": "10.10.100.15",
                        "PrefixLen": 24
                    }
                }
            }
        }
    }    
```


# Configure NAT traversal

EdgeVPN requires at least one STUN server in order to traverse the most common types of [NATs - the "cone" type (full, address-, or port-restricted)](https://en.wikipedia.org/wiki/Network_address_translation). Unless you can be sure that all devices will be behind "cone" NATs, you will need to also use TURN server(s).

If you use STUN only, you will be able to use existing, freely-available STUN servers on the Internet (see example below), or deploy your own STUN server(s). If you plan to use TURN as well, you need to either deploy and manage your own TURN server (e.g. on a cloud provider such as Amazon EC2), or use a TURN service. [This document provides information on how to deploy open-source STUN/TURN services](/stunturn), including on EC2. Commercial TURN-as-a-service options are also an option; for example, the software has been tested and works with [Xirsys](http://www.xirsys.com).

The setup in the configuration file is simple. An example with STUN, configured with a list of two freely-available Google STUN servers, and a fictitional TURN server.

You may add more STUN and TURN servers to this list if you wish:

```
    "LinkManager": {
        "Stun": [
            "stun.l.google.com:19302",
            "stun1.l.google.com:19302"
        ],
        "Turn": [
            {
                "Address": "your.turnsite.com:3478",
                "User": "evioturnuser_N",
                "Password": "turn_password"
            }
        ],
```

# Configure interfaces to ignore

If you have multiple interfaces in your computer, you want to configure EdgeVPN to ignore any interfaces that it is not supposed to use for tunneling. 
This is configured as a list:

```
        "IgnoredNetInterfaces": [
                    "ovs-system"
                ]
```                
