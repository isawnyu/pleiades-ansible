Pleiades Ansible Playbook
=========================

Ansible playbooks meant to be sufficient to build or rebuild a server with minimal preparation.

These do not handle live data transfer.

Prerequisites
-------------

Ubuntu 14.04 server and an ssh login with full sudo. Install required roles::

    ansible-galaxy -p roles install -r install_roles.yml

Typical use::

    ansible-playbook pleiades.yml
    (targets all servers)

If the server is installed on a cloud with its own firewall (like EC2), see firewall.yml for ports that must be opened.

Inventory
~~~~~~~~~

The default inventory file is inventory.cfg.
It defines both live and staging groups.

There is a matching vbox-host.cfg that defines the same groups, but targets the vagrant virtualbox.
Usage: ansible-playbook -i vbox-host.cfg pleiades.yml

Playbooks
---------

firewall.yml
~~~~~~~~~~~~

Targets: all

Closes all ports except 80, 443, 8080. Uses ufw.

pleiades.yml
~~~~~~~~~~~~~~~~~

Targets: all

Installs required packages and sets up:

* Postfix
* Plone
* Vaytrou
* supervisor
* haproxy
* varnish
* apache
