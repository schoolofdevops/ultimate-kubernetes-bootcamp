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

Each entity created with kubernetes is a resource including pod, service, pods, replication controller etc. Resources can be defined as YAML or JSON.  Here is the syntax to create a YAML specification.

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
      image: schoolofdevops/frontend:latest
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
kubectl exec -it front-end ps sh

kubectl exec -it front-end sh

kubectl logs front-end

```

## Troubleshooting Tip

If you would like to know whats the current status of the pod, and if its in a error state, find out the cause of the error, following command could be very handy.

```
kubectl get pod front-end -o yaml
```

Lets learn by example. Update pod spec and change the image to something that does not exist.

```
kubectl edit pod front-end
```

This will open a editor. Go to the line which defines image  and change it to a tag that does not exist

e.g.

```
spec:
  containers:
  - image: schoolofdevops/frontend:latst
    imagePullPolicy: Always
```

where tag **latst** does not exist. As soon as you save this file, kubernetes will apply the change.

Now check the status,
```
kubectl get pods  

NAME      READY     STATUS             RESTARTS   AGE
front-end      0/1       ImagePullBackOff   0          27m
```

The above output will only show the status, with a vague error. To find the exact error, lets get the stauts of the pod.

Observe the **status** field.  


```
kubectl get pod front-end -o yaml
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
      for schoolofdevops/frontend:latst not found'
    reason: ErrImagePull
hostIP: 139.59.232.248
```

This will help you to pinpoint to the exact cause and fix it quickly.


Now that you  are done experimenting with pod, delete it with the following command,

```
kubectl delete pod front-end

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


File: /k8s-code/pods/db-pod.yml

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
kubectl apply -f db-pod.yml

kubectl describe pod db

kubectl get events
```


## Selecting A Node to run on

```
kubectl get nodes --show-labels

kubectl label nodes <node-name> zone=aaa

kubectl get nodes --show-labels

```

Update pod definition with nodeSelector

`file: k8s-code/pods/frontend-pod.yml`

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
      image: schoolofdevops/frontend:latest
      ports:
        - containerPort: 8079
  nodeSelector:
    zone: 'aaa'
```

For this change, pod needs to be re created.

```
kubectl apply -f frontend-pod.yml
```

## Liveness Probe
Liveness probe checks the status of the pod(whether it is running or not). If livenessProbe fails, then the pod is subjected to its restart policy. The default state of livenessProbe is *Success*.

Let us add liveness probe to our *frontend* pod. The following probe will check whether it is able to *access the port or not*.

`File:  k8s-code/pods/frontend-pod.yml`

```
[...]
      containers:
        - name: front-end
          image: schoolofdevops/frontend
          imagePullPolicy: Always
          ports:
            - containerPort: 8079
          livenessProbe:
            tcpSocket:
              port: 8079
            initialDelaySeconds: 5
            periodSeconds: 5
```

`Expected output:`

```
kubectl apply -f front-end/frontend-pod.yml
kubectl get pods
kubectl describe pod front-end-757db58546-fkgdw

[...]
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              22s   default-scheduler  Successfully assigned front-end-757db58546-fkgdw to node4
  Normal  SuccessfulMountVolume  22s   kubelet, node4     MountVolume.SetUp succeeded for volume "default-token-w4279"
  Normal  Pulling                20s   kubelet, node4     pulling image "schoolofdevops/frontend"
  Normal  Pulled                 17s   kubelet, node4     Successfully pulled image "schoolofdevops/frontend"
  Normal  Created                17s   kubelet, node4     Created container
  Normal  Started                17s   kubelet, node4     Started container
```

Let us change the livenessProbe check port to 8080.

```
          livenessProbe:
            tcpSocket:
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
```

Apply this pod file and check the description of the pod

`Expected output:`
```
kubectl apply -f frontend-pod.yml
kubectl get pods
kubectl describe pod front-end-bf86ffd8b-bjb7p

[...]
Events:
  Type     Reason                 Age               From               Message
  ----     ------                 ----              ----               -------
  Normal   Scheduled              1m                default-scheduler  Successfully assigned front-end-bf86ffd8b-bjb7p to node3
  Normal   SuccessfulMountVolume  1m                kubelet, node3     MountVolume.SetUp succeeded for volume "default-token-w4279"
  Normal   Pulling                38s (x2 over 1m)  kubelet, node3     pulling image "schoolofdevops/frontend"
  Normal   Killing                38s               kubelet, node3     Killing container with id docker://front-end:Container failed liveness probe.. Container will be killed and recreated.
  Normal   Pulled                 35s (x2 over 1m)  kubelet, node3     Successfully pulled image "schoolofdevops/frontend"
  Normal   Created                35s (x2 over 1m)  kubelet, node3     Created container
  Normal   Started                35s (x2 over 1m)  kubelet, node3     Started container
  Warning  Unhealthy              27s (x5 over 1m)  kubelet, node3     Liveness probe failed: Get http://10.233.71.50:8080/: dial tcp 10.233.71.50:8080: getsockopt: connection refused
```

## Readiness Probe
Readiness probe checks whether your application is ready to serve the requests. When the readiness probe fails, the pod's IP is removed from the end point list of the service. The default state of readinessProbe is *Success*.

Readiness probe is configured just like liveness probe. But this time we will use *httpGet request*.

`File:  k8s-code/pods/frontend-pod.yml`

```
[...]
      containers:
        - name: front-end
          image: schoolofdevops/frontend
          imagePullPolicy: Always
          ports:
            - containerPort: 8079
          [...]
          readinessProbe:
            httpGet:
              path: /
              port: 8079
            initialDelaySeconds: 5
            periodSeconds: 3
```

`Expected output:`

```
kubectl apply -f front-end/frontend-pod.yml
kubectl get pods
kubectl describe pod front-end-c5bc89b57-g42nc

[...]
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              11s   default-scheduler  Successfully assigned front-end-c5bc89b57-g42nc to node4
  Normal  SuccessfulMountVolume  10s   kubelet, node4     MountVolume.SetUp succeeded for volume "default-token-w4279"
  Normal  Pulling                8s    kubelet, node4     pulling image "schoolofdevops/frontend"
  Normal  Pulled                 6s    kubelet, node4     Successfully pulled image "schoolofdevops/frontend"
  Normal  Created                5s    kubelet, node4     Created container
  Normal  Started                5s    kubelet, node4     Started container
```

**Task**: Change the readinessProbe port to 8080 and check what happens to the pod.

## Resource requests and limits

We can control the amount of resource requested and used by all the pods. This can be done by adding following data the pod template.

### Resource Request

`File:  k8s-code/pods/frontend-pod.yml`

```
[...]
      containers:
        - name: front-end
          image: schoolofdevops/frontend
          imagePullPolicy: Always
          ports:
            - containerPort: 8079
          [...]
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
```

This ensures that pod always get the minimum cpu and memory specified. But this does not restrict the pod from accessing additional resources if needed. Thats why we have to use **resource limit** to limit the resource usage by a pod.

`Expected output:`

```
kubectl describe pod front-end-5c64b7c5cc-cwgr5

[...]
Containers:
  front-end:
    Container ID:
    Image:          schoolofdevops/frontend
    Image ID:
    Port:           8079/TCP
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Requests:
      cpu:        250m
      memory:     128Mi
```

### Resource limit

`File: code/frontend-pod.yml`

```
[...]
      containers:
        - name: front-end
          image: schoolofdevops/frontend
          imagePullPolicy: Always
          ports:
          [...]
            limits:
              memory: "256Mi"
              cpu: "500m"
```

`Expected output:`

```
kubectl describe pod front-end-5b877b4dff-5twdd

[...]
Containers:
  front-end:
    Container ID:   docker://d49a08c18fd9651af2f3dd28772da977b238a4010f14372e72e0ca24dcec8554
    Image:          schoolofdevops/frontend
    Image ID:       docker-pullable://schoolofdevops/frontend@sha256:94b7a0843f99223a8a1d284fdeeb3fd5a731c03aea57a52751c6ebde40be1f50
    Port:           8079/TCP
    State:          Running
      Started:      Thu, 08 Feb 2018 17:14:54 +0530
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        250m
      memory:     128Mi
```

## Defining node/pod affinity and anti-affinity

We have discussed about scheduling a pod on a particular node using **NodeSelector**, but using node selector is a hard condition. If the condition is not met, the pod cannot be scheduled. Node/Pod affinity and anti-affinity solves this issue by introducing soft and hard conditions.

## Node Affinity

### Soft Node Affinity

Let us label our node1 to **fruit=apple**.

```
kubectl label node node1 fruit=apple

```

Let us modify our pod file to take advantage of this newly labeled node.

`File: k8s-code/pods/frontend-pod.yml`

```
apiVersion: v1
kind: Pod
metadata:
  name: front-end
  namespace: mogambo
  labels:
    app: front-end
    env: dev
spec:
  template:
    metadata:
      labels:
        app: front-end
        env: dev
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: fruit
                operator: In
                values:
                - apple
      containers:
        [...]
```

This is a soft affinity. If there are no nodes labeled as apple, the pod would have still be scheduled.

**Task:** Change the label value to some other fruit name and check the scheduling.

### Hard Node Affinity

Let us test the **hard node affinity**. This condition must be met for the pod to be scheduled. Change the pod file as given below.

`File: k8s-code/pods/frontend-pod.yml`

```
apiVersion: v1
kind: Pod
metadata:
  name: front-end
  namespace: mogambo
  labels:
    app: front-end
    env: dev
spec:
  template:
    metadata:
      labels:
        app: front-end
        env: dev
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: fruit
                operator: In
                values:
                - orange
      containers:
      [...]
```

`Expected output:`

```
kubectl describe pod front-end-d7b787cdf-hhvfr

[...]
Events:
  Type     Reason            Age               From               Message
  ----     ------            ----              ----               -------
  Warning  FailedScheduling  32s (x8 over 1m)  default-scheduler  0/4 nodes are available: 4 MatchNodeSelector.
```

## Node Anti-Affinity

Node anti-affinity can be achieved by using **NotIn** operator. This will help us to ignore nodes while scheduling.

`File:  k8s-code/pods/frontend-pod.yml`

```
apiVersion: v1
kind: Pod
metadata:
  name: front-end
  namespace: mogambo
  labels:
    app: front-end
    env: dev
spec:
  template:
    metadata:
      labels:
        app: front-end
        env: dev
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: fruit
                operator: NotIn
                values:
                - orange
      containers:
      [...]
```

This will schedule the pod on nodes other than node1.

## Pod Affinity

### Hard Pod Affinity

Node affinity allows you to schedule pods on selective nodes. But what if you want to run pods along with other pods selectively. Pod affinity helps us with that.

`File:  k8s-code/pods/frontend-pod.yml`

```
[..]
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - catalogue
              topologyKey: kubernetes.io/hostname
```

`Expected output:`

```
kubectl describe pod front-end-85cc68d56b-4djtt

[...]
Events:
  Type     Reason            Age               From               Message
  ----     ------            ----              ----               -------
  Warning  FailedScheduling  7s (x5 over 14s)  default-scheduler  0/4 nodes are available: 4 MatchInterPodAffinity, 4 PodAffinityRulesNotMatch
```

This is a hard pod affinity. If none of the node has a pod running with label **app=catalogue**, the pod will not be scheduled at all. Here **topologyKey** is a label of a node. This can be any node label.

### Pod Anti-Affinity

Pod anti-affinity works the opposite way of pod affinity.

Lets create **frontend** pod this time.

```
kubectl apply
kubectl apply -f catalogue-pod.yml
```

catalogue pod has a label called **app=catalogue**. Check on which node, catalogue pod is running.

```
kubectl get pods -o wide

NAME                         READY     STATUS    RESTARTS   AGE       IP             NODE
catalogue-86647dcb5b-f6t2j   1/1       Running   0          9s        10.233.71.52   node3
```

Change the front-end pod file as follows.

`File: k8s-code/pods/frontend-pod.yml`

```
[...]
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - catalogue
              topologyKey: kubernetes.io/hostname
```

Now apply the pod.

```
kubectl apply -f frontend-pod.yml
kubectl get pods -o wide

NAME                         READY     STATUS    RESTARTS   AGE       IP               NODE
catalogue-86647dcb5b-f6t2j   1/1       Running   0          2m        10.233.71.52     node3
front-end-5cbc986f44-mjr5m   1/1       Running   0          1m        10.233.102.134   node1
```

## Taints and tolerations

Taint the node.

```
kubectl taint node node4 dedicate=frontend:NoExecute
```

Apply toleration in the pod manifest.

`File:  k8s-code/pods/frontend-pod.yml`

```
apiVersion: v1
kind: pod
metadata:
  name: front-end
  namespace: mogambo
  labels:
    app: front-end
    env: dev
spec:
  template:
    metadata:
      labels:
        app: front-end
        env: dev
    spec:
      tolerations:
        - key: "dedicate"
          operator: "Equal"
          value: "frontend"
          effect: "NoExecute"
      containers:
        - name: frontend
          image: schoolofdevops/frontend
          imagePullPolicy: Always
          ports:
            - containerPort: 8079
```

Check the pod list. This frontend pod will only be scheduled in node4. If there were any pod running in node4, before it got tainted, will be evicted to other hosts.

## Creating Multi Container Pods

`file: k8s-code/pods/multi_container_pod.yml`

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
