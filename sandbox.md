## Build AWX HA via `Vagrant Up`
![Vagrant credits](https://www.vagrantup.com/assets/images/og-image-9baf72d1.png)

#### Pre-requisites:
1. Clone the AWX HA repository
2. Install [Vagrant](https://www.vagrantup.com/downloads.html), [VirtualBox](https://www.virtualbox.org/wiki/Downloads) & [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

#### Update the Sandbox environemnt values
1. [inventory/hosts](./inventory/hosts) ( Recommended to go through the [known_issues](https://github.com/sujiar37/AWX-HA-InstanceGroup#known-issues) )
```yaml
[all]

[awx_instance_group_web]
10.10.10.21
10.10.10.22

[awx_instance_group_task]
10.10.10.23
```
2. [inventory/group_vars/all.yml](./inventory/group_vars/all.yml)
```yaml
---
ansible_user: "vagrant"
ansible_ssh_pass: "Test123"
ansible_become_pass: "{{ansible_ssh_pass}}"

### AWX Default Settings
awx_unique_secret_key: awxsecret
awx_admin_default_pass: password

### Postgre DB details
pg_db_host: "10.10.10.20"
pg_db_pass: "awxpass"
pg_db_port: "5432"
pg_db_user: "awx"
pg_db_name: "awx"


###  RabbitMQ default settings
rabbitmq_cookie: "cookiemonster"
rabbitmq_username: "awx"
rabbitmq_password: "password"
```

#### Let's run it
```bash
$ cd AWX-HA-InstanceGroup/

$ vagrant up;ansible-playbook -i inventory/hosts awx_ha.yml --verbose
```
Once the command has succeded, it will bring the sandbox environment with HA mode and the same can access via either http://10.10.10.21 / http://10.10.10.22

Once you are done with sandbox environment, you could destroy those via,
```bash
$ cd AWX-HA-InstanceGroup/

$ vagrant destroy -f
```

#### Vagrant up, what is it?
Vagrant is a DevOps tool developed by HashiCorp that widely used for automating development activities. It can talk with your virtual appliances [oracle virtualbox etc]based on your [Vagrantfile](./Vagrantfile) and bring a sandbox environment for you to play around with it. 

#### Disclaimer: I try to reduce scripting whenever I could instead bring some innovative solutions with an optimal level of transparency and readability. Hence, my [Vagrantfile](./Vagrantfile) here isn't no where perfect when it comes with the best practices. I am sorry for that, but it just works with our case !!! Happy Automating !!!