# Ansible deployment for AIDMOIt

This project aims to deploy a datalake and its ecosystem using  **[Ansible](https://docs.ansible.com/)** and/or **[Vagrant](https://www.vagrantup.com/)**.

Acutally, this project does (you have to choose if you want 1 single node HDFS or a cluster HDF) :
* Deploy a mono-node HDFS on :
	* [Your own computer using a virtual machine with Vagrant & ansible provision](#deploy-mono-node-hdfs-on-a-vm)
	* [a server using ansible](#deploy-mono-node-hdfs-on-a-server)
* Deploy a cluster HDFS on :
	* [Your own computer using as many virtual machine you want using Vagrant & ansible](#deploy-cluster-hdfs-with-multiple-vms)
	* [multiple servers using ansible](#deploy-cluster-hdfs-on-servers)

You can add, on top of your HDFS, un metadata system management with a search engine  : 
* Deploy a geonetwork as metadata system management
	* [Your own computer using a virtual machine with Vagrant & ansible provision](#geonetwork-with-vagrant-and-ansible)
	* [deploy on a server](#geonetwork-on-erver)

**Prod Environment** : Deploy a mononode HDFS on a server using ansible 
**Sandbox Environment**: Deploy a mononode HDFS on a VM from Vagrant & Ansible

## Prerequisites
* Ansible
	+ Install ansible on your own computer (for Ubuntu or Debian) :
	```shell
	apt-get install ansible
	```
	+ If you connect to your remote machine using password instead of ssh-key (as recommanded), you have to install this apt :
	```shell
	apt-get install sshpass
	```
	+ Configure ansible (allow becoming unprivileged user without error : "Failed to set permissions on the temporary files Ansible needs to create when becoming an unprivileged user")
	```shell
	sed -i 's/.*pipelining.*/pipelining = True/' /etc/ansible/ansible.cfg
	sed -i 's/.*allow_world_readable_tmpfiles.*/allow_world_readable_tmpfiles = True/' /etc/ansible/ansible.cfg
	```
* Vagrant with virtualox
	+ Install virtualbox on your own computer (for Ubuntu or Debian):
	```
	apt-get install virtualbox
	```
	+ Install vagrant on your own computer (for Ubuntu or Debian):
	```shell
	apt-get install vagrant
	```
	

### Ansible
#### Description of Ansible
[Wikipedia definition :](https://en.wikipedia.org/wiki/Ansible_(software))
> Ansible is an open-source software provisioning, configuration management, and application-deployment tool.[2] It runs on many Unix-like systems, and can configure both Unix-like systems as well as Microsoft Windows. It includes its own declarative language to describe system configuration.

Ansible is a good tool to deploy and maintain IT systems. Based on [Yaml ](https://en.wikipedia.org/wiki/YAML) configuration files, ansible makes it easy to describe your configuration and share it with your collaborators. 
Then you can deploy it to your infrastructure, you only need to have a ssh access to your servers.

#### Getting started
Ansible playbook is a list of system instructions which has to be send to a machine. That's why you only need 2 things :
- Get ansible installed on your own computer
- Have a remote machine (physical, vmware, virtualbox, docker, lxc, ...) with a ssh server running
### Vagrant & Ansible
#### Description of Vagrant
[Wikipedia definition :](https://en.wikipedia.org/wiki/Vagrant_(software))
> Vagrant is an open-source software product for building and maintaining portable virtual software development environments,[5] e.g. for VirtualBox, KVM, Hyper-V, Docker containers, VMware, and AWS. It tries to simplify the software configuration management of virtualizations in order to increase development productivity. Vagrant is written in the Ruby language, but its ecosystem supports development in a few languages. 

Vagrant manages your virtual machine (VM) on command line. The benefits are :
- Quickly create VM with a know & controlled environment
- Restore your VM to a known state
- Destribute yours VM easly 

Vagrant and ansible can be combined to create/deploy/maintain your VM as we do in this project

#### Getting started
You only need virtual box and vagrant installed on your computer. This project is going to create VM that you need for your datalake

# Deploy a mono-node HDFS 
## Deploy mono node HDFS on a VM
1. In cli : Go to the directory which contains the [VagrantFile ](vagrant/mononode/Vagrantfile)
2. In cli : start your VM:
```shell
vagrant up
```
Pay attention, some OS (as debian buster, need to be sudo for this instruction : sudo vagrant up)
## Deploy mono-node HDFS on a server
First configure your IP adress in the [inventory file](inventory/static)

Then run the script [ansible-launch.sh](ansible-launch.sh) :
```shell
/bin/bash ansible-launch.sh
```
# Deploy a HDFS cluster
## Deploy cluster HDFS with multiple VMs
1. Set your nodes' IP address in [VagrantFile](vagrant/cluster/Vagrantfile).
Inside this file, edit your network setting (as DNS nameserver: if your host machine is on a corporate network, your network administrator may have set rules about using DNS servers. In this case, your network allows only your company's DNS. Please edit them in the settings section *and* in provision shell in vagrantFile. If this is not the case (i.e. no DNS rules), please assign "false" to the COMPANY_NETWORK_DNS_RULE variable.
2. Declare those IP for ansible provision in [vars](playbook/roles/hosts-file/vars/main.yml). If you did not change IP setting, skip this step.
3. Configure your own computer to access to your nodes using their hostname (need for access to hadoop web ui)
	```shell
	vim /etc/hosts
	```	
	if you did not edit your cluster settings since you have git pulled, you may want to use this default settings:
	```shell
	10.0.0.9 aidmoit-geonetwork
	10.0.0.10 namenode
	10.0.0.11 datanode1
	10.0.0.12 datanode2
	```	
4. in cli : start your multiple VM from this [directory : vagrant/cluster](vagrant/cluster) :
	```shell
	vagrant up
	```
	Pay attention, some OS (as debian buster, need to be sudo for this instruction : sudo vagrant up)
5. Format HDFS :
 	* ssh on namenode 
 	```shell
 	vagrant ssh namenode
 	```
 	Pay attention, some OS (as debian buster, need to be sudo for this instruction : sudo vagrant up)
 	* in cli : as user hadoop : change directory & format HDFS
 	```shell
 	sudo su hadoop
 	cd /usr/local/hadoop/bin/
 	./hdfs namenode -format
 	exit
 	```
6. Start HDFS deamon on your cluser
  	* ssh on namenode
 	* in cli : as root : start service hadoop
 	 ```shell
 	sudo systemctl start hadoop
 	```
7. Verify your cluster is up: 
 	* on your own device, use a webbrowser
 	* go on [IP-of-your-namenode]:9870
 	if default : http://10.0.0.10:9870 or http://namenode:9870
## Deploy cluster HDFS on servers
work in progress

# Deploy GeoNetwork
## GeoNetwork With Vagrant and ansible
1. Set your nodes' IP address in [VagrantFile](vagrant/geonetwork/Vagrantfile).
Inside this file, edit your network setting (as DNS nameserver: if your host machine is on a corporate network, your network administrator may have set rules about using DNS servers. In this case, your network allows only your company's DNS. Please edit them in the settings section *and* in provision shell in vagrantFile. If this is not the case (i.e. no DNS rules), please assign "false" to the COMPANY_NETWORK_DNS_RULE variable.
2. in cli : start a geonetwork vm form this directory : [vagrant/geonetwork](vagrant/geonetwork) :
	```shell
	vagrant up
	```
	Pay attention, some OS (as debian buster, need to be sudo for this instruction : sudo vagrant up)
3. On your own computer, using a webrowser, go on http://10.0.0.9:8080/geonetwork (if IP address as default) or on http://aidmoit-geonetwork:8080/geonetwork

## GeoNetwork On server
work in progress

## License
![licence](https://img.shields.io/badge/Licence-GPL--3-blue.svg)

**Aidmoit's ansible deployment** is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.


This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.


You should have received a copy of the GNU General Public License
along with this program.  If not, see http://www.gnu.org/licenses
