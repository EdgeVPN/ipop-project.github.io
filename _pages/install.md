---
permalink: /install/
title: "Installing EdgeVPN.io"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# Install and Run Evio from .deb package

As of version 21.9.0, Evio can run on both amd64 platforms (the majority of commodity edge/cloud resources), as well as on armhf and arm64 (edge platforms such as Raspberry Pi) with Ubuntu Linux 18.04 and 20.04. 

*NOTE* For Raspberry Pi users, *we currently only support Ubuntu 20.04 server*. Among other issues, Raspian does more aggressive use of DHCP and, while in principle it should be possible to work around it, we do not have a clean approach to supporting Raspian as of yet. If you would like to try Raspian, it requires disabling DHCP on *all* interfaces, as described at the end of this document.

## Install deb Package

*Note:* The arm64 package has been tested in Raspberry Pi, Amazon ARM64 instances, and on nVidia Jetson devices. The stock nVidia Jetson Linux kernel, however, does not come with proper dependences installed for Open vSwitch; [follow the instructions to build a custom kernel](/jetson) or contact us if you're intested in improving the process of porting for Jetson devices.

Before installing the package, you need to add the evio repository to your node - this step needs only be done once for a host:

```shell
sudo bash
# echo "deb [trusted=yes] https://apt.fury.io/evio/ * *" > /etc/apt/sources.list.d/fury.list
# apt update
```

To install the package:

```shell
# apt install evio
```

If the installation fails due to the libffi-dev dependence, you might need to add that manually:

```shell
# apt-get install libffi-dev
# apt install evio
```

## Edit Configuration File
After installation, but before starting, configure your node by editing `/etc/opt/evio/config.json`. The easiest way to get started with a working configuration is to [request a trial account](/trial). You can also use [the template from this page and add XMPP credentials, setting the IP address, and applying other configurations as needed](/configbasics) 

## Run Service

You may use `systemctl` to `start`/`stop`/`restart`/`status` `evio`.

## Dependencies
The installer has dependencies on, and will install `python3 (>=3.8)`, `python3-dev (>=3.8)`,  `python3-pip`, `iproute2`, `bridge-utils`.


## Related Files and Directories
By default, the following files and directories are created:
1. `/opt/evio/tincan`
2. `/opt/evio/controller/`
3. `/etc/opt/evio/config.json`

## Disabling or removing the software

### To disable Start on Boot
```shell
sudo systemctl disable evio
```

### To remove the package
```shell
sudo apt remove -y evio
```

### Disabling DHCP in Raspian-based Raspberry Pis

We only support Ubuntu 20.04; if you'd like to try Evio on Raspian, until we find a reliable solution, use at your own risk. You need to disable DHCP by editing /etc/dhcpcd.conf as follows (note: the bridge names will depend on the prefix names and overlay name you give to your Evio network - the example below assumes an overlay 101000F with prefixes app and evi for the App and Evio bridges, respectively):

```
noipv4ll
denyinterfaces app101000F ovs-system
denyinterfaces evi101000F ovs-system
```


