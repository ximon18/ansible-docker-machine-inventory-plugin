# Docker Machine dynamic inventory plugin for Ansible

A [dynamic inventory plugin](https://docs.ansible.com/ansible/latest/plugins/inventory.html) for Ansible (tested with 2.7.10 and Python 3.6.7 on Ubuntu Linux 18.04 LTS).

## Inspiration

This plugin is based on and is very similar to the Ansible 2.8 [docker_swarm](https://docs.ansible.com/ansible/devel/plugins/inventory/docker_swarm.html?highlight=docker_swarm) and [aws_ec2](https://docs.ansible.com/ansible/devel/plugins/inventory/aws_ec2.html?highlight=aws_ec2) Dynamic Inventory plugins.

While there are other similar solutions out there, I did not find an existing Docker Machine dynamic inventory _plugin_, only _scripts_ (e.g. [this](https://gist.github.com/nathanleclaire/1bbf18de7c73f89aa36c)).

## Features

This plugin teaches Ansible about which Docker Machine machines exist and how to connect to them via SSH so that the Ansible controller can execute commands on the remote machine.

It also makes available the DOCKER_xxx environment variables output by `docker-machine env <machine-name>` as Ansible host variables, prefixed with `dm_`. These are intended to be used set Ansible or environment variables such that Docker commands (e.g. docker ps, docker-compose up) will be executed against the Docker daemon running on the remote machine and not the local Docker daemon.

Like the Docker Swarm and AWS EC2 plugins it supports `keyed_groups` and other `constructable` ways of dynamically defining Ansible host groups.

## Requirements

Docker Machine must be [installed](https://docs.docker.com/machine/install-machine/) on the Ansible controller.

## Usage

1. Tell Ansible where to find the plugin. See the [Ansible documentation](https://docs.ansible.com/ansible/latest/dev_guide/developing_locally.html#adding-a-plugin-locally) for all the possible ways that you can do this.
2. Tell Ansible how to configure the plugin by pointing it to a `docker_machine.yml` configuration file that you create.

E.g.

```
$ export ANSIBLE_INVENTORY_ENABLED=docker_machine
$ export ANSIBLE_INVENTORY_PLUGINS=/tmp/ansible-docker-machine-inventory-plugin
$ git clone https://github.com/ximon18/ansible-docker-machine-inventory-plugin.git ${ANSIBLE_INVENTORY_PLUGINS}
$ vi ${ANSIBLE_INVENTORY_PLUGINS}/docker_machine.env
$ ansible -i ${ANSIBLE_INVENTORY_PLUGINS}/docker_machine.yml -m ping all
```

To find out what the plugin has discovered do:

```
$ ansible-inventory -i ${ANSIBLE_INVENTORY_PLUGINS}/docker_machine.yml --graph --vars
```

END
