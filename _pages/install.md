---
permalink: /install/
title: "Installing EdgeVPN.io"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# Install and Run Evio from .deb package

As of version 20.12.0, Evio can run on both amd64 platforms (the majority of commodity edge/cloud resources), as well as on armhf (edge platforms such as Raspberry Pi) with Ubuntu Linux 18.04 and 20.04. 

*NOTE* For Raspberry Pi users, *we currently only support Ubuntu 20.04 server*. Raspian does more aggressive use of DHCP and, while possible to work around it, we do not have a clean approach to supporting Raspian as of yet. If you would like to try Raspian, it requires disabling DHCP on *all* interfaces, as described at the end of this document.

## Get deb Package
Download the latest release from [the GitHub repository](https://github.com/EdgeVPNio/evio/releases/).

### Version 20.12.0

[Download the package for x86/amd64 Ubuntu nodes](https://github.com/EdgeVPNio/evio/releases/download/v20.12.0/evio_20.12.0_amd64.deb)

[Download the package for armhf Ubuntu Raspberry Pis](https://github.com/EdgeVPNio/evio/releases/download/v20.12.0/evio_20.12.0_armhf.deb)

## Install deb Package

### On x86/amd64 Ubuntu:

```shell
sudo apt install -y <path/to>/evio_*.deb
```

### On amrhf Raspberry Pi Ubuntu:

```shell
sudo apt-get install libffi-dev
sudo apt install -y <path/to>/evio_*.deb
```

## Edit Configuration File
After installation, but before starting, configure your node by editing `/etc/opt/evio/config.json`. The easiest way to get started with a working configuration is to [request a trial account](/trial). You can also use [the template from this page and add XMPP credentials, setting the IP address, and applying other configurations as needed](/configbasics) 

## Run Service
```shell
sudo systemctl start evio
``` 

Additionally, use `systemctl` to `start`/`stop`/`restart`/`status` `evio`.

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

We only support Ubuntu 20.04; if you'd like to try Evio on Raspian, until we find a reliable solution, use at your own risk. You need to disable DHCP by editing /etc/dhcpcd.conf as follows (note: the bridge names will depend on the identifier you give to your Evio network - the example below assumes 101000F):

```
noipv4ll
denyinterfaces brl101000F ovs-system
denyinterfaces edgbr101000F ovs-system
```

