# Docker Volume Plugin provisioning with Ansible


## What is this?

Easy to deploy NetApp docker volume plugin (nDVP). 

Main use case is setting up PoC environment.


## Requirements

Currently supported following environment.

- Ubuntu 16.04 
- latest docker version
- using ssh public-key authentication (required)

Target host must satisfied following condition.

- Ansible 
- /etc/ssh/sshd_config 
    - Enabled Key authentication
- python installed

## Current features

- Install docker
- Install NetApp Docker Volume Plugin
    - backend storage can set using group_vars/ndvp ndvp_plugins variable.

## How to use?

Configure following files.

- lab: inventory file. Replace your hosts ip.
- group_vars/all: Set ansibleuser and docker user. 
- group_vars/ndvp: Add backend storage.
- ndvp/files: Modify backend storage json for your environment. Please see official ndvp documentation.
    - ONTAP: http://netappdvp.readthedocs.io/en/latest/install/ndvp_ontap_config.html#ontap-configuration
    - SolidFire: http://netappdvp.readthedocs.io/en/latest/install/ndvp_sf_config.html
    - E-Series: http://netappdvp.readthedocs.io/en/latest/install/ndvp_e_config.html



```Examples
$ git clone makotow/docker-volume-plugin-provisioning
$ cd /path/to/docker-volume-plugin-provisioning
$ ansible-playbook -i lab site.yml --private-key ~/.ssh/lab_id_rsa  --ask-become-pass
```

## TODO
- CentOS/RHEL provisioning
- flexible configuration
    - Docker version. Currently, latest version only.
- docker-compose install
- Netapp docker volume plugin configuration file using template.
- sudo password set using Ansible Vault? (Is it require?)

## Authors

- [@makotow](https://github.com/makotow)

## License

MIT
