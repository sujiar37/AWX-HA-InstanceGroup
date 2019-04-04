# AWX V4.0.0 - Instance Group

AWX is an upstream project of Ansible Tower. I have been following that project since from the version `1.x` and the current latest version is `4.x`. Below the diagram illustrates an overall idea about the clustering functionality in Ansible Tower version `3.X`. The same functionality can achieve in AWX by tweaking few config settings. I came across a solution to automate this clustering process via playbook after I had received some insights from [AWX google groups](https://groups.google.com/forum/#!forum/awx-project) as well as the official Ansible Tower installation playbook. 

![AWX Job Runtime Behaviour](https://docs.ansible.com/ansible-tower/latest/html/administration/_images/tower-clustering-visual.png)


## Points To Remember
1. PostgreSQL DB should be centralized since all nodes will act as Primary-Primary. This playbook does not cover the installation of PostgreSQL DB, however you can build it your own / use below docker-compose informations to deploy as a container in a separate node.

```bash
$ mkdir /pgdocker/

$ cat docker-compose.yml 

version: '2'
services:
  postgres:
    image: postgres:9.6
    restart: unless-stopped
    volumes:
      - /pgdocker:/var/lib/postgresql/data:Z
    environment:
      POSTGRES_USER: awx
      POSTGRES_PASSWORD: awxpass
      POSTGRES_DB: awx
      PGDATA: /var/lib/postgresql/data/pgdata
    ports:
      - "5432:5432"
```
2. Here is the inventory details I populate with ansible to add **'n'** number of hosts into the existing cluster. All I need to add by updating the machine ip address under **`[awx_instance_group_agent]`** and run the playbook **`awx_ha_v4.yml`**.

```bash
$ cat inventory/hosts

[all]

[awx_instance_group_master]
Primary_Node_A

[awx_instance_group_agent]
Primary_Node_B
Primary_Node_C
Primary_Node_D
```


3. Following are the default [global variables](inventory/group_vars/main.yml) used across this playbook.

```yaml
### AWX Default Settings
awx_unique_secret_key: awxsecret
awx_admin_default_pass: password

### Postgre DB details
pg_db_host: "Database_Node_IP"
pg_db_pass: "awxpass"
pg_db_port: "5432"
pg_db_user: "awx"
pg_db_name: "awx"


###  RabbitMQ default settings
rabbitmq_cookie: "cookiemonster"
rabbitmq_username: "awx"
rabbitmq_password: "password"
```

## Let's Run it

```bash
ansible-playbook -i inventory/hosts playbooks/awx_ha_v4.yml --verbose
```

Running the above command with `--check` mode may fail for new machines since it is trying to execute few commands to make sure that whether the RabbitMQ cluster is active / not. However, the issue won't trigger if we run it to a machine which clustered already.



### Convey your thanks by clicking the **`STAR`** icon.