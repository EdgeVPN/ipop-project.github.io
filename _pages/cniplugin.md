---
permalink: /cniplugin/
title: "EdgeVPN.io Kubernetes CNI Plugin"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# <i class="fas fa-cubes"></i> Introduction

One of the key use cases of EdgeVPN.io is to enable deployment and management of Docker containers on resources both at the edge and cloud, and creating a virtual network spanning all the managed resources. To this end, Evio offers a custom CNI (Container Network Interface) plug-in supports seamless integration with [Kubernetes](https://kubernetes.io) and avoids the extra encapsulation overhead introduced by Flannel.

Inter-node communication between pods on Kubernetes is enabled by networking infrastructure set up by CNI plugins. This page overviews the architecture and setup of the Evio CNI plugin enabling Kubernetes hosts and pods to be connected by a virtual network - regardless of whether they reside in distributed edge resources with private addresses and behind NATs, where inter-node pod traffic is carried across authenticated, encrypted virtual network tunnels between nodes part of the Kubernetes cluster. 

# <i class="fas fa-cubes"></i> Overview

The general approach to integrate Evio with Kubernetes relies on two overlays, which can be deployed as a single instance of the Evio service on each k8s cluster node. 

## Host overlay

The host overlay connects the k8s **hosts** together in a flat virtual network namespace. Each host gets its own virtual IP address in this namespace. In the examples below, we'll use the 10.10.0.0/16 network namespace for the host overlay. The host overlay must be setup in order for k8s hosts to join a cluster, and before the CNI plugin works.

## Pod overlay

The pod overlay connects the k8s **pods** together in a different network namespace. This namespace is partitioned into separate subnets, one subnet per k8s host. Each pod gets its own virtual IP address in the subnet assigned to the host. In the examples below, we'll use the 10.244.0.0/16 network namespace for the pod overlay and 8-bit subnets for each host. For instance, pods in host A might be assigned addresses in the 10.244.__1__.2-10.244.__1__.254 range, and in host B in the 10.244.__2__.2-10.244.__2__.254 range, and so on.

The following figure illustrates; in the figure, two hosts on separate edge networks (with different NATs) are connected by an Evio tunnel. In edge host 1, the host is assigned an Evio virtual address 10.10.0.1, while host 2 is assigned 10.10.0.2. The two hosts use this host overlay interface to communicate (e.g. if host 1 is the kubeadm node, host 2 uses this interface to join the cluster). Pods deployed on hosts 1, 2 are assigned addresses on the pod overlay (e.g. in the figure below, pod 10.244.1.2 at the top left in host 1, pod 10.244.2.3 top right in host 2). Both overlays are tunneled through Evio. Note that the host IP addresses are not relevant - they could be private addresses on different namespaces; Evio's NAT traversal capabilities allow establishing tunnels that bootstrap the host/pod overlays with their own virtual address namespaces.

![Overview of Evio plug-in for Kubernetes](/assets/images/evio-cni-overview_3.png)

## Evio CNI plugin architecture

The figure below zooms in to a host, highlighting how the Evio CNI plug-in operates. The architecture is based on the Bridge CNI plugin, which is extended by incorporating support for OVS (Open vSwitch) SDN bridges. It has two major modules/executables: the CNI plugin, and the CNI configuration generator (CNI-CONFIG-GEN in the figure). Both of these executables are deployed as a daemon set on the Kubernetes cluster, where they run on every participating node.

![Overview of EdgeVPN.io CNI plug-in](/assets/images/evio-cni-host_2.png)

### CNI Plugin

The key functionality implemented by the Evio plugin consists of:

* Check if the Evio OVS bridge is up; if not, set up the bridge
* Configure the Evio OVS bridge with the gateway address (10.244.3.1 in the figure) and bring it up
* Set up a Veth interface pair for every pod, with the host end attached to ports of the Evio OVS bridge. This, in effect, "plugs in" the pod to the Evio-controlled switched, and allows packets to flow among pods through the Evio encrypted tunnels
* Configure the pod network namespace with an IP address and set up routes and IP Masq. This functionality is delegated to the IPAM host-local plugin.

### CNI config gen

CNI config gen runs as an Init Container and generates a configuration file which specifies the CNI plugin to be picked up by Kubelet. The Evio CNI plugin carries out the actual task of setting up the network.

Two key aspects of networking need to be configured by the user to drive this module:

* PodCIDR: the subnet for the pod network, e.g. 10.244.0.0/16
* NodeBits: number of bits to identify the maximum number of allowed nodes in the cluster

To illustrate: if NodeBits is 8, a total of 8 bits will be reserved for node identification and the cluster can have 2^8 = 256 nodes. Assuming a 16-bit namespace for PodCIDR (e.g. 10.244.0.0/16), this leaves us with 32-16-8=8 bits for allocating addresses for pods on a given node. CNI config gen will use this information to generate the address range for all pods instantiated on a given node (e.g. 10.244.3.2 in the figure); with 8 bits, there are a total of 253 pod addresses per host (.0 and .255 are not used, and .1 is reserved to the Evio OVS pod bridge on the host)

CNI config gen keeps track of the address range of pods allocated to each node by leveraging Kubernete's Etcd database. When a new node joins the cluster, it generates a configuration file with a non-conflicting address range for pods on this node, and saves it in the default "/etc/cni/net.d" directory.

