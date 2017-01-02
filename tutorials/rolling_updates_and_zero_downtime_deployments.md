# Rolling Updates and Zero Downtime Deployments

A rolling update works by:
1. Creating a new replication controller with the updated configuration.
2. Increasing/decreasing the replica count on the new and old controllers until the correct number of replicas is reached.
3. Deleting the original replication controller.

#### Rolling Updates
Lets create a my-nginx RC.

```
touch my-nginx.yaml
```

```
vi my-nginx.yaml
```

Paste the below contents:
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Now we can create a RC:
```
kubectl create -f my-nginx.yaml
```

We have created 5 pods with nginx of the version 1.7.9.

Now we going to give rolling update by changing the version of the nginx.

```
kubectl rolling-update my-nginx --image=nginx:1.9.1
```
Thus we have updated our pods with the version of 1.9.1

#### Rollback
If we occurd any error when updating we can rollback by:
```
kubectl rolling-update my-nginx --rollback
```
