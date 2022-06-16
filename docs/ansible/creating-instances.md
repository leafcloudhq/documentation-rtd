# Creating an instance with Ansible
In this tutorial I'm going to show an example of how to start a server and run a `apache2` on Leafcloud, using Ansible.

## Install ansible

you can use te package manager of your linux distribution

On Ubuntu:
```shell
sudo apt install ansible
```

On RHEL or similar:
```shell
sudo dnf install ansible
```

pip can be used on any operating system:
```shell
pip install ansible
```

## Install the openstack.cloud cllection

To be able to create a server on Leafcloud you will need an extra module collection
```shell
ansible-galaxy collection install openstack.cloud
```

## Create your playbooks

The following playbook `create-server.yaml` will create:

- A security group for http
- A security group rule for http
- A an ec1.small VM instance

```shell
---
  - name: Create a new instance and attaches to a network and passes metadata to the instance
    hosts: localhost
    connection: local
    tasks:
      - name: Create an http security group
        openstack.cloud.security_group:
          state: present
          name: http-hello-world
          description: security group for http
      - name: Create a security group rule for http
        openstack.cloud.security_group_rule:
          security_group: http-hello-world
          protocol: tcp
          port_range_min: 80
          port_range_max: 80
          remote_ip_prefix: 0.0.0.0/0
      - name: Create the server
        openstack.cloud.server:
          state: present
          auth:
            auth_url: https://create.leaf.cloud:5000
            username: you@yourcompany.test
            password: <password>
            project_name: my-project
          name: test-ansible
          image: 0e98efbf-de1e-4e58-bf77-1ca80668e305
          key_name: my_key
          timeout: 200
          flavor: ec1.small
          nics:
            - net-name: external
          security_groups:
            - http-hello-world
            - default
          boot_from_volume: yes
          volume_size: 20
      - name: get ip address of created server
        openstack.cloud.server_info:
          server: test-ansible
        register: result
      - name: Output the ip address
        debug:
          msg: "{{ result.openstack_servers[0].addresses.external[0].addr }}"
```

For installing apache2 you can make a playbook `install-apache2.yaml`
```shell
---
  - name: Install apache
    hosts: test-ansible
    become: true
    tasks:
      - name: install apache2
        apt:
          name: apache2
          update_cache: yes
          state: present
```

For the cleanup you can make the following playbook `cleanup.yaml`
```shell
  - name: Clean up server and security group
    hosts: localhost
    connection: local
    tasks:
      - name: Remove the server
        openstack.cloud.server:
          name: test-ansible
          state: absent
      - name: Remove http security group
        openstack.cloud.security_group:
          state: absent
          name: http-hello-world
      - name: Remove security group rule for http
        openstack.cloud.security_group_rule:
          security_group: http-hello-world
          state: absent
```

## Running your playbooks

You will start with `create-server.yaml`

```shell
ansible-playbook create-server.yaml
```

This will create a VM and output its ip address.
Copy the ip address and create your `inventory` file:

```shell
test-ansible  ansible_host=<ip-address> ansible_user=ubuntu
```
Replace `<ip-address>` with the ip address of your server and save the file.

Now you are ready to install `apache2`:
```shell
ansible-playbook -i inventory install-apache2.yaml
```

That's it, you can now type the ip address of your server into your favorire browser,
and you'll see the welcome page of apache2.

## Cleanup

To remove the instance and the security groups you've jsut created you can run:
```shell
ansible-playbook create-server.yaml cleanup.yaml
```

This will remove any security group and rule named `http-hello-world` in your project.
It will also remove any server in your project named `test-ansible`
