---
title: "AWS MSK – How to expose the cluster on the public network"
type: post
date: 2023-10-10T18:20:14+02:00
draft: false
---

Welcome to your new remote MSK cluster!

For AWS users with the new Apache Kafka as a Service. MSK is one of the new services that just came out of beta; it provides AWS users with a managed Apache Kafka service.

The fact that you cannot change the instance type in your cluster (for the moment at least). There is a lot of documentation to read before deciding whether to use this service in your environment. Please make sure you go through all the FAQs, as some of the usual features may not work as expected. For example, the cluster does not support auto-scaling, and you cannot change the instance type in your cluster (at least for now).

**POC for the Dev team:** needed to explore the capabilities of this product. Being something that needs to be accessed directly from the developers' laptop, we needed this cluster to be publicly available to our network, and this is how it started.

I just started working with it, and the first challenge was to create a POC for the Dev team to explore the capabilities of this product. Because it needs to be accessed directly from developers' laptops, we required the cluster to be publicly available to our network, and this is how it started.

The workaround we found may not work for you, but I hope it may be helpful to get an insight of how the service works.

The new MSK service is not engineered to be available on the public network, and the workaround we found may not work for you. However, I hope it will be helpful to get an insight into how the service works.

First, I created a new VPC spanning three availability zones: a standard VPC with three public subnets, three private subnets, one NAT gateway, three Internet gateways, and a security group that allows only outbound traffic. Over this VPC, I created a basic MSK cluster with no TLS authentication, 10 GB disk space, and three nodes—one in each availability zone, each in a public subnet and assigned to the previously created security group.

At this point, after 15 minutes, you will have a brand-new cluster. Clicking the client details button will give you the connection strings to use with your consumer/producer clients to reach Kafka (via TLS or plain authentication) and ZooKeeper. The problem is that this cluster is, at the moment, only reachable from inside the VPC.

To make the cluster visible without using a VPN, I followed these steps:

1. Request three Elastic IPs.
2. Looking at the connection strings for the cluster, resolve the names to the private IP addresses.
3. Now look in the Network Interface section in the EC2 menu and associate to each one of the cluster interfaces a public IP address (use the IP address we found above to understand which of the network interfaces are part of the cluster).
4. Once done, you will have a cluster with 3 public interfaces but not yet reachable because of the Security Group, so if we now modify the inbound rules to accept connections on the right port for ZooKeeper and Kafka.
5. Now give it a try with telnet to verify that everything is open.
6. At this point, you may try to connect but Kafka may throw an error at you saying that "Connection to the cluster is working but the Kafka cluster is not responsive". This is because we are not calling the Kafka nodes with the proper names.
7. Edit your hosts file and insert for each node of the Kafka cluster an entry with the public IP address and the name of that node; save it, close it, try again.

As well, remember that AWS will bill you for every IP address which is not assigned, so please take care of cleaning up properly after you have done with it. For me this may be a one-off job, but I may write a CloudFormation template to couple Public IP addresses with Network Interfaces from the Kafka nodes.

A VPN may work better, but this is just a POC and the less infrastructure we put in place, the less likely this POC may move from POC to pre-prod or prod environment.

Please note: every time you rebuild a cluster, you will have to go through all of this again. It may be a good idea to automate the procedure if you think you will be destroying and rebuilding the cluster over and over. Also, remember that AWS will bill you for every unassigned IP address, so clean up properly after you're done.

For me, this may be a one-off job, but I might write a CloudFormation template to associate public IP addresses with the network interfaces of the Kafka nodes. A VPN may work better, but this is just a POC, and the less infrastructure we put in place, the less likely this POC will move from POC to pre-prod or prod.