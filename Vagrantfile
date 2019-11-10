# -*- mode: ruby -*-
# vi: set ft=ruby :
ENV["LC_ALL"] = "en_US.UTF-8"

# This script to cofigure the nodes awx_web and awx_task on top of CentOS7
$script = <<-SCRIPT
echo setup the ssh password login
echo "vagrant:Test123" | chpasswd
echo update the ssh
sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
service sshd restart
# Get the IP address that VirtualBox has given this VM
IPADDR=`ip addr | grep -i eth1 | tail -1 | awk '{print $2}' | cut -d "/" -f1`
echo This VM has IP address $IPADDR
cat <<EOF >> /etc/hosts
10.10.10.20       node4.example.com       node4
10.10.10.21       node1.example.com       node1
10.10.10.22       node2.example.com       node2
10.10.10.23       node3.example.com       node3
EOF
echo showing hosts file details
cat /etc/hosts
SCRIPT

# This script to cofigure the node postgresql as docker on top of CentOS7
$postgre_script= <<-SCRIPT
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io -y
sudo systemctl start docker
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose;sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
sudo mkdir /pgdocker/
cat <<EOF > docker-compose.yml 
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
EOF
sudo docker-compose up -d
SCRIPT

# Vagrant configurations for awx_web, awx_task and postgresql nodes
Vagrant.configure("2") do |config|

  (1..3).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.box = "centos/7"
      node.vm.hostname = "node#{i}.example.com"
      node.vm.network "forwarded_port", guest: 80, host: 80, host_ip: "10.10.10.#{i + 20}"
      node.vm.network "private_network", ip: "10.10.10.#{i + 20}"
      node.vm.provision "shell", inline: $script
 
      node.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--cpuexecutioncap", "100"]
        v.memory = 1024
        v.cpus = 1
      end
    end
  end

  config.vm.define "postgre" do |external|
    external.vm.box = "centos/7"
    external.vm.hostname = "postgre"
    external.vm.hostname = "node4.example.com"
    external.vm.network "forwarded_port", guest: 5432, host: 5432, host_ip: "10.10.10.20"
    external.vm.network "private_network", ip: "10.10.10.20"
    external.vm.provision "shell", inline: $postgre_script

    external.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--cpuexecutioncap", "100"]
      v.memory = 512
      v.cpus = 1
    end
  end
end