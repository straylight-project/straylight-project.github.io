---
layout: post
title:  Installing NetBox using the Straylight Ansbile Roles
date:   2018-06-27 14:05:54 -0400
categories: straylight ansible netbox
---

## Requirements

- Vagrant
- Ansible

## Setup

1. Change to the cloned directory: `cd straylight-dev`
2. Add the repos for the Ansible roles into the `roles` directory (which is ignored):
```
git clone git@github.com:straylight-project/ansible-role-netbox-db.git roles/netbox-db
git clone git@github.com:straylight-project/ansible-role-netbox-app.git roles/netbox-app
git clone git@github.com:straylight-project/ansible-role-netbox-web.git roles/netbox-web
```
3. Start the vagrant netbox-unified VM: `vagrant up netbox-unified`. This should create a VM and use the `netbox_unified.yml` playbook to provision the the dev VM in vagrant.
4. Get into the VM: `vagrant ssh netbox-unified`
5. Create a superuser for NetBox (in the VM): `/srv/netbox/bin/python /srv/netbox/netbox/manage.py createsuperuser`. Follow the prompts.

Connect to the NetBox interface at https://localhost:20443/

