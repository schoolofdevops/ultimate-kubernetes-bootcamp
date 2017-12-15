## Deploying Pods

Life of a pod

  * Pending : in progress
  * Running
  * Succeeded : successfully exited
  * Failed
  * Unknown

### Probes
  * livenessProbe : Containers are Alive
  * readinessProbe : Ready to Serve Traffic


### Resource Configs

Each entity created with kubernetes is a resource including pod, service, deployments, replication controller etc. Resources can be defined as YAML or JSON.  Here is the syntax to create a YAML specification.

**AKMS** => Resource Configs Specs

```
apiVersion: v1
kind:
metadata:
spec:
```

Spec Schema: https://kubernetes.io/docs/user-guide/pods/multi-container/

#### Common Configurations

Througout this tutorial, we would be deploying differnt components of  example voting application. Lets assume we are deploying it in a **dev** environment.  Lets create the common specs for this app with the AKMS schema discussed above.

file: common.yml

```
apiVersion: v1
kind:
metadata:
  name: vote
  labels:
    app: vote
    role: ui
    tier: front
spec:
```



Lets now create the  Pod config by adding the kind and specs to above schema.

Filename: vote-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: vote
  labels:
    app: vote
    role: ui
    tier: front
spec:
  containers:
    - name: vote
      image: schoolofdevops/vote:latest
      ports:
        - containerPort: 80
```


### Launching and operating a Pod

Syntax:

```
 kubectl apply -f FILE
```

To Launch pod using configs above,

```
kubectl apply -f vote-pod.yaml

```

To view pods

```
kubectl get pods

kubectl get pods vote
```

To get detailed info

```
kubectl describe pods vote
```

[Output:]
```
Name:           vote
Namespace:      default
Node:           kube-3/192.168.0.80
Start Time:     Tue, 07 Feb 2017 16:16:40 +0000
Labels:         app=voting
Status:         Running
IP:             10.40.0.2
Controllers:    <none>
Containers:
  vote:
    Container ID:       docker://48304b35b9457d627b341e424228a725d05c2ed97cc9970bbff32a1b365d9a5d
    Image:              schoolofdevops/vote:latest
    Image ID:           docker-pullable://schoolofdevops/vote@sha256:3d89bfc1993d4630a58b831a6d44ef73d2be76a7862153e02e7a7c0cf2936731
    Port:               80/TCP
    State:              Running
      Started:          Tue, 07 Feb 2017 16:16:52 +0000
    Ready:              True
    Restart Count:      0
    Volume Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2n6j1 (ro)
    Environment Variables:      <none>
Conditions:
  Type          Status
  Initialized   True
  Ready         True
  PodScheduled  True
Volumes:
  default-token-2n6j1:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-2n6j1
QoS Class:      BestEffort
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath           Type            Reason          Message
  ---------     --------        -----   ----                    -------------           --------        ------          -------
  21s           21s             1       {default-scheduler }                            Normal          Scheduled       Successfully assigned vote to kube-3
  20s           20s             1       {kubelet kube-3}        spec.containers{vote}   Normal          Pulling         pulling image "schoolofdevops/vote:latest"
  10s           10s             1       {kubelet kube-3}        spec.containers{vote}   Normal          Pulled          Successfully pulled image "schoolofdevops/vote:latest"
  9s            9s              1       {kubelet kube-3}        spec.containers{vote}   Normal          Created         Created container with docker id 48304b35b945; Security:[seccomp=unconfined]
  9s            9s              1       {kubelet kube-3}        spec.containers{vote}   Normal          Started         Started container with docker id 48304b35b945
```

Commands to operate the pod

```
kubectl exec -it vote ps sh

kubectl exec -it vote  sh

kubectl logs vote

```

delete
```
kubectl delete pod vote

kubectl get pods
```

## Attach a Volume to the Pod

Lets create a pod for database and attach a volume to it. To achieve this we will need to

  * create a **volumes** definition
  * attach volume to container using **VolumeMounts** property

Volumes are of two types:
  * emptyDir
  * hostPath

File: db-pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: db
  labels:
    app: postgres
    role: database
    tier: back
spec:
  containers:
    - name: db
      image: postgres:9.4
      ports:
        - containerPort: 5432
      volumeMounts:
      - name: db-data
        mountPath: /var/lib/postgresql/data
  volumes:
  - name: db-data
    emptyDir: {}

```
To create this pod,

```
kubectl apply -f db-pod.yaml

kubectl describe pod db

kubectl get events
```


## Selecting Node to run on

```
kubectl get nodes --show-labels

kubectl label nodes <node-name> rack=1

kubectl get nodes --show-labels

```

Update pod definition with nodeSelector

file: vote-pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: vote
  labels:
    app: vote
    role: ui
    tier: front
spec:
  containers:
    - name: vote
      image: schoolofdevops/vote:latest
      ports:
        - containerPort: 80
  nodeSelector:
    rack: '1'
```

For this change, pod needs to be re created.

```
kubectl apply -f vote-pod.yaml
```

## Creating Multi Container Pods

file: multi_container_pod.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    app: nginx
    role: ui
    tier: front
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
      - name: data
        mountPath: /opt/d1
    - name: loop
      image: schoolofdevops/loop
      volumeMounts:
      - name: data
        mountPath: /opt/d1
  volumes:
  - name: data
    emptyDir: {}
```

To create this pod

```
kubectl apply -f multi_container_pod.yml
```

Check Status

```
root@kube-01:~# kubectl get pods
NAME      READY     STATUS              RESTARTS   AGE
nginx     0/2       ContainerCreating   0          7s
vote      1/1       Running             0          3m
```

Checking logs, logging in
```
kubectl logs  web  -c loop
kubectl logs  web  -c nginx

kubectl exec -it web  sh  -c nginx
kubectl exec -it web  sh  -c loop

```

## Exercise

Create a pod definition for redis and deploy.

#### Reading List :

Node Selectors, Affinity
https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
