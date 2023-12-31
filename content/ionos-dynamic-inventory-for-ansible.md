---
title: "IONOS Dynamic Inventory For Ansible"
type: post
date: 2023-10-10T18:20:14+02:00
draft: false
---
So, this week end I did a thing!

I have recently joined IONOS as a Senior Cloud Solutions Architect and started working with the products they offer to build a nice and robust cloud solution.

I have found myself in need of a Dynamic Inventory for Ansible though, and as I couldn&#8217;t make the one already existing to work, I have taken on the challenge to write one myself.

And here we go, a python script able to create groups based on your Virtual Data Center setup and on hostname <a href="https://github.com/0dataloss/ionos-ansible-dynamic-inventory/tree/development" target="_blank" rel="noreferrer noopener">https://github.com/0dataloss/ionos-ansible-dynamic-inventory/tree/development</a>

The Inventory script is still in development, there are quite a few bits to take care of, let&#8217;s say that for the moment works fine with Cloud Servers, which was the intended target.

If you have ideas or improvement you are welcome to contribute, the GitHub repo is open to everyone, just open Pull requests to the development branch.

<pre class="wp-block-code"><code>IONOS Dynamic Inventory

This is a personal attempt to create a decent Dynamic Inventory for IONOS Public Cloud So far the features of this script are pretty limited to the basics:

    Groups made by hosts in the same Virtual Data Center, grouped by the Virtual Data Center's Name
    Groups made by single host, grouped by the server's Name

Table of Contents

    Description
    Installation
    Usage
    Test your setup

Description

IONOSinventory.py is an Ansible dynamic inventory, able to generate JSON output from requests sent to the using n external inventory system that can generate a proper JSON output from the Ionos API infrastructure.
Installation

No special install procedure is required. The script can be used stand-alone or with Ansible The IONOSinventory.py has been tested with Python 3.8
Configuration

To use the scrip is recommended to export your IONOS credential for a user with admin access as environment variables:

export IONOS_USERNAME="your@username.com" && export IONOS_PASSWORD="your password"

or Is possible to specify username and password inside the file itself (recommended ONLY for dev purposes)

###############################################
## You can configure username and password here
###############################################
username=0
password0
###############################################

or If none of the two options above have been set-up, the script will request user input username and password.

It is also possible to change the API end-point URL; there is no need right now as the script has been design for the latest version of the API end-point (v6.0), and this functionality is reserved for future use.

apiEp="https://api.ionos.com/cloudapi/v6"

Usage

IONOSinventory.py exposes bi default the --list functionality required by the Ansible inventory system; the --list switch is the only one implemented so far.

In this mode, the IONOSinventory.py will expose servers grouped by Virtual Data Center and by Name. At this moment there is an issue with the 'grouped by name' as if there are multiple machines with the same name across multiple Virtual Data Center they will all be used as 'host' by Ansible.
Test your setup

Is it possible to run the script stand-alone against the IONOS API to verify if the username and password used are working as expected in terms of accessing resources

$ export IONOS_USERNAME="your@username.com" && export IONOS_PASSWORD="your password" ; inventory.py 

{
    "workbench": {
        "hosts": &#91;
            "77.68.67.000"
        ],
        "vars": {}
    },
    "Center": {
        "hosts": &#91;
            "77.68.67.000"
        ],
        "vars": {}
    },
    "_meta": {
        "hostvars": {
            "77.68.67.000": {
                "name": "workbench",
                "id": "10e4fb9f-1a89-4a06-8796-f77b3338cb3c"
            }
        }
    }
}
</code></pre>