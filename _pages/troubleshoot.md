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

## Troubleshoot XMPP 

### Gathering information

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

### Fixing the problem

* This is outside the scope of Evio - you need to fix either the configuration, or the XMPP server. Once you are confident the node authenticates to XMPP, if the problem persists, continue to the next step

## Troubleshoot bounded flood (BF) 

### Gathering information

* It is possible the BF module fails to initialize properly. One way to check this is:

```
grep DPSet bf.log
```

If there is no output, or if the output shows an empty set under port_state, as shown below, your BF module was not initialized properly:

```
0915 10:55:39.469 INFO: DPSet.port_state {}
```

### Fixing the problem

* The current solution to this problem is to restart Evio:

```
sudo systemctl restart evio
```

* If, after a restart, the problem with the BF module has gone away, but your node is still unable to join the netowrk, continue to the next step

## Troubleshoot NAT traversal

### Gathering information

* It is possible that either your Evio node is misconfigured, or STUN and/or TURN services are not available to perform NAT traversal

* First, ensure that the IP address(es) and port(s) listed under LinkManager in the configuration file are correct for your STUN server, and that the STUN server is reachable from your node. If you use Google STUN servers, chances are they are working properly, as they are well-provisioned and reliable:

```
"Stun": [
      "stun.l.google.com:19302",
      "stun1.l.google.com:19302"
    ],
```

* You can check STUN activity in the log file as follows; ou should see several lines with local_candidate at the output:

```
grep -i stun tincan_log_* | more
```

* If you see STUN messages that indicate correct activity, it's possible your node is behind a symmetric NAT and needs TURN for NAT traversal

* One tool to help check the behavior of your NAT is [Stuntman](http://www.stunprotocol.org/) - install it on your node, and check the output of the command:

```
stunclient --mode full stun.stunprotocol.org
```

* An example of an output of a STUN-amenable NAT is:

```
Binding test: success
Local address: 172.17.111.10:38382
Mapped address: XXX.YYY.ZZZ.WWW:38382
Behavior test: success
Nat behavior: Endpoint Independent Mapping
Filtering test: success
Nat filtering: Endpoint Independent Filtering
```

* An example of a NAT that requires TURN for traversal:

```
Binding test: success
Local address: 172.18.0.2:40881
Mapped address: XXX.YYY.ZZZ.WWW:40881
Behavior test: success
Nat behavior: Endpoint Independent Mapping
Filtering test: success
Nat filtering: Address and Port Dependent Filtering
```

* If you have TURN configured, check that Evio is able to successfully allocate TURN sessions. The output of the following command should be non-empty:

```
grep "TURN allocate requested successfully" tincan_log_* | more
```

### Fixing the problem

* Ensure that the STUN server(s) is(are) properly configured and reachable

* If your NAT requires TURN traversal, ensure the TURN server(s) are properly configured and reachable


# <i class="fas fa-bug"></i> Finding and Submitting Bugs

If you encounter any bugs while running EdgeVPN.io, please let us know to help us improve the software. 

## Check whether it is a known issue

It's possible the bug you've encountered is a [reported known issue](https://github.com/EdgeVPNio/evio/issues). Please check comments for potential work-arounds while the issue is being investigated.


## Share logging information

The logs can get large - it's best if you are able to compress/archive and upload the log files to an accessible repository where developers can retrieve them (e.g. Google Drive)

## Submit an issue via GitHub

Once you have submitted logs, [create a GitHub issue to report the bug](https://github.com/EdgeVPNio/evio/issues). Additionally, provide the EdgeVPN.io version, the platform being used, and configuration file(s) used (please ensure you remove sensitive information, such as XMPP and TURN user name/password)
