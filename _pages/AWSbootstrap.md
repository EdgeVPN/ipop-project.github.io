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

You can get started with a baseline EC2 Ubuntu 20.04 or 18.04 instance and use an IP address assigned by AWS on startup. The recommended deployment model is to use an Elastic IP address that has a DNS mapping, but it is not a requirement.


# Deploy AWS instance

First, log in to the AWS console, and deploy an instance as follows:

* Select the AWS Ubuntu 20.04 AMI
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

You need to customize your deployment by setting the following three environment variables with: the password you want for your SQL server; the AWS_PUBLIC_IP; and a base address for a Docker network which will be created by Docker compose.

The docker compose deployment needs to create a docker network to bind the mysql/openfire containers too. Pick any address space that is available on your host (e.g. 172.16.238/24), but only use the first 3 bytes (separated by two dots, e.g. 172.16.238) when you configure this environment variable. The mysql container will bind to a static IP $EVIODB_DOCKER_NET.2 and openfire will bind to static IP $EVIODB_DOCKER_NET.3 - the coturn container binds to the host network, as it requires a large number of ports.

```
export MYSQL_ROOT_PASSWORD="Enter mysql root password here"
export AWS_SERVER_IP="Enter IP Here"
export EVIODB_DOCKER_NET="172.16.238"
```

Now run the setup script:

```
bash setup_evio_bootstrap_server_docker-compose.sh
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

```
cd evio_config_gen
vi generate_evio_config_trial.py
```

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






