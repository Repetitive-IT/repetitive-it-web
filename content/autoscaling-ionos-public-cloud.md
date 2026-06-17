---
title: "Autoscaling on IONOS Public Cloud"
type: post
date: 2023-10-10T18:20:14+02:00
draft: false
---

I did this a while ago while working as a Solutions Architect at IONOS UK.

This highlights why cloud is so different from technologies like dedicated servers and virtual machines (and Kubernetes – if you want to save money, you can run it on cloud servers instead of a PaaS environment).

Cloud means automation, APIs, and interaction between what was once only hardware and is now software. With smaller cloud providers, prices are lower, so automating becomes an easy win because you can save money (and who wouldn't like to reduce their own running costs?).

At the time I am writing this, IONOS still does not provide an auto-scaling API. Sure, there are Terraform and Ansible modules, but both require you to build your systems around them (Infrastructure as Code and Configuration Management). Using one or both means your infrastructure must be built from scratch with Terraform and your systems configured with Ansible, which requires a lot of effort for an infrastructure that is already standing.

The type of auto-scaler I am showcasing here is more of a mid-level solution for people willing to walk the "road to automation" but not yet ready to go the full length.

This is obviously a simple Proof Of Concept and I beg you not to use in a production environment as it is.But if you like the idea, give me a shout if you need help, I will be glad to improve my code adding features or fixing bugs.

The code is available here: https://github.com/0dataloss/IONOS-FAS