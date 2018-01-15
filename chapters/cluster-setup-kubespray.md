# Kubernetes - Cluster Setup using Kubespray

Kubespray is an *Ansible* based kubernetes provisioner. It helps us to setup a production grade, highly available and highly scalable Kubernetes cluster.

## Prerequisites
  * Operating System: Ubuntu 16.04
  * Number of VMs: 3/4
  * Memory: 2 GB
  * Hard disk: 10/20 GB

## Setting up the environment
### Step 1: Install Ansible

`On workstation`

Ansible is the base provisioner in our cluster. But installing Ansible is out of the scope of this training. You can learn about installing and configuring Ansible from [here.](http://docs.ansible.com/ansible/latest/intro_installation.html)

### Step 2: Setup passwordless SSH

`On workstation`

Ansible uses passwordless ssh<sup>1</sup> to create the cluster. Let us see how to set it up from your *workstation*.
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

```
cat ~/.ssh/id_rsa.pub | ssh ubuntu@10.40.1.26 'cat >> ~/.ssh/authorized_keys'
```

This will copy our newly generated public key to the remote machine. After running this command you will be able to SSH into the machine directly without using a password. Replace *10.40.1.26* with your respective machine's IP.

### Step 3: Enable IPv4 Forwarding

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

### Step 4: Stop and Disable UFW Service

`On all nodes`

Execute the following commands to stop and disable ufw service.

```
systemctl stop ufw.service
systemctl disable ufw.service
```

### Step 5: Install Python

`On all nodes`

Ansible needs python to be installed on all the machines.

```
sudo apt update
sudo apt install python
```



---
<sup>1</sup> You can use private key / password instead of passwordless ssh. But it requires additional knowledge in using Ansible.

## Configuring Kubespray

### Clone the Repository

Kubespray is hosted on GitHub. Let us the clone the [official repository](https://github.com/kubernetes-incubator/kubespray.git).

```
git clone https://github.com/kubernetes-incubator/kubespray.git
```
