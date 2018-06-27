---
layout: post
title:  Installing NetBox using the Straylight Ansbile Roles
date:   2018-06-27 14:05:54 -0400
---

This post will walk you through the process of using the Straylight Project's Ansible roles to get [NetBox](http://netbox.readthedocs.io/) up and running on an Ubuntu 18.04 ("bionic") box. This tutorial needs python2 installed on the target (`apt-get install python`).

Start by making sure you have the straylight-project.netbox-\* Ansible Roles installed on your control machine (these don't need to be on the target box).

```
$ ansible-galaxy install straylight-project.netbox-db
$ ansible-galaxy install straylight-project.netbox-app
$ ansible-galaxy install straylight-project.netbox-web
```

You'll need a SSL key and certifcate to get to have the web proxy use HTTPS. You can use your own, or generate a self-signed SSL certificate.

```
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout self-signed.key -out self-signed.crt
```

This command will prompt for some information, then generate key and certificate in your working directory.

Create a playbook `netbox.yml` that uses these roles.

{% raw %}
```
---
- name: provision netbox web and database
  hosts: netbox.example.com
  vars:
    install_dir: /srv
    database_name: netbox_db
    database_user: netbox_user
    database_password: secretpassword
    hostname: netbox.example.com
  become: true
  become_method: sudo
  roles:
    - name: straylight-project.netbox-db
      netbox_database_name: "{{ database_name }}"
      netbox_database_user: "{{ database_user }}"
      netbox_database_password: "{{ database_password }}"

    - name: straylight-project.netbox-app
      netbox_install_directory: "{{ install_dir }}"
      netbox_database_name: "{{ database_name }}"
      netbox_database_user: "{{ database_user }}"
      netbox_database_password: "{{ database_password }}"
      netbox_allowed_hosts:
        - "{{ hostname }}"
        # - "localhost:8443" # workaround for CSRF protection when running on a non-standard port for development or testing

    - name: straylight-project.netbox-web
      netbox_install_directory: "{{ install_dir }}"
      netbox_webserver_ssl_key: "{{ lookup('file', 'self-signed.key') }}"
      netbox_webserver_ssl_crt: "{{ lookup('file', 'self-signed.crt') }}"
```
{% endraw %}

Note that the playbook is looking for the `self-signed.key` and `self-signed.crt` files (created in the steps above) to be in the same directory as the playbook.

You should now be able to connect to the application at `https://netbox.example.com/`

In order to login, you'll need to create a superuser account. To do that SSH to your target box and run this command:

```
$ /srv/netbox/bin/python /srv/netbox/netbox/manage.py createsuperuser
```

This will prompt you for a information to create the new user. Once you finish that, you can use those credentials to log in to the web interface.
