## Replication Controller Operations

A replication controller ensures that a specified number of pod “replicas” are running at any one time. If there are too many, it will kill some. If there are too few, it will start more.

#### Creating a replication controller

Follow the below command to create a RC:

```
touch replication.yaml
vi replication.yaml
```

Paste the below code in it:
```
apiVersion: v1
kind: ReplicationController
metadata:
  labels:
    run: voting-appp
  name: voting-appp
  namespace: default
spec:
  replicas: 1
  template:
    metadata:
      labels:
        run: voting-appp
    spec:
      containers:
      - image: venkatsudharsanam/votingapp-python:8.0.0
        imagePullPolicy: Always
        name: voting-appp
        ports:
        - containerPort: 80
          protocol: TCP
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      securityContext: {}
```

Save and exit

Now to create a Replication controller:
```
kubectl create -f replication.yaml
```

To list replication controllers on a cluster

```
kubectl get rc
```
Sample Output:
```
kubectl get rc
NAME          DESIRED   CURRENT   READY     AGE
voting-appp   1         1         1         1m
```
To view detailed information about a specific replication controller

```
kubectl describe rc voting-app
```

Sample Output:

```
kubectl describe rc voting-app
Name:           voting-appp
Namespace:      default
Image(s):       venkatsudharsanam/votingapp-python:8.0.0
Selector:       run=voting-appp
Labels:         run=voting-appp
Replicas:       1 current / 1 desired
Pods Status:    1 Running / 0 Waiting / 0 Succeeded / 0 Failed
No volumes.
Events:
  FirstSeen     LastSeen        Count   From                            SubObjectPath   Type            Reason                  Message
  ---------     --------        -----   ----                            -------------   --------        ------                  -------
  3m            3m              1       {replication-controller }                       Normal          SuccessfulCreate        Created pod: voting-appp-jgplb
```

#### Deleting the RC:

To delete a replication controller as well as the pods that it controls, use

```
kubectl delete rc voting-appp
```
