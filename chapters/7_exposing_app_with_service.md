# Exposing Application with  a Service

Types of Services:   

  * ClusterIP
  * NodePort
  * LoadBalancer
  * ExternalName


![kubernetes service](https://github.com/schoolofdevops/ultimate-kubernetes-bootcamp/blob/master/images/k8s_service.jpg?raw=true)


```
kubectl get pods
kubectl get svc
```

Sample Output:
```
NAME                READY     STATUS    RESTARTS   AGE
voting-appp-1j52x   1/1       Running   0          12m
voting-appp-pr2xz   1/1       Running   0          9m
voting-appp-qpxbm   1/1       Running   0          15m
```

## Publishing a service with NodePort

Filename: vote-svc.yaml

```
---
apiVersion: v1
kind: Service
metadata:
  labels:
    role: svc
    tier: front
  name: vote-svc
  namespace: instavote
spec:
  selector:
    app: vote
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  type: NodePort
```

Save the file.

Now to create a service:

```
kubectl apply -f vote-svc.yaml
kubectl get svc
```

Now to check which port the pod is connected
```
kubectl describe service vote
```
Check for the Nodeport here

Sample Output
```
Name:                     vote
Namespace:                instavote
Labels:                   role=svc
                          tier=front
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"role":"svc","tier":"front"},"name":"vote","namespace":"instavote"},"spec":{...
Selector:                 app=vote
Type:                     NodePort
IP:                       10.108.108.157
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31429/TCP
Endpoints:                10.38.0.4:80,10.38.0.5:80,10.38.0.6:80 + 2 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Go to browser and check hostip:NodePort

Here the node port is 31429.

Sample output will be:

![Vote](images/vote.png)

## Exposing the app with ExternalIP

```
spec:
  selector:
    app: vote
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  type: NodePort
  externalIPs:
    - 192.168.12.11
    - 192.168.12.12
```

apply
```
kubectl apply -f vote-svc.yaml
kubectl  get svc
kubectl describe svc vote
```
