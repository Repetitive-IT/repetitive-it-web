---
title: "Autoscaling @IONOS Public Cloud"
type: post
date: 2023-10-10T18:20:14+02:00
draft: false
---
I did this a while ago while working as Solutions Architect @IONOS UK

This for me highlight the WHY Cloud is so different from technologies like Dedicated Servers and Virtual Machines (And K8S yes at if you want to save some buck you may run it on cloud servers and not in a PaaS env.).

Cloud means AUTOMATION, means API, means interaction between what once was only hardware and now is software and especially with the smaller Cloud Providers where the prices are a bit lower, being able to automate becomes an easy win because of the money you could save (and who wouldn&#8217;t like to reduce his own running costs?) ! 

IONOS still does not provides you with an auto-scaling API, sure there are Terraform and Ansible Modules but both will require you to build your systems around them (respectively Infrastructure as a Code and Configuration Management), to use one or both it means your Infrastructure must be build from the scratch with Terraform and your systems configured with Ansible, but that requires a lot of effort for an infrastructure which is already standing.

The type of Auto-scaler I am showcasing here instead is more of a mid-level solution for people willing to walk the &#8216;road to automation&#8217; but not really ready to go the full length.

Or if you want, a way for you to be able with a simple click to de-provision servers during the slow months and provision servers at will during peak time (sales or campaign it does not matter!)

I see this as the missing link type of idea, where someone can create an infrastructure manually, but once done will be able to automate it in few simple steps.<figure class="wp-block-embed aligncenter is-type-video is-provider-youtube wp-block-embed-youtube wp-embed-aspect-4-3 wp-has-aspect-ratio">

<div class="wp-block-embed__wrapper">
</div></figure> 

This is obviously a simple Proof Of Concept and I beg you not to use in a production environment as it is.

But if you like the idea, give me a shout if you need help, I will be glad to improve my code adding features or fixing bugs.

The code is available here: https://github.com/0dataloss/IONOS-FAS