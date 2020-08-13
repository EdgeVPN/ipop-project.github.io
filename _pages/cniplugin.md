---
permalink: /cniplugin/
title: "CNI Plugin for k8s integration"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# <i class="fas fa-cubes"></i> Introduction

One of the key use cases of EdgeVPN.io is to enable deployment and management of Docker containers on resources both at the edge and cloud, and creating a virtual network spanning all the managed resources. To this end, Evio has a CNI (Container Network Interface) plug-in that allows for seamless integration with Kubernetes.

Inter-node communication between pods on Kubernetes is enabled by networking infrastructure set up by CNI plugins. This page overviews the architecture and setup of the Evio CNI plugin enabling Kubernetes hosts and pods to be connected by a virtual network - regardless of whether they reside in distributed edge resources with private addresses and behind NATs, where inter-node pod traffic is carried across authenticated, encrypted virtual network tunnels between nodes part of the Kubernetes cluster. 

# <i class="fas fa-cubes"></i> Overview

The general approach to integrate Evio with Kubernetes relies on two overlays, which can be deployed as a single instance of the Evio service on each k8s cluster node:

## Host overlay

The host overlay connects the k8s **hosts** together in a flat virtual network namespace. Each host gets its own virtual IP address in this namespace. In the examples below, we'll use the 10.10.0.0/16 network namespace for the host overlay. The host overlay must be setup in order for k8s hosts to join a cluster, and before the CNI plugin works.

## Pod overlay

The pod overlay connects the k8s **pods** together in a different network namespace. This namespace is partitioned into separate subnets, one subnet per k8s host. Each pod gets its own virtual IP address in the subnet assigned to the host. In the examples below, we'll use the 10.244.0.0/16 network namespace for the pod overlay and 8-bit subnets for each host. For instance, pods in host A might be assigned addresses in the 10.244.**1**.0-10.244.**1**.254 range, and in host B in the 10.244.**2**.0-10.244.**2**.254 range, and so on.

