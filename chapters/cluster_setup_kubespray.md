# Kubernetes - Cluster Setup using Kubespray

Kubespray is an *Ansible* based kubernetes provisioner. It helps us to setup a production grade, highly available and highly scalable Kubernetes cluster.

## Prerequisites

### Hardware Pre requisites  

  * 4 Nodes: Virtual/Physical Machines  
  * Memory: 2GB   
  * CPU: 1 Core
  * Hard disk: 20GB available  


### Software Pre Requisites  

`On All Nodes`

  * Ubuntu 16.04 OS  
  * Python  
  * SSH Server
  * Privileged user

`On Ansible Control  Node`    

  * Ansible version  2.4 or greater  
  * Jinja  

### Networking Pre Requisites

  * Internet access to download docker images and install softwares
  * IPv4 Forwarding should be enabled  
  * Firewall should allow ssh access as well as ports required by Kubernetes. Internally open all the ports between node.




## Architecture of a high available kubernetes cluster


## Preparing the nodes

### Install Python

`On all nodes`

Ansible needs python to be installed on all the machines.

```
sudo apt update
sudo apt install python3
sudo apt install python3-pip
```


### Enable IPv4 Forwarding

`On all nodes`

```
ssh ubuntu@10.40.1.26
sudo su

vim /etc/sysctl.conf
```
Enalbe IPv4 forwarding by uncommenting the following line

```
net.ipv4.ip_forward=1
```

### Stop and Disable Firewall (UFW Service)

`On all nodes`

Execute the following commands to stop and disable ufw service.

```
systemctl stop ufw.service
systemctl disable ufw.service
```

### Set Locale

```
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
sudo dpkg-reconfigure locales
```
Do no select any other locale in the menu. Just press <OK> in the next two screens.

### Setup passwordless SSH between ansible controller and kubernetes nodes

`On control node`

Ansible uses passwordless ssh<sup>1</sup> to create the cluster. Let us see how to set it up from your *control node*.

Generate ssh keypair if not present already using the following command.

```
ssh-keygen -t rsa

Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:yC4Tl6RYc+saTPcLKFdGlTLOWOIuDgO1my/NrMBnRxA ubuntu@node1
The key's randomart image is:
+---[RSA 2048]----+
|   E    ..       |
|  . o +..        |
| . +o*+o         |
|. .o+Bo+         |
|. .++.X S        |
|+ +ooX .         |
|.=.OB.+ .        |
| .=o*= . .       |
|  .o.   .        |
+----[SHA256]-----+
```

Just leave the fields to defaults. This command will generate a public key and private key for you.

Copy over the public key to all nodes.

Example,  assuming **ubuntu** as the user which has a privileged access on the node with ip address **10.10.1.101**,

```
ssh-copy-id ubuntu@10.10.1.101
```


This will copy our newly generated public key to the remote machine. After running this command you will be able to SSH into the machine directly without using a password. Replace *10.40.1.26* with your respective machine's IP.

e.g.

```
ssh ubuntu@10.10.1.101
```


Make sure to copy the public key to all kubernetes nodes. `Replace username with the actual user on your system`.
If the above mentioned command fails, then copy your public key and paste it in the remote machine's `~/.ssh/authorized_keys` file.

e.g. (Only if ssh-copy-id fails)
```
cat ~/.ssh/id_rsa.pub
ssh ubunut@10.10.1.101
vim ~/.ssh/authorized_keys
# Paste the public key
```


## Setup Ansible Control node and Kubespray



`On control node`

Kubespray is hosted on GitHub. Let us the clone the [official repository](https://github.com/kubernetes-incubator/kubespray.git).

```
git clone https://github.com/kubernetes-incubator/kubespray.git
cd kubespray
```

### Install Prerequisites
Install the python dependencies. **This step installs Ansible as well. You do not need to install Ansible separately**.

```
sudo pip3 install -r requirements.txt
```

### Set Remote User for Ansible


Add the following section in ansible.cfg file

```
remote_user=ubuntu
```

If the user you are going to connect is differnt, use that instead.

Your *ansible.cfg* file should look like this.

```
[ssh_connection]
pipelining=True
ssh_args = -o ControlMaster=auto -o ControlPersist=30m -o ConnectionAttempts=100 -o UserKnownHostsFile=/dev/null
#control_path = ~/.ssh/ansible-%%r@%%h:%%p
[defaults]
host_key_checking=False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp
stdout_callback = skippy
library = ./library
callback_whitelist = profile_tasks
roles_path = roles:$VIRTUAL_ENV/usr/local/share/kubespray/roles:$VIRTUAL_ENV/usr/local/share/ansible/roles
deprecation_warnings=False
remote_user=ubuntu
```

### Create Inventory


```
cp -rfp inventory/sample inventory/prod
```

where **prod** is the custom configuration name. Replace is with whatever name you would like to assign to the current cluster.

To build the inventory file, execute the inventory script along with the IP addresses of our cluster as arguments

```
CONFIG_FILE=inventory/prod/hosts.ini python3 contrib/inventory_builder/inventory.py 10.10.1.101 10.10.1.102 10.10.1.103 10.10.1.104
```

Where replace the IP addresses (e.g. 10.10.1.101) with the actual IPs of your nodes

Once its run, you should see an inventory file generated which may look similar to below


`file: inventory/prod/hosts.ini`
```
[all]
node1    ansible_host=10.10.1.101 ip=10.10.1.101
node2    ansible_host=10.10.1.102 ip=10.10.1.102
node3    ansible_host=10.10.1.103 ip=10.10.1.103
node4    ansible_host=10.10.1.104 ip=10.10.1.104

[kube-master]
node1
node2

[kube-node]
node1
node2
node3
node4

[etcd]
node1
node2
node3

[k8s-cluster:children]
kube-node
kube-master

[calico-rr]

[vault]
node1
node2
node3
```

## Provisioning  kubernetes cluster with kubespray

`On control node`

We are set to provision the cluster. Run the following ansible-playbook command to provision our Kubernetes cluster.

```
ansible-playbook -b -v -i inventory/prod/hosts.ini cluster.yml
```

Option -i = Inventory file path  
Option -b = Become as root user  
Option -v = Give verbose output  


This Ansible run will take around 30 mins to complete.

## Install Kubectl

`On any master node`

Before we proceed further, we will need to install **kubectl** binary in our control node.
To install **kubectl** on Ubuntu

```
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo touch /etc/apt/sources.list.d/kubernetes.list
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```


**Note**: You could also install kubectl on your laptop/workstation. To learn how to install it for your OS,   [refer to the  procedure here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).


## Getting the Kubernetes Configuration File

`On same  master node as above`

Once the cluster setup is done, we have to copy over the cluster config file from the master machine. We will discuss about this file extensively in the next chapter.

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Check the State of the Cluster

`On the node where kubectl is setup`

Let us check the state of the cluster by running,

```
kubectl cluster-info

Kubernetes master is running at https://10.10.1.101:6443
KubeDNS is running at https://10.10.1.101:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```
kubectl get nodes

NAME      STATUS    ROLES         AGE       VERSION
node1     Ready     master,node   21h       v1.9.0+coreos.0
node2     Ready     master,node   21h       v1.9.0+coreos.0
node3     Ready     node          21h       v1.9.0+coreos.0
node4     Ready     node          21h       v1.9.0+coreos.0
```

If you are able to see this, your cluster has been set up successfully.

---
<sup>1</sup> You can use private key / password instead of passwordless ssh. But it requires additional knowledge in using Ansible.


##### References  

  * [Installing Kubernetes On Premises/On Cloud with Kubespray ](https://kubernetes.io/docs/setup/custom-cloud/kubespray/)  
  * [Kubespray on Github](https://github.com/kubernetes-incubator/kubespray)  
