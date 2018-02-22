### Recreate

When the **Recreate** deployment strategy is used,
  * The old pods will be deleted
  * Then the new pods will be created.

This will create some downtime in our stack. Hence, this strategy is only recommended for development use cases.

Let us change the deployment strategy to **recreate** and image tag to *latest*.

`File: k8s-code/projects/mogambo/dev/catalogue-deploy.yml`

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: catalogue
  namespace: mogambo
  labels:
    app: catalogue
    env: dev
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: catalogue
        env: dev
    spec:
      tolerations:
        - key: "dedicate"
          operator: "Equal"
          value: "catalogue"
          effect: "NoExecute"
      containers:
        - name: catalogue
          image: schoolofdevops/catalogue:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 80

```

### Pause/Unpause

When you are in the middle of a new update for your application and you found out that the application is behaving as intended. In those situations,
  1. we can pause the update,
  2. fix the issue,
  3. resume the update.

Let us change the image tag to V2 in pod spec.

`File: catalogue-deploy.yml`

```
      containers:
        - name: catalogue
          image: schoolofdevops/catalogue:V2
          imagePullPolicy: Always
          ports:
            - containerPort: 80
```

Apply the changes.

```
kubectl apply -f catalogue-deploy.yml

kubectl get pods

[Output]
NAME                         READY     STATUS         RESTARTS   AGE
catalogue-6c4f7b49d8-g5dgc   1/1       Running        0          16m
catalogue-765554cc7-xsbhs    0/1       ErrImagePull   0          9s
```

Our deployment is failing. From some debugging, we can conclude that we are using a wrong image tag.

Now pause the update

```
kubectl rollout pause
```

Set the deployment to use **v2** version of the image.

```
kubectl set image deployment catalogue catalogue=schoolofdevops/catalogue:v2

[output]
deployment "catalogue" image updated
```

Now resume the update

```
kubectl rollout resume deployment catalogue
kubectl rollout status deployment catalogue

[Ouput]
deployment "catalogue" successfully rolled out
```

```
kubectl get pods

[Output]
NAME                         READY     STATUS    RESTARTS   AGE
catalogue-6875c8df8f-k4hls   1/1       Running   0          1m
```

When we do this, we skip the need of creating a new rolling update altogether.

### Additional Release Strategies

Along with rolling update and recreate strategies, we can also do,

  * Canary releases
  * Blue/Green deployments.

### Blue/Green Deployments


---
2. Recreate type gives an error. Need to be fixed.
3. Add Use-cases for recreate strategy type.
4. Add additional details of why we skip creating a replica set / rolling update in pause/unpause section.
