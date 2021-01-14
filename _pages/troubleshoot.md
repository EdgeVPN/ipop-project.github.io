---
permalink: /troubleshoot/
title: "Troubleshooting EdgeVPN.io"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# <i class="fas fa-tools"></i> Troubleshooting common problems

Often, problems that arise when running Evio are due to mis-configuration of Evio and/or the services it uses:

## Gather logging information

The first step is to locate the log files - they will give you clues to troubleshoot the problem, and if you cannot find a solution, it will also help developers reproduce and diagnose the fault. If you run into errors/crashes, please proceed as follows:

* Set the [logging level in the configuration file to DEBUG](/configfile), and try to recreate the problem

* Locate the log files for controller, bounded flood, and tincan. These are specified in the configuration file - by default, you will find them under a directory created in /var/log/, with names: bf.log, ctrl.log.*, and tincan_log_*

## Check for XMPP problems

If your Evio node is unable to connect and authenticate to the configured XMPP server, the node will not work. So, this is the first area to investigate; you should check that:

* The XMPP server is up and running, and is reachable via the public Internet from the Evio node

* The IP address and credentials to authenticate to the XMPP server are correctly configured in the json file

* Clues can be gathered by looking at the ctrl.log file; if you see a single log entry with "[DATE TIME] INFO:Logger: Module loaded", your node is not authenticated to XMPP. The beginning of the log for an authenticated node will look like this:

```
[20210113 22:25:25.442] INFO:Logger: Module loaded
[20210113 22:25:25.742] INFO:TincanInterface: Creating Tincan control link
[20210113 22:25:25.743] INFO:TincanInterface: Module loaded
[20210113 22:25:25.743] INFO:Signal: No key-ring found
[20210113 22:25:25.744] INFO:Signal: Module loaded
[20210113 22:25:25.744] WARNING:LinkManager: OverlayVisualizer module not loaded. Visualization data will not be sent.
[20210113 22:25:25.745] INFO:LinkManager: Module Loaded
[20210113 22:25:25.745] WARNING:Topology: OverlayVisualizer module not loaded. Visualization data will not be sent.
[20210113 22:25:25.745] INFO:Topology: Module loaded
[20210113 22:25:25.745] INFO:BridgeController: Module Loaded
[20210113 22:25:25.745] INFO:UsageReport: Module loaded
[20210113 22:25:27.085] INFO:Topology: Authorizing peer edge from 101000F:add7376->de4771d
```

* If you have access to the XMPP server (e.g. if you are running your own Openfire), log in to the admin Web interface and, while the Evio node is running, verify that there is an active session to the XMPP server

## Check for bounded flood problems

## Check for NAT traversal problems

# <i class="fas fa-bug"></i> Finding and Submitting Bugs

If you encounter any bugs while running EdgeVPN.io, please let us know to help us improve the software. 

## Check whether it is a known issue

It's possible the bug you've encountered is a [reported known issue](https://github.com/EdgeVPNio/evio/issues). Please check comments for potential work-arounds while the issue is being investigated.


## Share logging information

The logs can get large - it's best if you are able to compress/archive and upload the log files to an accessible repository where developers can retrieve them (e.g. Google Drive)

## Submit an issue via GitHub

Once you have submitted logs, [create a GitHub issue to report the bug](https://github.com/EdgeVPNio/evio/issues). Additionally, provide the EdgeVPN.io version, the platform being used, and configuration file(s) used (please ensure you remove sensitive information, such as XMPP and TURN user name/password)
