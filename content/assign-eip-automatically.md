---
title: "Assign EIP Automatically"
type: post
date: 2023-10-10T18:20:14+02:00
draft: false
---

In this GitHub repo there is a small utility I use every time a customer needs to scale up and down a web cluster, but also wants the flexibility of having EIP addresses automatically assigned by the Auto-Scaling group.

Almost every time, the reason for this is whitelisting the platform for transactions against payment systems, but it could be anything really.

I hope this will be useful to someone out there.

**[AWS-EIP-AutoScale](https://github.com/0dataloss/AWS-EIP-AutoScale)**