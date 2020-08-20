---
layout: splash-home
permalink: /
title: "Open-source VPN for Edge Computing"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
  actions:
    - label: "<i class='fas fa-download'></i> Latest Release: EdgeVPNio 20.7.2, July 2020"
      url: "/releases/"
excerpt: "Seamlessly connect edge resources with a scalable virtual Ethernet"
---

{::nomarkdown}<div class="background-white"><div class="center">{:/nomarkdown}
### <i class="fas fa-cubes"></i> EdgeVPN.io is an open-source software for deploying scalable VPNs across distributed edge resources 

It groups distributed nodes into a logical Ethernet. EdgeVPN.io (EVio, for short) has built-in support for packet capture, encryption, tunneling, forwarding, and NAT traversal, It builds on standard protocols, flexible software-defined networking, and a scalable overlay network architecture suitable for containerized applications



**Scalable and self-configuring:** EdgeVPN.io is designed based on principles used in scalable, ring-structured key/value stores and peer-to-peer overlays, there are no central VPN traffic bottlenecks, and the network scales and configures itself as nodes are added/removed


**Run existing software:** EVio exposes virtual Ethernet network interfaces and private IP addresses allowing existing and future Ethernet/IP based software stacks to run, unmodified. It can provide a virtual cluster of Docker containers across edge/cloud resources. 


**Deploy anywhere:** EVio transparently connects edge devices with private addresses behind Network Address Translators (NATs) and firewalls, as well as cloud computing resources, encrypting and tunneling traffic peer-to-peer across the Internet
{::nomarkdown}</div></div>{:/nomarkdown}


{::nomarkdown}<div class="background-grey"><div class="center">{:/nomarkdown}
### <i class="fas fa-cubes"></i> Key features

**Structured topology:** 
EdgeVPN.io implements a structured peer-to-peer overlay topology where nodes self-organize into a ring ordered by unique node IDs, and with randomly-assigned “long-distance” links, based on the approach described in [Symphony](http://infolab.stanford.edu/~bawa/Pub/symphony.pdf). *This topology is scalable:* the average distance between two nodes can scale as a log(N) function, where N is the number of nodes. Topology handling is modular, such that other topologies can be implemented.

**Encrypted links:**
EVio links are encrypted and authenticated with standard SSL-based transport-layer security implemented by the open-source [WebRTC framework](https://webrtc.org/). *Communication among nodes is private:* links use UDP-based Datagram TLS (DTLS) over NAT hole-punched tunnels

**Easy grouping:**
EVio uses the standard [XMPP protocol](https://xmpp.org/) with short messages to discover and exchange connection information with peers. While packet switching and routing is decentralized and based on a scalable P2P overlay, *membership can be managed centrally:*  Device groups can be easily configured for networks small to large in an XMPP server, such as the open-source [OpenFire](https://www.igniterealtime.org/projects/openfire/) and [eJabberd](https://www.ejabberd.im/) servers

**Layer-2 virtual network:**
EVio exposes a virtual Ethernet to its endpoints, and supports the ARP protocol, and unicast and IGMP-based multicast IP applications. *You can run existing IP-based applications over EdgeVPN without modifications*

**Programmable and extensible:**
The core packet-switching in EdgeVPN.io is programmable, using the [OpenFlow protocol for Software-Defined Networking](https://www.opennetworking.org/). EVio integrates with [Open vSwitch](https://www.openvswitch.org/) virtual switches, and endpoint interfaces are exposed via a virtual tap device. EVio can be deployed on physical and virtual machines, and in [Docker](https://www.docker.com/) containers. 

**Integration with Kubernetes:**
Integration with [Kubernetes](https://kubernetes.io) is supported with either [Flannel or a custom Evio CNI plug-in](/kubernetes).

**Built on standards:**
EdgeVPN.io leverages standards for NAT traversal ([ICE](https://tools.ietf.org/html/rfc5245) with STUN and TURN), transport-layer security (TLS, DTLS), software-defined networking (OpenFlow), and short messaging (XMPP), and reuses the WebRTC open-source framework 
{::nomarkdown}</div></div>{:/nomarkdown}
