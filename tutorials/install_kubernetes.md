# Kubernetes

Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.


### Creating Kubernetes Repository

We need to create a repository to download Kubernetes.

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```
```
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
```


### Installation of the packages

We should update the machines before installing so that we can update the repository.
```
apt-get update -y
```
Installing all the packages with dependencies:
```
apt-get install -y docker.io kubelet kubeadm kubectl kubernetes-cni
```
```
rm -rf /var/lib/kubelet/*
```

The above steps has to be followed in all the nodes.
### Initialization of Master:

Choose any one node as a master node and start the master service.

```
kubeadm init
```

##### After finishing the last line of output will be:

Kubernetes master initialised successfully!

You can connect any number of nodes by running:

kubeadm join –token <token> <master-ip>


### Initialization of the Minions

When you run the master service the last line would give you the command that should be run in all the other nodes in the cluster so that all the nodes will become a minion node.

```
kubeadm join –token token master-ip
```

Master and minion Setup is done
