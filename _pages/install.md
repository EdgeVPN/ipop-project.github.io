---
permalink: /install/
title: "Installing EdgeVPN.io"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# Install and Run Evio from .deb package

As of version 20.12.0, Evio can run on both amd64 platforms (the majority of commodity edge/cloud resources), as well as on armhf (edge platforms such as Raspberry Pi) with Ubuntu Linux 18.04 and 20.04. 

*NOTE* For Raspberry Pi users, we strongly recommend running Ubuntu 20.04 server. Raspian installation is more cumbersome, and we focus our support on Ubuntu 20.04.

## Get deb Package
Download the latest release from [the GitHub repository](https://github.com/EdgeVPNio/evio/releases/).

### Version 20.12.0

[amd64 package](https://github.com/EdgeVPNio/evio/releases/download/v20.12.0/evio_20.12.0_amd64.deb)
[armhf package](https://github.com/EdgeVPNio/evio/releases/download/v20.12.0/evio_20.12.0_armhf.deb)

## Install deb Package

```shell
sudo apt install -y <path/to>/evio_*.deb
```

*NOTE* For Raspberry Pi users, if installation fails during building of slixmpp dependences, please add:

```shell
sudo apt-get install libffi-dev
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
