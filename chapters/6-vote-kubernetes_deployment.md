## Creating a Deployment

A Deployment is a higher level abstraction which sits on top of replica sets and allows you to manage the way applications are deployed, rolled back at a controlled rate.

Deployment has mainly two responsibilities,

  * Provide Fault Tolerance: Maintain the number of replicas for a type of service/app. Schedule/delete pods to meet the desired count.
  * Update Strategy: Define a release strategy and update the pods accordingly.

```
/k8s-code/projects/instavote/dev/
cp vote-rs.yaml vote-deploy.yaml
```


Deployment spec (deployment.spec) contains everything that replica set has + strategy. Lets add it as follows,




File: vote-deploy.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote
  namespace: instavote
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  revisionHistoryLimit: 4
  paused: false
  replicas: 8
  minReadySeconds: 20
  selector:
    matchLabels:
      role: vote
    matchExpressions:
      - {key: version, operator: In, values: [v1, v2, v3]}
  template:
    metadata:
      name: vote
      labels:
        app: python
        role: vote
        version: v1
    spec:
      containers:
        - name: app
          image: schoolofdevops/vote:v1
          envFrom:
            - configMapRef:
                name: vote
          ports:
            - containerPort: 80
              protocol: TCP
```


This time, start monitoring with --show-labels options added.

```
watch -n 1 kubectl get  pod,deploy,rs,svc --show-labels
```


Lets  create the Deployment. Do monitor the labels of the pod while applying this.

```
kubectl apply -f vote-deploy.yaml
```

Observe the chances to pod labels, specifically the **pod-template-hash**.


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
kubectl scale deployment/vote --replicas=15
```

Sample output:
```
kubectl rollout status
kubectl scale deployment/vote --replicas=12
deployment "vote" scaled
```

You could also update the deployment by editing it.

```
kubectl edit deploy/vote
```

[change replicas to 15 from the editor, save and observe]


## Rolling Updates in Action

file: vote-deploy.yaml
```
spec:
  containers:
    - name: app
      image: schoolofdevops/vote:v1

```

apply

```
kubectl apply -f vote-deploy.yaml

kubectl rollout status
```

Observe rollout status and monitoring screen.



```

kubectl rollout history deploy/vote

kubectl rollout history deploy/vote --revision=1

```

## Undo and Rollback

file: vote-deploy.yaml
```
spec:
  containers:
    - name: app
      image: schoolofdevops/vote:rgjerdf

```

apply

```
kubectl apply -f vote-deploy.yaml

kubectl rollout status

kubectl rollout history deploy/vote

kubectl rollout history deploy/vote --revision=xx
```

where replace xxx with revisions

Find out the previous revision with sane configs.

To undo to a sane version (for example revision 3)

```
kubectl rollout undo deploy/vote --revision=3
```
