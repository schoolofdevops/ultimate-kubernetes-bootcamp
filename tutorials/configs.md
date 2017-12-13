
In this lesson we are going to cover the following topics

  * Setting up monitors
  * Configs
  * Context
  * Namespaces


## Setup monitoring console for Kubernetes
Unix **screen** is a great utility for a devops professional. You could setup a simple monitoring for kubernetes cluster using a **screenrc** script as follows on kube-01 node (node where *kubectl* is configured)

file: k8s-code/monitoring/rc.screenrc

```
screen watch -n 1 kubectl get pods
split
focus down
screen watch -n 1 kubectl get rc
focus bottom
```

Open a dedicated terminal to run this utility.  Launch it using

```
screen -c monitoring/rc.screenrc

```


## Listing Configurations

Check current config
```
kubectl config view
```

You could also examine the current configs in file **cat ~/.kube/config**

## Creating a dev namespace

Namespaces offers separation of resources running on the same physical infrastructure into virtual clusters. It is typically useful in mid to large scale environments with multiple projects, teams and need separate scopes. It could also be useful to map to your workflow stages e.g. dev, stage, prod.   

Lets create a namespace called **dev**  

file: dev_ns.yaml
```
kind: Namespace
apiVersion: v1
metadata:
  name: dev
  labels:
    name: dev
```

To create namespace

```
kubectl apply -f dev_ns.yaml
```


And switch to it
```
kubectl config set-context $(kubectl config current-context) --namespace=dev

```
