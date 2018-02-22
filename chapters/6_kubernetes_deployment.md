## Creating a Deployment

A Deployment is a higher level abstraction which sits on top of replica sets and allows you to manage the way applications are deployed, rolled back at a controlled rate.

Deployment has mainly two responsibilities,

  * Provide Fault Tolerance: Maintain the number of replicas for a type of service/app. Schedule/delete pods to meet the desired count.
  * Update Strategy: Define a release strategy and update the pods accordingly.


`File: k8s-code/projects/mogambo/dev/frontend-deploy.yml`

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: front-end
  namespace: mogambo
spec:
  replicas: 3
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
        app: front-end
        role: ui
        tier: front
    spec:
      containers:
      - image: schoolofdevops/frontend
        imagePullPolicy: Always
        name: front-end
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
kubectl apply -f frontend-deploy.yml --record
```

Now that the deployment is created. To validate,

```
kubectl get deployment
kubectl get rs
kubectl get deploy,pods,rs
kubectl rollout status deployment/front-end
kubectl get pods --show-labels
```
Sample Output
```
kubectl get deployments

[output]
NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
front-end   3         3         3            3           59s
```


## Scaling a deployment  

To scale a deployment in Kubernetes:

```
kubectl scale deployment/front-end --replicas=5

[output]
deployment "front-end" scaled
```
