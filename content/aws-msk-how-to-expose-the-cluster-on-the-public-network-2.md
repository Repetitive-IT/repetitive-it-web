---
title: "AWS MSK – How to expose the cluster on the public network"
type: post
date: 2023-10-10T18:20:14+02:00
draft: false
---
 

MSK is one of the new services which just came out of Beta, it provides  
the AWS users with the new Apache Kafka as a Service.

There is a lot in the documentation to read before you decide to use or  
not this service in your environment so please make sure you go through  
all the F.A.Q. because some of the usual stuff may not work as expected.  
One of these is the auto-scaling of the cluster (does not have one) or  
the fact that you cannot change the instance type in your cluster (for  
the moment at least).

I just started working with it, and the first challenge was to create a  
POC for the Dev team, needed to explore the capabilities of this product.  
Being something that needs to be accessed directly from the developers’  
laptop we needed this cluster to be publicly available to our network,  
and this is how it started.

The new MSK service is not engineered to be available on the public  
network and the workaround we found may not work for you, but I hope it  
may be helpful to get an insight of how the service works.

First thing first I have created a new VPC over 3 availability zones, a  
standard one with 3 Public subnets, 3 Private subnets, 1 NatGw, 3  
InternetGw and a security group with only outgoing traffic allowed.  
Over this VPC I have created a basic MSK cluster, no TLS auth, 10GB disk  
space, 3 nodes one on each availability zone, each one in a public  
subnet and assigned to the security group created before.

At this point, after 15 minutes, you will find yourself with a brand new  
cluster and if you click the client details button you will get the  
connection strings to use with your consumer/producer clients to reach  
Kafka on the TLS or plain auth and ZooKeeper.  
Problem being that this cluster is at the moment only reachable from the  
inside of the VPC.

To make the cluster visible without the use of a VPN this is what I have  
done:  
– Request 3 EIP  
– looking at the connection strings for the cluster resolve the names to  
the private IP addresses  
– Now look in the Network Interface section in the EC2 menu and  
associate to each one of the cluster interface a public IP address (use  
the IP address we found above to understand which of the network  
interfaces are part of the cluster)  
– Once done you will have a cluster with 3 public interfaces but not yet  
reachable because the Security Group, so if we now modify the inbound  
rules to accept connections on the right port for ZooKeper and Kafka.  
– Now give it a try with telnet to verify that everything is open  
– At this point you may try to connect but Kafka may throw an error at  
you saying that ‘Connection to the cluster is working but the Kafka  
cluster is not responsive’.  
This is because we are not calling the Kafka nodes with the proper names.  
– Edit your host file and insert for each node of the Kafka cluster an  
entry with the public IP address and the name of that node; save it,  
close it, try again.

Welcome to your new remote MKS cluster

Please note: every time you rebuild a cluster you will have to go  
through all of this again, it may be a good idea to automate the  
procedure if you think you will be destroying and rebuilding the cluster  
over and over.  
As well, remember that AWS will bill you for every IP address which is  
not assigned, so please take care of cleaning up properly after you have  
done with it.  
For me this may be a one off job, but I may write a Cloudformation  
template to couple Public IP addresses to Network Interfaces from the  
Kafka nodes.  
A VPN may work better, but this is just a POC and the less  
infrastructure we put in place the less likely this POC may move from  
POC to pre-prod or prod environment.