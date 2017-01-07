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

### Installing Pod Network.

Installing Pod network is necessary because that all the pods should communicate to each other. It is necessary to do this before you try to deploy any applications to your cluster.

```
kubectl apply -f https://git.io/weave-kube
```

Now we need to check whether all the pods are running successfully:
```
kubectl get pods --all-namespaces
```

Check that all the pods are running.
It will take a few minutes.


### Kubernetes Dashboard

After the Pod networks is installled, We can install another add-on service which is Kubernetes Dashboard.

Installing Dashboard:
```
kubectl create -f https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml
```
This will create a pod for the Kubernetes Dashboard.


To access the Dashboard in th browser, run the below command
```
kubectl describe svc kubernetes-dashboard -n kube-system
```

Sample output:
```
kubectl describe svc kubernetes-dashboard -n kube-system
Name:                   kubernetes-dashboard
Namespace:              kube-system
Labels:                 app=kubernetes-dashboard
Selector:               app=kubernetes-dashboard
Type:                   NodePort
IP:                     10.98.148.82
Port:                   <unset> 80/TCP
NodePort:               <unset> 32756/TCP
Endpoints:              10.40.0.1:9090
Session Affinity:       None
```

Now check for the node port, here it is 32756, and go to the browser,

```
masterip:32756
```
The Dashboard Looks like:

![alt text](images/Kubernetes-Dashboard.png "Kubernetes Dashboard")
