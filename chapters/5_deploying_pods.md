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

To list supported version of apis

```
kubectl api-versions
```

#### Common Configurations

Throughout this tutorial, we would be deploying different components of  example voting application. Lets assume we are deploying it in a **dev** environment. Lets create the common specs for this app with the AKMS schema discussed above.

`file: k8s-code/projects/mogambo/common.yml`

```
apiVersion: v1
kind:
metadata:
  name: front-end
  labels:
    app: front-end
    role: ui
    tier: front
spec:
```



Lets now create the  Pod config by adding the kind and specs to above schema.

Filename: k8s-code/pods/frontend-pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: front-end
  labels:
    app: front-end
    role: ui
    tier: front
spec:
  containers:
    - name: front-end
      image: schoolofdevops/front-end:latest
```

[Use this link to refer to pod spec](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#pod-v1-core)


### Launching and operating a Pod

Syntax:

```
 kubectl apply -f FILE
```

To Launch pod using configs above,

```
kubectl apply -f frontend-pod.yml

```

To view pods

```
kubectl get pods

kubectl get po -o wide

kubectl get pods front-end
```

To get detailed info

```
kubectl describe pods front-end
```

[Output:]
```
Name:         front-end
Namespace:    mogambo
Node:         gke-test-cluster-default-pool-d29bc0e9-jz8w/10.128.0.2
Start Time:   Thu, 22 Feb 2018 14:13:01 +0530
Labels:       app=front-end
              role=ui
              tier=front
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"front-end","role":"ui","tier":"front"},"name":"front-end","namespace":"mo...
Status:       Running
IP:           10.8.1.12
Containers:
  front-end:
    Container ID:   docker://587d8f8e40a23548eb2e2a70c1a43d32dce9922155781d1f76fbdd64f920c545
    Image:          schoolofdevops/frontend:latest
    Image ID:       docker-pullable://schoolofdevops/frontend@sha256:94b7a0843f99223a8a1d284fdeeb3fd5a731c03aea57a52751c6ebde40be1f50
    Port:           <none>
    State:          Running
      Started:      Thu, 22 Feb 2018 14:13:12 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-lfsps (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-lfsps:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-lfsps
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From                                                  Message
  ----    ------                 ----  ----                                                  -------
  Normal  Scheduled              32s   default-scheduler                                     Successfully assigned front-end to gke-test-cluster-default-pool-d29bc0e9-jz8w
  Normal  SuccessfulMountVolume  31s   kubelet, gke-test-cluster-default-pool-d29bc0e9-jz8w  MountVolume.SetUp succeeded for volume "default-token-lfsps"
  Normal  Pulling                30s   kubelet, gke-test-cluster-default-pool-d29bc0e9-jz8w  pulling image "schoolofdevops/frontend:latest"
  Normal  Pulled                 21s   kubelet, gke-test-cluster-default-pool-d29bc0e9-jz8w  Successfully pulled image "schoolofdevops/frontend:latest"
  Normal  Created                21s   kubelet, gke-test-cluster-default-pool-d29bc0e9-jz8w  Created container
  Normal  Started                21s   kubelet, gke-test-cluster-default-pool-d29bc0e9-jz8w  Started container
```

Commands to operate the pod

```
kubectl exec -it vote ps sh

kubectl exec -it vote  sh

kubectl logs vote

```

## Troubleshooting Tip

If you would like to know whats the current status of the pod, and if its in a error state, find out the cause of the error, following command could be very handy.

```
kubectl get pod vote -o yaml
```

Lets learn by example. Update pod spec and change the image to something that does not exist.

```
kubectl edit pod vote
```

This will open a editor. Go to the line which defines image  and change it to a tag that does not exist

e.g.

```
spec:
  containers:
  - image: schoolofdevops/vote:latst
    imagePullPolicy: Always
```

where tag **latst** does not exist. As soon as you save this file, kubernetes will apply the change.

Now check the status,
```
kubectl get pods  

NAME      READY     STATUS             RESTARTS   AGE
vote      0/1       ImagePullBackOff   0          27m
```

The above output will only show the status, with a vague error. To find the exact error, lets get the stauts of the pod.

Observe the **status** field.  


```
kubectl get pod vote -o yaml
```

Now the status field shows a detailed information, including what the exact error. Observe the following snippet...

```
status:
...
containerStatuses:
....
state:
  waiting:
    message: 'rpc error: code = Unknown desc = Error response from daemon: manifest
      for schoolofdevops/vote:latst not found'
    reason: ErrImagePull
hostIP: 139.59.232.248
```

This will help you to pinpoint to the exact cause and fix it quickly.


Now that you  are done experimenting with pod, delete it with the following command,

```
kubectl delete pod vote

kubectl get pods
```

## Attach a Volume to the Pod

Lets create a pod for database and attach a volume to it. To achieve this we will need to

  * create a **volumes** definition
  * attach volume to container using **VolumeMounts** property

Local host volumes are of two types:  
  * emptyDir  
  * hostPath  

We will pick hostPath. [Refer to this doc to read more about hostPath.](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)


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
    hostPath:
      path: /var/lib/pgdata
      type: DirectoryOrCreate
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

kubectl label nodes <node-name> zone=aaa

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
    zone: 'aaa'
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
      image: nginx:stable-alpine
      ports:
        - containerPort: 80
      volumeMounts:
      - name: data
        mountPath: /var/www/html-sample-app
    - name: sync
      image: schoolofdevops/synch
      volumeMounts:
      - name: data
        mountPath: /var/www/html-sample-app
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
kubectl logs  web  -c sync
kubectl logs  web  -c nginx

kubectl exec -it web  sh  -c nginx
kubectl exec -it web  sh  -c sync

```

## Exercise

Create a pod definition for redis and deploy.

#### Reading List

  * PodSpec: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#pod-v1-core
  * Managing Volumes with Kubernetes: https://kubernetes.io/docs/concepts/storage/volumes/
  * Node Selectors, Affinity: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
