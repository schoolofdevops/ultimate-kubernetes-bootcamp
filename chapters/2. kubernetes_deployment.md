## Creating a Deployment

A Deployment is a higher level abstraction which sits on top of replica sets and allows you to manage the way applications are deployed, rolled back at a controlled rate.

Topics    
  * Rollout a Replicaset  
  * Deploy a new version : Creates a new replica set every time, moves pods from RS(n) to RS(n+1)  
  * Rollback to previous RS      
  * Auto Scaling  
  * Pause Deployments  


File: vote-deploy.yaml

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: vote
  namespace: dev
spec:
  replicas: 8
  selector:
    matchLabels:
      tier: front
      app: vote
    matchExpressions:
      - {key: tier, operator: In, values: [front]}
  revisionHistoryLimit: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 2
  minReadySeconds: 20
  paused: false
  template:
    metadata:
      labels:
        app: vote
        role: ui
        tier: front
    spec:
      containers:
      - image: schoolofdevops/vote
        imagePullPolicy: Always
        name: vote
        ports:
        - containerPort: 80
          protocol: TCP
```

Deployment spec (deployment.spec) contains the following,

  * replicaset specs
    * selectors  
    * replicas  
  * deployment spec
    * strategy
    * rollingUpdate
    * minReadySeconds
  * pod template
    * metadata, labels
    * container specs



Lets  create the Deployment
```
kubectl apply -f vote_deploy.yaml --record
```

Now that the deployment is created. To validate,

```
kubectl get deployment
kubectl get rs
kubectl rollout status deployment/vote
kubectl get pods --show-labels
```
Sample Output
```
kubectl get deployments
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
vote   3         3         3            1           3m
```


## Scaling a deployment  

To scale a deployment in Kubernetes:

```
kubectl scale deployment/vote --replicas=5
```

Sample output:
```
kubectl scale deployment/vote --replicas=5
deployment "vote" scaled
```
