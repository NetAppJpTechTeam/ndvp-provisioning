# Docker Volume Plugin provisionig with Ansible


## Requirements

Currently supported following environment.

- Ubuntu 16.04 
- latest docker version
- using ssh public-key authentication (required)

target host satisfied following condition.

- Ansible 
- /etc/ssh/sshd_config 
    - Keyautehticated
- python installed


## What is this?

Provisioner for NetApp Docker Volume Plugin.

- docker related software install
    - docker
- Add docker user and group
- Enable service and started

## Current features



## How to use?

Provisioning via Ansible!

just do below
```
$ git clone makotow/docker-volume-plugin-provisioning
$ cd /path/to/repository-top && make
```


### Provision following environment

- Docker environment setup 
- NetApp Docker Volume Plugin

## TODO
- CentOS provisioning
- flexible configuration
    - docker version. Currently, latest version only.
- docker-compose install
- Netapp docker volume plugin for tempalate.
- sudo password set 
