---
title: "IONOS Dynamic Inventory for Ansible"
type: post
date: 2023-10-10T18:20:14+02:00
draft: false
---

So, this weekend I did a thing!

I have recently joined IONOS as a Senior Cloud Solutions Architect and started working with the products they offer to build a nice and robust cloud solution.

I have found myself in need of a Dynamic Inventory for Ansible, though, and as I couldn't make the one already existing work, I have taken on the challenge to write one myself.

And here we go: a Python script able to create groups based on your Virtual Data Center setup and hostname. You can find it here: [IONOS Ansible Dynamic Inventory](https://github.com/0dataloss/ionos-ansible-dynamic-inventory/tree/development)

The Inventory script is still in development; there are quite a few bits to take care of. Let's say that for the moment it works fine with Cloud Servers, which was the intended target.

If you have ideas or improvements, you are welcome to contribute. The GitHub repo is open to everyone; just open Pull Requests to the development branch.

## IONOS Dynamic Inventory

This is a personal attempt to create a decent Dynamic Inventory for IONOS Public Cloud. So far, the features of this script are pretty limited to the basics:

- Groups made by hosts in the same Virtual Data Center, grouped by the Virtual Data Center's Name
- Groups made by single host, grouped by the server's Name

## Table of Contents

- [Description](#description)
- [Installation](#installation)
- [Usage](#usage)
- [Test your setup](#test-your-setup)

## Description

`IONOSinventory.py` is an Ansible dynamic inventory, able to generate JSON output from requests sent to an external inventory system that can generate a proper JSON output from the IONOS API infrastructure.

## Installation

No special install procedure is required. The script can be used stand-alone or with Ansible. `IONOSinventory.py` has been tested with Python 3.8.

## Configuration

To use the script, it is recommended to export your IONOS credentials for a user with admin access as environment variables:

```bash
export IONOS_USERNAME="your@username.com" && export IONOS_PASSWORD="your password"
```

Or it is possible to specify username and password inside the file itself (recommended ONLY for dev purposes):

```python
###############################################
## You can configure username and password here
###############################################
username=your_username
password=your_password
###############################################
```

If none of the two options above have been set up, the script will request user input for username and password.

It is also possible to change the API endpoint URL; there is no need right now as the script has been designed for the latest version of the API endpoint (v6.0), and this functionality is reserved for future use.

```python
apiEp="https://api.ionos.com/cloudapi/v6"
```

## Usage

`IONOSinventory.py` exposes by default the `--list` functionality required by the Ansible inventory system; the `--list` switch is the only one implemented so far.

In this mode, the `IONOSinventory.py` will expose servers grouped by Virtual Data Center and by Name. At this moment, there is an issue with the 'grouped by name' as if there are multiple machines with the same name across multiple Virtual Data Centers, they will all be used as 'host' by Ansible.

## Test your setup

It is possible to run the script stand-alone against the IONOS API to verify if the username and password used are working as expected in terms of accessing resources:

```bash
$ export IONOS_USERNAME="your@username.com" && export IONOS_PASSWORD="your password" ; python3 IONOSinventory.py
```

```json
{
    "workbench": {
        "hosts": [
            "77.68.67.000"
        ],
        "vars": {}
    },
    "Center": {
        "hosts": [
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
```