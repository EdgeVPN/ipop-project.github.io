---
permalink: /AWSbootstrap/
title: "Deploying EdgeVPN.io bootstrapping server on Amazon EC2"
header:
  overlay_color: "#303030"
  overlay_image: /assets/images/texture.png
---

# Introduction and requirements

This document describes how you can deploy your own server on the Amazon EC2 cloud to serve as a bootstrapping node with XMPP and TURN services for your own Evio networks.

The main requirement is that you have an Amazon AWS account, and that you are familiar with launching EC2 instances and configuring security groups. 

You can get started with a baseline EC2 Ubuntu 18.04 instance and use an IP address assigned by AWS on startup. The recommended deployment model is to use an Elastic IP address that has a DNS mapping, but it is not a requirement.

# Deploy AWS instance

First, log in to the AWS console, and deploy an instance as follows:

* Select the AWS Ubuntu 18.04 AMI
* Suggested configuration: t3.medium with 32GB disk
* You must configure the following security group policies for inbound traffic:

| Type | Protocol | Port range | Source | Description |
| --- | ----------- |
