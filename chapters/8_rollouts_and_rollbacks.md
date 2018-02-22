## Rolling updates with deployments

Update the version of the image in frontend-deploy.yml

`File: k8s-code/projects/mogambo/dev/frontend-deploy.yaml`

```
...
    app: front-end
    spec:
      containers:
      - image: schoolofdevops/frontend:dogs

```

Apply Changes and monitor the rollout

```
kubectl apply -f frontend-deploy.yaml
kubectl rollout status deployment/front-end
```

## Rolling Back a Failed Update

Lets update the image to a tag which is non existent. We intentionally introduce this intentional error to fail fail the deployment.

File: frontend-deploy.yaml
```
...
    app: vote
    spec:
      containers:
      - image: schoolofdevops/frontend:movi

```

Do a new rollout and monitor

```
kubectl apply -f frontend-deploy.yml
kubectl rollout status deployment/front-end
```

Also watch the pod status which might look like

```
vote-3040199436-sdq17   1/1       Running            0          9m
vote-4086029260-0vjjb   0/1       ErrImagePull       0          16s
vote-4086029260-zvgmd   0/1       ImagePullBackOff   0          15s
vote-rc-fsdsd               1/1       Running            0          27m
vote-rc-mcxs5               1/1       Running            0
```

To get the revision history and details  
```
kubectl rollout history deployment/front-end
kubectl rollout history deployment/front-end --revision=x
[replace x with the latest revision]
```

[Sample Output]

```
root@kube-01:~# kubectl rollout history deployment/front-end
deployments "front-end"
REVISION	CHANGE-CAUSE
1		kubectl scale deployment/front-end --replicas=5
3		<none>
6		<none>
7		<none>

root@kube-01:~# kubectl rollout history deployment/front-end --revision=7
deployments "front-end" with revision #7
Pod Template:
  Labels:	app=front-end
	env=dev
	pod-template-hash=4086029260
	role=ui
	stack=mogambo
	tier=front
  Containers:
   vote:
    Image:	schoolofdevops/frontend:movi
    Port:	8079/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

To undo rollout,

```
kubectl rollout undo deployment/front-end
```

or

```
kubectl rollout undo deployment/front-end --to-revision=1
kubectl get rs
kubectl describe deployment front-end
```
