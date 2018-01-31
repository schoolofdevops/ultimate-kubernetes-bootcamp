## Creating a Deployment

A Deployment is a higher level abstraction which sits on top of replica sets and allows you to manage the way applications are deployed, rolled back at a controlled rate.

Deployment has mainly two responsibilities,

  * Provide Fault Tolerance: Maintain the number of replicas for a type of service/app. Schedule/delete pods to meet the desired count.
  * Update Strategy: Define a release strategy and update the pods accordingly.


File: /k8s-code/projects/instavote/dev/vote-deploy.yaml

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: vote
  namespace: instavote
spec:
  replicas: 8
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
kubectl apply -f vote-deploy.yaml --record
```

Now that the deployment is created. To validate,

```
kubectl get deployment
kubectl get rs
kubectl get deploy,pods,rs
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
