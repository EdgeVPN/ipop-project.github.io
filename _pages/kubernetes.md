---
permalink: /kubernetes/
title: "Edge computing with Kubernetes and EdgeVPN.io"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# <i class="fas fa-cubes"></i> Introduction

EdgeVPN.io provides a virtual network foundation for the deployment of various unmodified platforms and applications. A widely-used platform for the deployment of container-based services is [Kubernetes](https://kubernetes.io). 

Kubernetes is often deployed in cloud data centers, where the assumption is that all nodes are in the same address space - i.e., there are no NATs between Kubernetes hosts/pods. However, when deploying workloads across multiple _edge_ networks, this is seldom the case - nodes may be served by different providers, and be assigned private, NATed addresses. 

Enter EdgeVPN.io - it provides a foundational virtual network layer that _exposes the networking model that Kubernetes requires_, essentially presenting Kubernetes daemons with an enviroment that is logically the same as they would encounter in a data center. Thus, EdgeVPN.io allows deployments across multiple disparate cloud/edge networks, where private addresses, NATs and firewalls are not uncommon - _without any changes_ - thus greatly improving the ability to deploy and management workloads that span cloud and edge resources and support "fog" computing for processing near IoT sensors/actuators.

# <i class="fas fa-cubes"></i> Deployment modes

There are two different ways EdgeVPN.io (Evio) can be used to support Kubernetes (K8s) deployments, both of which rely on the use of [CNI plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#cni):

* Flannel: The Flannel CNI plugin is used in many K8s deployments. Like Evio itself, Flannel creates an _overlay network_ that exposes a virtual network namespace to K8s pods and uses encapsulation to tunnel messages between pods. Unlike Evio, however, Flannel _does not support NAT traversal_. Flannel can, however, leverage Evio's NAT traversal and virtualization - it's possible to deploy a Flannel overlay _atop the Evio overlay_:

![K8s with Flannel CNI plugin over Evio](/assets/images/evio-flannel-overview_3.png)

* Evio CNI plugin: While Flannel works unmodified atop an Evio overlay, there is a performance price that is paid: double-encapsulation. In essence, messages sent among pods are encapsulated twice (by Flannel, and by Evio). To address this drawback, Evio has its own CNI plugin, which allows messages to be encapsulated only once - by Evio:

![K8s with Evio CNI plugin](/assets/images/evio-evio-overview_3.png)

# <i class="fas fa-cubes"></i> Choosing a deployment mode

Both approaches outlined above - Flannel and Evio CNI plugins - are possible. Which one is right for you? 

* If you are already a user of Flannel, or if would like to test Evio for K8s for the first time, the Flannel approach is the simplest to deploy; [follow the documentation on using Evio with Flannel](/flannel) for more information on how to configure and deploy Evio for K8s.

* If you want to extract the best performance and avoid the double-encapsulation overhead, [follow the documentation on deploying the Evio plugin](/cniplugin) for more information on how to configure and deploy Evio and the CNI plugin for K8s.
