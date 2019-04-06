# Docker Machine dynamic inventory plugin for Ansible

A [dynamic inventory plugin](https://docs.ansible.com/ansible/latest/plugins/inventory.html) for Ansible (tested with 2.7.10 and Python 3.6.7 on Ubuntu Linux 18.04 LTS).

## Inspiration

This plugin is based on and is very similar to the Ansible 2.8 [docker_swarm](https://docs.ansible.com/ansible/devel/plugins/inventory/docker_swarm.html?highlight=docker_swarm) and [aws_ec2](https://docs.ansible.com/ansible/devel/plugins/inventory/aws_ec2.html?highlight=aws_ec2) Dynamic Inventory plugins.

While there are other similar solutions out there, I did not find an existing Docker Machine dynamic inventory _plugin_, only _scripts_ (e.g. [this](https://gist.github.com/nathanleclaire/1bbf18de7c73f89aa36c)).

## Features

This plugin teaches Ansible about which Docker Machine machines exist and how to connect to them via SSH so that the Ansible controller can execute commands on the remote machine.

It also makes available the DOCKER_xxx environment variables output by `docker-machine env <machine-name>` as Ansible host variables, prefixed with `dm_`. These are intended to be used to set Ansible or environment variables such that Docker commands (e.g. docker ps, docker-compose up) will be executed against the Docker daemon running on the remote machine and not the local Docker daemon.

Like the Docker Swarm and AWS EC2 plugins it supports `keyed_groups` and other `constructable` ways of dynamically defining Ansible host groups.

If `verbose_output` is enabled it also exposes the entire Docker Machine ```inspect``` output as an ansible inventory host variable called `docker_machine_node_attributes` (matching the naming style used by the Docker Swarm inventory plugin).

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
$ vi ${ANSIBLE_INVENTORY_PLUGINS}/docker_machine.yml
$ ansible -i ${ANSIBLE_INVENTORY_PLUGINS}/docker_machine.yml -m ping all
```

To find out what the plugin has discovered do:

```
$ ansible-inventory -i ${ANSIBLE_INVENTORY_PLUGINS}/docker_machine.yml --graph --vars
```

## Documentation

To read the full plugin docs you can use the `ansible-doc` command:

```
$ ansible-doc -t inventory docker_machine
```

E.g. at the time of writing on my local Ansible 2.7.10 installation this produces:

```
> INVENTORY    (/opt/nlnetlabs/gantry/ansible/dmaip/docker_machine.py)

        Get inventory hosts from Docker Machine. Uses a YAML configuration file that ends with docker_machine.(yml|yaml).

OPTIONS (= is mandatory):

- cache
        Toggle to enable/disable the caching of the inventory's source data, requires a cache plugin setup to work.
        [Default: False]
        set_via:
          env:
          - name: ANSIBLE_INVENTORY_CACHE
          ini:
          - key: cache
            section: inventory
        
        type: boolean

- cache_connection
        Cache connection data or path, read cache plugin documentation for specifics.
        [Default: (null)]
        set_via:
          env:
          - name: ANSIBLE_INVENTORY_CACHE_CONNECTION
          ini:
          - key: cache_connection
            section: inventory
        

- cache_plugin
        Cache plugin to use for the inventory's source data.
        [Default: (null)]
        set_via:
          env:
          - name: ANSIBLE_INVENTORY_CACHE_PLUGIN
          ini:
          - key: cache_plugin
            section: inventory
        

- cache_timeout
        Cache duration in seconds
        [Default: 3600]
        set_via:
          env:
          - name: ANSIBLE_INVENTORY_CACHE_TIMEOUT
          ini:
          - key: cache_timeout
            section: inventory
        
        type: integer

- compose
        create vars from jinja2 expressions
        [Default: {}]
        type: dictionary

- groups
        add hosts to group based on Jinja2 conditionals
        [Default: {}]
        type: dictionary

- keyed_groups
        add hosts to group based on the values of a variable
        [Default: []]
        type: list

= plugin
        token that ensures this is a source file for the 'docker_machine' plugin.
        (Choices: docker_machine)

- split_separator
        for keyed_groups when splitting tags this is the separator to split the tag value on.
        [Default: :]
        type: str

- split_tags
        for keyed_groups add two variables as if the tag were actually a key value pair separated by a colon, instead of just a single value.
        [Default: False]
        type: bool

- strict
        If true make invalid entries a fatal error, otherwise skip and continue
        Since it is possible to use facts in the expressions they might not always be available and we ignore those errors by default.
        [Default: False]
        type: boolean

- verbose_output
        Toggle to (not) include all available nodes metadata (e.g. Image, Region, Size, HostOptions, SwarmOptions, EngineOptions)
        [Default: True]
        type: bool


REQUIREMENTS:  Docker Machine

NAME: docker_machine
PLUGIN_TYPE: inventory

EXAMPLES:

# Minimal example
plugin: docker_machine

# Example using constructed features to create groups
# keyed_groups may be used to create custom groups
strict: False
keyed_groups:
  - prefix: tag
    key: 'dm_tags'

# Example using tag splitting where the tag is like 'dm_tag_gantry_component:routinator'
strict: False
split_tags: True
split_separator: ":"
keyed_groups:
  - prefix: gantry_component
    key: 'dm_tag_gantry_component'
```

END
