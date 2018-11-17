# How to buildout and deploy Pleiades using ansible

Ansible runs on your own local machine and uses ssh to connect to the target host(s). You must run it with a user that has ssh defined for that host and has sudo there. 

## just update and re-run the plone buildout

```
ansible-playbook -l {live|staging} deploy.yml
```

## set up a full server, including apache, etc.

```
ansible-playbook -l {live|staging} pleiades.yml
```

## execute a single task in the pleiades.yml

E.g., updating pleiades-frontpage.

**if** there is a "tag" defined, you can:

```
ansible-playbook -l staging pleiades.yml --tags frontpage
```

## roles (groups)

You may need to install roles if you get an error message trying to do other things:

```
ansible-galaxy install -r install_roles.yml
```

## versions of packages and products for the plone buildout

Specific versions of packages used in the plone buildout are pinned in ```pleiades-buildout/buildout.cfg```.

Specific versions/branches/commits of other things needed by an ansible playbook may be specified in the individual '.yml' playbook file.

## inventory (list of servers)

See ```inventory.cfg``` in the ansible directory.

## templates

Templates for some of the files that are created by the ansible playbooks may be found in the ```templates``` directory (e.g., apache conf files).

