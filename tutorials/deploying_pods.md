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
    stack: voting
    app: vote
    role: ui
    tier: front
    env: dev
spec:
```



Lets now create the  Pod config by adding the kind and specs to above schema.

Filename: vote_pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: vote
  labels:
    stack: voting
    app: vote
    role: ui
    tier: front
    env: dev
spec:
  containers:
    - name: vote
      image: schoolofdevops/vote:latest
      ports:
        - containerPort: 80
```


### Launching the Pod

Syntax:

```
 kubectl apply -f FILE
```

To Launch pod using configs above,

```
kubectl apply -f vote_pod.yaml

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

To run commands against a pod,

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

## Attach a Volume to a Pod

Lets create a pod for database and attach a volume to it. To achieve this we will need to

  * create a **volumes** definition
  * attach volume to container using **VolumeMounts** property

Volumes are of two types:
  * emptyDir
  * hostPath

File: db_pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: db
  labels:
    stack: voting
    app: postgres
    role: database
    tier: back
    env: dev
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
kubectl apply -f db_pod.yaml

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

file: vote_pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: vote
  labels:
    stack: voting
    app: vote
    role: ui
    tier: front
    env: dev
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
kubectl apply -f vote_pod.yaml
```

## Exercise

Create a pod definition for redis and deploy.

#### Reading List :

Node Selectors, Affinity
https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
