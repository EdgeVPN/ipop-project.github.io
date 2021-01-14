---
permalink: /troubleshoot/
title: "Troubleshooting EdgeVPN.io"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# <i class="fas fa-bug"></i> Finding and Submitting Bugs

If you encounter any bugs while running EdgeVPN.io, please let us know to help us improve the software. 

## Check whether it is a known issue

It's possible the bug you've encountered is a [reported known issue](https://github.com/EdgeVPNio/evio/issues). Please check comments for potential work-arounds while the issue is being investigated.

## Gather logging information

Gathering relevant logging information will help developers reproduce and diagnose the fault. If you run into errors/crashes, please proceed as follows:

* Set the [logging level in the configuration file to DEBUG](/configfile), and try to recreate the problem

* Retrieve the log files for controller, bounded flood, and tincan. These are specified in the configuration file - by default, you will find them under a directory created in /var/log/, with names: bf.log, ctrl.log.*, and tincan_log_*

## Share logging information

The logs can get large - it's best if you are able to compress/archive and upload the log files to an accessible repository where developers can retrieve them (e.g. Google Drive)

## Submit an issue via GitHub

Once you have submitted logs, [create a GitHub issue to report the bug](https://github.com/EdgeVPNio/evio/issues). Additionally, provide the EdgeVPN.io version, the platform being used, and configuration file(s) used (please ensure you remove sensitive information, such as XMPP and TURN user name/password)
