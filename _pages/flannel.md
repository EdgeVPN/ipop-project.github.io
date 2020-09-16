---
permalink: /flannel/
title: "Using Flannel CNI Plugin with EdgeVPN.io"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# <i class="fas fa-cubes"></i> Introduction

One of the key use cases of EdgeVPN.io (Evio) is to enable deployment and management of Docker containers on resources both at the edge and cloud, and creating a virtual network spanning all the managed resources. Evio supports Flannel, a CNI (Container Network Interface) plug-in that seamlessly integrates with [Kubernetes](https://kubernetes.io).

This page overviews the approach to configure and setup Evio to support Kubernetes (K8s) with Flannel.

You may also check out [step-by-step instructions to deploy a demo Kubernetes cluster with Flannel over Evio from scratch](https://github.com/EdgeVPNio/edgevpnio.github.io/wiki/Demo-of-Kubernetes-using-Flannel-over-EdgeVPN.io)

# <i class="fas fa-cubes"></i> Overview

The general approach to connect K8s pods via Flannel over Evio relies on deploying Evio software on all K8s hosts in the cluster. This creates a "flat" virtual network namespace connecting all K8s hosts. Then, you need to configure Kubernetes to use the virtual network address exposed by Evio. Then, Flannel can be setup and configured as you would normally do in a local cluster. The approach is illustrated below:

![Overview of K8s Flannel plugin over Evio](/assets/images/evio-flannel-overview_3.png)

## Host overlay

The host overlay connects the K8s **hosts** together in a flat virtual network namespace. Each host gets its own virtual IP address in this namespace. In the examples below, we'll use the 10.10.0.0/16 network namespace for the host overlay. The host overlay must be setup in order for K8s hosts to join a cluster, and before the Flannel plugin works.

## Pod overlay

The pod overlay connects the K8s **pods** together in a different network namespace. This namespace is managed by Flannel, and tunneled over the Evio virtual network.

# <i class="fas fa-cubes"></i> Deploying the Evio host overlay

Please refer to the documentation on how to [install](/install) and [configure](/configbasics) Evio on your Kubernetes hosts. Currently, we have a Debian package tested on Ubuntu 18.04 distributions - and we welcome community contributions to create packages for other distros!

# <i class="fas fa-cubes"></i> Configuring Kubernetes to use the Evio host overlay

The two key steps to configure Kubernetes to use the Evio host overlay are:

* Configure kubelet to use the Evio private IP address by adding the --node-ip flag to the configuration file under KUBELET-KUBECONFIG_ARGS. In the snippet example shown below, the Evio host IP address is 10.10.100.2:

```
$ sudo cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml --node-ip=10.10.100.2"
...
```

* Retrive kube-flannel.yml file (e.g. using wget or git) from the [coreos/flannel website](https://github.com/coreos/flannel/tree/master/Documentation)

* Edit kube-flannel.yml to add the argument  - --iface=<Evio_app_bridge_name> corresponding to container command "/opt/bin/flanneld" in the kube-flannel.yml deployment file as shown in the example below. 

* Note: make sure you edit all applicable entry(ies) as there might be multiple entries under containers:

* Replace <Evio_app_bridge_name> with the name of the bridge created by Evio in your host overlay (this name is the concatenation of the "NamePrefix" and overlay ID parameters from the [Evio configuration file](/configbasics); in the example below, it is "brl101000F"  

```
 containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.12.0-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=brl101000F
        resources:
          requests:
```  

(Note: you can double-check the Evio app bridge interface (brl101000F in the example) is configured with the virtual IP address (10.10.100.2 in the example) as follows:

```
$ ifconfig
brl101000F: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1410
        inet 10.10.100.2  netmask 255.255.0.0  broadcast 0.0.0.0
        inet6 fe80::1407:73ff:fe7e:5945  prefixlen 64  scopeid 0x20<link>
        ether 16:07:73:7e:59:45  txqueuelen 1000  (Ethernet)
        RX packets 2424  bytes 227433 (227.4 KB)
        RX errors 0  dropped 1  overruns 0  frame 0
        TX packets 2093  bytes 1565945 (1.5 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```    

* Now you can deploy the Flannel plugin:

```
$ kubectl apply -f kube-flannel.yml 
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds-amd64 created
daemonset.apps/kube-flannel-ds-arm64 created
daemonset.apps/kube-flannel-ds-arm created
daemonset.apps/kube-flannel-ds-ppc64le created
daemonset.apps/kube-flannel-ds-s390x created
```
  
  
