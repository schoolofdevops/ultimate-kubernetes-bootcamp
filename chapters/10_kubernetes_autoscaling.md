# Kubernetes Horizonntal Pod Autoscaling

With Horizontal Pod Autoscaling, Kubernetes automatically scales the number of pods in a replication controller, deployment or replica set based on observed CPU utilization (or, with alpha support, on some other, application-provided metrics).

The Horizontal Pod Autoscaler is implemented as a Kubernetes API resource and a controller. The resource determines the behavior of the controller. The controller periodically adjusts the number of replicas in a replication controller or deployment to match the observed average CPU utilization to the target specified by user

### Prerequisites

  * Metrics Server (https://github.com/kubernetes-incubator/metrics-server). This replaces heapster from k8s 1.9
  * Resource Requests for Containers in Pod Spec is a must

#### Deploying Metrics Server


```
git clone  https://github.com/kubernetes-incubator/metrics-server.git
kubectl apply -f kubectl create -f metrics-server/deploy/1.8+/
```

Validate
```
kubectl get deploy,pods -n kube-system --selector='k8s-app=metrics-server'
```

[sample output]
```
NAME                                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/metrics-server   1         1         1            1           28m

NAME                                  READY     STATUS    RESTARTS   AGE
pod/metrics-server-6fbfb84cdd-74jww   1/1       Running   0          28m
```

Monitoring has been setup.

### Create a HPA

To demonstrate Horizontal Pod Autoscaler we will use a custom docker image based on the php-apache image

`file: vote-hpa.yaml`


```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: vote
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vote
  minReplicas: 4
  maxReplicas: 12
  targetCPUUtilizationPercentage: 40
```

apply

```
kubectl apply -f vote-hpa.yaml
```

Validate

```
kubectl get hpa

kubectl describe hpa vote

kubectl get pod,deploy


```



###  Load Test (TODO from here on)

Now we can increase the load and trying testing what will happen.
We will start a container, and send an infinite loop of queries to the php-apache service

```
kubectl run -i --tty -n dev load-generator --image=busybox /bin/sh

Hit enter for command prompt

while true; do wget -q -O- http://vote; done

```
Now open a new window of the same machine.

And check the status of the hpa
```
kubectl get hpa
```
Sample Output:

```
kubectl get hpa
NAME         REFERENCE                     TARGET    CURRENT   MINPODS   MAXPODS   AGE
php-apache   Deployment/php-apache/scale   50%       305%      1         10        3m
```

Now if you check the pods it will be automatically scaled to the desired value.

```
kubectl get pods
```

Sample Output
```
kubectl get pods
NAME                              READY     STATUS    RESTARTS   AGE
load-generator-1930141919-1pqn0   1/1       Running   0          1h
php-apache-3815965786-2jmm9       1/1       Running   0          1h
php-apache-3815965786-4f0ck       1/1       Running   0          1h
php-apache-3815965786-73w24       1/1       Running   0          1h
php-apache-3815965786-80n2x       1/1       Running   0          1h
php-apache-3815965786-c6w0k       1/1       Running   0          1h
php-apache-3815965786-f06dg       1/1       Running   0          1h
php-apache-3815965786-nfs8d       1/1       Running   0          1h
php-apache-3815965786-phrhs       1/1       Running   0          1h
php-apache-3815965786-z6rnm       1/1       Running   0          1h
```

### Stop load

In the terminal where we created the container with busybox image, terminate the load generation by typing <Ctrl> + C

Then we will verify the result state (after a minute or so)
```
kubectl get hpa
NAME         REFERENCE                     TARGET    CURRENT   MINPODS   MAXPODS   AGE
php-apache   Deployment/php-apache/scale   50%       0%        1         10        11m

$ kubectl get deployment php-apache
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
php-apache   1         1         1            1           27m
```

##### Reading List

  * [Core Metrics Pipeline]( https://kubernetes.io/docs/tasks/debug-application-cluster/core-metrics-pipeline/)
  * [Metrics Server](https://github.com/kubernetes-incubator/metrics-server)
  * [Assignign Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/#specify-a-cpu-request-and-a-cpu-limit)
  * [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
