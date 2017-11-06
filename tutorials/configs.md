### Topics

  * Configs
  * Context
  * Namespaces


Check current config
```
kubectl config view
```

You could also examine the current configs in file **cat ~/.kube/config**

### Creating a dev namespace

Namespaces offers separation of resources running on the same physical infrastructure into virtual clusters. It is typically useful in mid to large scale environments with multiple projects, teams and need separate scopes. It could also be useful to map to your workflow stages e.g. dev, stage, prod.   

Lets create a namespace called **dev**  

file: dev_ns.yml
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
kubectl apply -f dev_ns.yml
```


And switch to it
```
kubectl config set-context $(kubectl config current-context) --namespace=dev

```
