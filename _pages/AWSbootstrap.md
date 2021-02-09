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

## Acknowledgment

*This cloud resource is supported by community funds provided by [CloudBank](https://www.cloudbank.org/), which is supported by National Science Foundation award [#1925001](https://www.nsf.gov/awardsearch/showAward?AWD_ID=1925001)*

# Deploy AWS instance

First, log in to the AWS console, and deploy an instance as follows:

* Select the AWS Ubuntu 18.04 AMI
* Suggested configuration: t3.medium with 32GB disk
* You must configure the following security group policies for inbound traffic:

| Type            | Protocol | Port range          | Source           | Description |
| --------------- | -------- | ------------------- | ---------------- | ----------- |
| Custom UDP Rule | UDP      | 3478                | 0.0.0.0/0        | coturn      |
| Custom TCP Rule | TCP      | 3478                | 0.0.0.0/0        | coturn      |
| Custom UDP Rule | UDP      | 49160 - 59200       | 0.0.0.0/0        | coturn      |
| SSH             | TCP      | 22                  | 0.0.0.0/0        | SSH         |
| Custom TCP Rule | TCP      | 5222 - 5223         | 0.0.0.0/0        | XMPP        |
| Custom TCP Rule | TCP      | 9090 - 9091         | See note!!       | openfire    |
|                 | TCP      |                     | AWS_PUBLIC_IP/32 | openfire    |

Note: the last TCP rule for ports 9090-9091 specify which addresses can access the admin interface for Openfire. It is strongly recommended that, instead of opening up to the world (0.0.0.0/0) you provide the list of IP addresses of each admin user who will manage the XMPP server - including your AWS_PUBLIC_IP so you can ssh-tunnel into it.

# Log in to your instance and deploy services

From the EC2 interface, find out the public address of your instance (we’ll call it AWS_PUBLIC_IP in this document) and ssh into it with your AWS key:

```
ssh -i your_aws_key.pem ubuntu@AWS_PUBLIC_IP
```

Now, clone the evio-config-gen repo:

```
git clone https://github.com/renatof/evio_config_gen.git
cd evio_config_gen/
```

Edit the setup_evio_bootstrap_server.sh, and customize for your deployment by changing the following two variables with the password you want for your SQL server, and the AWS_PUBLIC_IP:

```
# Replace with the password you want for your MySQL server
MYSQL_ROOT_PASSWORD="my_sql_root_password"
# Replace with the AWS instance's public IP address
AWS_SERVER_IP="AWS_SERVER_IP"
```

Now run the setup script:

```
./setup_evio_bootstrap_server.sh
```

The script will pause mid-way and ask you to continue Openfire setup in your browser; follow the instructions on the terminal

*Once the script finishes. Log out of your instance, then ssh back in, so can use Docker as the ubuntu user.* You should now see three containers running, evio-mysql, evio-coturn, and evio-openfire:

```
exit
ssh -i your_aws_key.pem ubuntu@AWS_PUBLIC_IP
docker ps
```

# Create XMPP/TURN user accounts, and Evio configuration files

First, change into evio_config_gen, and edit the generate_evio_config_trial.py script.

You need to change two constants in the script (SERVER_ADDRESS, XMPP_DOMAIN), as follows:

* If you are using a numeric IP address *without DNS configured*, enter the AWS public IP address and set the XMPP_DOMAIN as openfire.local:
SERVER_ADDRESS: AWS_SERVER_IP
XMPP_DOMAIN: openfire.local

* If you do have a valid public DNS mapping for your server, enter its fully qualified domain name (FQDN) for both:
SERVER_ADDRESS: FQDN
XMPP_DOMAIN: FQDN

Now let's create a five test accounts and an overlay named Test1:

```
cd ~/evio-config-gen
python3 generate_evio_config_trial.py --sqlpass=your_mysql_root_password --email=someone@somewhere.com --evioname=Test1 --baseip=10.10.100 --numnodes=5 --numacct=5
```

If you cd into Test1, you will see five .json configuration files, and a couple of shell scripts. You will need to run the main script as follows to enter your accounts into the MySQL database:

```
cd Test1
chmod 755 *.sh
./docker-config-openfire-turn.sh
```

Now you can copy the .json files for each of your Evio nodes, and start them up as described in the [installation instructions](https://edgevpn.io/install/) or, [these instructions if you use Docker](https://edgevpn.io/dockeredgevpn/).







