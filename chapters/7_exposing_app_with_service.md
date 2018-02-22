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
NAME                         READY     STATUS    RESTARTS   AGE
front-end-64597c55f8-dk6g7   1/1       Running   0          4m
front-end-64597c55f8-mkp2h   1/1       Running   0          4m
front-end-64597c55f8-r2c54   1/1       Running   0          4m
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
  name: front-end
  namespace: mogambo
spec:
  selector:
    app: front-end
  ports:
  - port: 8079
    protocol: TCP
    targetPort: 8079
  type: NodePort
```

Save the file.

Now to create a service:

```
kubectl apply -f frontend-svc.yaml
kubectl get svc
```

Now to check which port the pod is connected
```
kubectl describe service front-end
```
Check for the Nodeport here

Sample Output
```
Name:                     front-end
Namespace:                mogambo
Labels:                   role=svc
                          tier=front
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"role":"svc","tier":"front"},"name":"front-end","namespace":"mogambo"},"spec...
Selector:                 app=front-end
Type:                     NodePort
IP:                       10.11.248.77
Port:                     <unset>  8079/TCP
TargetPort:               8079/TCP
NodePort:                 <unset>  30197/TCP
Endpoints:                10.8.1.13:8079,10.8.1.14:8079,10.8.1.15:8079 + 3 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Go to browser and check hostip:NodePort

Here the node port is 30197.

Sample output will be:

![Vote](images/vote.png)

## Exposing the app with ExternalIP

`file: /k8s-code/projects/mogambo/dev/frontend-svc.yml`

```
spec:
  selector:
    app: front-end
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
kubectl apply -f frontend-svc.yml
kubectl  get svc
kubectl describe svc front-end
```
