---
permalink: /install/
title: "Installing EdgeVPN.io"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# Install and Run Evio from .deb package

As of version 23.3.0, Evio can run on both amd64 platforms (the majority of commodity edge/cloud resources), as well as on armhf and arm64 (edge platforms such as Raspberry Pi) with Ubuntu Linux 18.04 and 20.04. 

*NOTE* For Raspberry Pi users, *we currently only support Ubuntu 20.04 server*. Among other issues, Raspian does more aggressive use of DHCP and, while in principle it should be possible to work around it, we do not have a clean approach to supporting Raspian as of yet. If you would like to try Raspian, it requires disabling DHCP on *all* interfaces, as described at the end of this document.

## Install deb Package

*Note:* The arm64 package has been tested in Ubuntu 20.04 Raspberry Pi, Amazon and Oracle Cloud ARM64 instances

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
The installer has dependencies on, and will install `python3 (>=3.9)`, `python3-dev (>=3.9)`,  `python3-pip`, `iproute2`, `bridge-utils`.


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



