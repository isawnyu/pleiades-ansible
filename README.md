Pleiades Ansible Playbook
=========================

Ansible playbooks meant to be sufficient to build or rebuild a server with
minimal preparation.

These do not handle live data transfer.

Prerequisites
-------------

Ubuntu 16.04 server and an ssh login with full sudo. The server must have the
following packages installed::

  * python
  * aptitude
  * language-pack-en (?)

    sudo apt-get update
    sudo apt-get install language-pack-en aptitude python

Install required roles::

    ansible-galaxy -p roles install -r install_roles.yml

Typical use::

    ansible-playbook pleiades.yml
    (targets all servers)

    ansible-playbook pleiades.yml -l staging
    (targets only the staging server)

If the server is installed on a cloud with its own firewall (like EC2), see
``firewall.yml`` for ports that must be opened manually using that service.

Inventory
~~~~~~~~~

The default inventory file is ``inventory.cfg``.
It defines both live and staging groups.

There is a matching vbox-host.cfg that defines the same groups, but targets the vagrant virtualbox.
Usage: ``ansible-playbook -i vbox-host.cfg pleiades.yml``

Host and Group Variables
~~~~~~~~~~~~~~~~~~~~~~~~

For each host in the inventory, it is possible to customize deployment some
variables using the files in ``host_vars``. For example
``host_vars/ec2_staging.yml`` contains host specific variables for the EC2
staging server.

Similarly, deployment variables common across a group of servers can be
configured in ``group_vars``. For example, ``group_vars/staging.yml``
contains settings common to all servers in the staging group.

Currently, we expect the following variables to be set for
each server:

* newrelic_license_key: Sets the API key for the New Relic account to be used for 
  monitoring
* newrelic_appname: Sets the prefix to be used in new relic for all apps and
  plugins.
* plone_client_count: Sets the number of plone instances to run on the
  server (limit 5, the first will not be included in the load balancer and is
  accessed directly for administrative purposes only)
* restart_delay: Sets the number of seconds to wait between rolling instance
  restarts (half the time will be spend waiting after the instance stops and 
  half waiting after the intance starts). Defaults to 60 seconds.
* cache_preload_urls: url paths (including site path) to load after restarting
  an instance. This helps prevent slowness after reload by pre-filling the
  Zope and ZEO caches with frequently used objects.
* playbook_plones: a list of loadbalanced backeds to create, each should define:
  * plone_instance_name: A name for the backend
  * loadbalancer_port: The port the loadbalancer listens on
  * plone_client_base_port: The port for the first plone instance
  * webserver_virtualhosts: A list of virtual host mappings, consisting of:
    * host: the host name
    * plone_path: the path to the site root


Playbooks
---------

add_users.yml
~~~~~~~~~~~~~

Targets: all

Establishes basic user set, generally should be run befor running pleiades.yml
for the first time. Only needs to be run to create/update user accounts on the
server.

pleiades.yml
~~~~~~~~~~~~

Targets: all

Installs required packages and sets up:

* Postfix mailer config
* Plone buildouts for Plone 3 and 4
* Vaytrou
* supervisor
* haproxy
* varnish
* Repositories for static websites from ``pleiades-frontpage`` and ``pleiades-api``
* apache (virtualhosts for ``pleiades.stoa.org``, ``api.pleiades.stoa.org``, ``atlantides.org``, and ``concordia.atlantides.org``)
* Sets server timezone to ``America/New_York``
* Creates nightly cron jobs for CSV, KML, and sitemap exports (Plone 4)
* Creates weekly cron job for RDF export (Plone 4)
* Automated nightly Ubuntu security updates

deploy.yml
~~~~~~~~~~

Targets: all

Updates Plone 4 buildout repo, runs buildout, performs a rolling restart of
plone instances with configurable delay. Can only be run after a successful
initial server deploy using ``pleiades.yml``.

firewall.yml
~~~~~~~~~~~~

Targets: all

Closes all ports except 80, 443, 8080. Uses ufw. Only needs to be run to
update firewall rules.

monitoring.yml
~~~~~~~~~~~~~~

Sets up New Relic monitoring for server, along with plugins to monitor Apache,
HAProxy and Varnish. New Relic app monitoring setup is done in
``pleiades.yml`` and ``deploy.yml``.
