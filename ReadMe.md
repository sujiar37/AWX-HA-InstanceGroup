[![Build Status](https://dev.azure.com/sujiar37/AWX-HA-InstanceGroup%20-%20CI/_apis/build/status/sujiar37.AWX-HA-InstanceGroup?branchName=master)](https://dev.azure.com/sujiar37/AWX-HA-InstanceGroup%20-%20CI/_build/latest?definitionId=2&branchName=master)
# AWX V9.1.1 - Instance Group - HA

[AWX](https://github.com/ansible/awx) is an upstream project of Ansible Tower. I have been following this project since from the version `1.x` to the current latest version which is `9.x`. Below the diagram illustrates an overall idea about the clustering functionality in Ansible Tower version `3.X`. More likely the same functionality can achieve in AWX by tweaking few file modifications and settings. Hence, I came across a solution to automate this clustering process via playbook after I had a few insights from [AWX google groups](https://groups.google.com/forum/#!forum/awx-project) as well as the official Ansible Tower installation playbook. 

![AWX Job Runtime Behaviour](https://docs.ansible.com/ansible-tower/latest/html/administration/_images/tower-clustering-visual.png)

![AWX HA - Instance Group](https://drive.google.com/uc?export=view&id=1PUj3t3GSgU2ky8vm3uxMyzVHzdV4ZFv2)

## Points To Remember
1. PostgreSQL DB should be centralized since all the nodes will act as Primary-Primary. This playbook does not cover the installation of PostgreSQL DB, however you can build it your own / use below docker-compose information to deploy as a container on a separate node.

```bash
$ mkdir /pgdocker/
```
```yaml
$ cat docker-compose.yml 

version: '2'
services:
  postgres:
    image: postgres:10
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
2. Here is the inventory details I populate with ansible to add **'n'** number of hosts into the existing cluster. All I have to update the machine ip address of new node under **`[awx_instance_group_task]`** and run the playbook **`awx_ha.yml`**. In case if you want to enable web front end, then you can update the new machine information under **`[awx_instance_group_web]`** and run the playbook. One cool feature is, you can always perform `plug and play` with the hosts by using these two `awx_instance_group_web & awx_instance_group_task` inventory groups. It is all about your desire how many web nodes and task nodes you would like to have since HA doesn't require to run AWX web container in all nodes. Also, it is important that **all these nodes can communicate each other with their hostnames.**

```bash
$ cat inventory/hosts

[all]

## Inventory Group where you need HA in AWX with web front end 
[awx_instance_group_web]
Primary_Node_A
Primary_Node_B

## Inventory Group where you need HA in AWX without web front end
[awx_instance_group_task]
Primary_Node_C
Primary_Node_D
```


3. Following are the default [global variables](inventory/group_vars/all.yml) used across this playbook.

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
$ ansible-playbook -i inventory/hosts awx_ha.yml --verbose

# Only run any of the below commands if you don't wish to enable and configure rules in firewalld daemon, which is an optional.
$ ansible-playbook -i inventory/hosts awx_ha.yml --skip-tags fw_rules --verbose

                          [ OR ]

$ ansible-playbook -i inventory/hosts awx_ha.yml -e "fw_rules=false" --verbose
```

Running the above command with `--check` mode may fail in a new machines since there are few commands to check whether the RabbitMQ cluster is active / not. However, the issue won't trigger if you had run it to a machine which is in clustered already

#### If you had a wish to run all these to a sandbox environment before deploying to your actual servers, please check the [instructions for Vagrant here](./sandbox.md).

## Known Issues
As we all are aware, the initial deployment of AWX containers will try to access the DB and perform migration if required. In such cases, you may ended up seeing below error inside the affected task containers where it get locked up for DB access and will fail to join the cluster by throwing out below error,

```bash
$ docker container logs -f build_image_task_1 

    raise RuntimeError("No instance found with the current cluster host id")
RuntimeError: No instance found with the current cluster host id
2019-04-04 21:12:52,184 INFO exited: dispatcher (exit status 1; not expected)
```

This could be fixed by any of the below methods,

1. Deploy a single web node under `[awx_instance_group_web]` during initial run, wait for the migration to complete and let the AWX GUI comes up. After that you can pass **'n'** number of hosts into any of these inventory groups `[awx_instance_group_web] OR [awx_instance_group_task]` and those nodes will be added automatically in HA and will be visible from AWX portal. 

2. If you don't wish to follow the first method, then deploy to all and go to each nodes and try restart those affected containers once the DB migration completes. This would fix the issue and here is the references of few commands to perform that,

```bash
# ls 
docker-compose.yml  Dockerfile  Dockerfile.task  launch_awx.sh  launch_awx_task.sh  settings.py  system_uuid.txt

# pwd
/var/lib/awx/build_image

# docker-compose restart
Restarting build_image_task_1      ... done
Restarting build_image_memcached_1 ... done

```

### Last but not least, if you like this piece of work, kindly rate this repo by providing your valuable **`STAR`** input.