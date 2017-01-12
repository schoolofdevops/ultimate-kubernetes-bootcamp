# Rolling Updates and Zero Downtime Deployments

A rolling update works by:
1. Creating a new replication controller with the updated configuration.
2. Increasing/decreasing the replica count on the new and old controllers until the correct number of replicas is reachedor by updating the version of the pod which is running.

#### Creating a Service for RC
We already have the RC

```
kubectl get rc
```
Sample Output:
```
kubectl get rc
NAME          DESIRED   CURRENT   READY     AGE
voting-appp   1         1         1         1m
```

We need to create a service for this RC.
```
touch rc-service
vi rc-service
```
Paste the below content:
```
---
apiVersion: v1
kind: Service
metadata:
  labels:
    run: voting-appp
  name: voting-appp
  namespace: default
spec:
  selector:
    run: voting-appp
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  type: LoadBalancer
```

Save the file.

Now to create a service:

```
kubectl create -f rc-service
```
Sample Output:
```
kubectl create -f voting-app-service.yaml
service "voting-appp" created
```

Now to check which port the pod is connected
```
kubectl describe service voting-appp
```
Check for the Nodeport here

Sample Output
```
kubectl describe service voting-appp
Name:                   voting-appp
Namespace:              default
Labels:                 run=voting-appp
Selector:               run=voting-appp
Type:                   LoadBalancer
IP:                     10.101.201.57
Port:                   <unset> 80/TCP
NodePort:               <unset> 30165/TCP
Endpoints:              10.40.0.15:80
Session Affinity:       None
No events.
```

Go to browser and check hostip:NodePort

Here the node port is 30165.

Sample output will be:

![alt text](images/RC-Nginx.png "Nginx Page")

#### Rolling update

Now we can see in the RC yaml file that the pods are running with the version of image : venkatsudharsanam/votingapp-python:8.0.0.

Lets try upgrading our pods to the version venkatsudharsanam/votingapp-python:10.0.0 without deleting the RC or the pods.

That's how the rolling updates helps in Kubernetes.

To do so:
```
kubectl rolling-update voting-appp --image=venkatsudharsanam/votingapp-python:10.0.0
```
Sample Output:
```
kubectl rolling-update voting-appp --image=venkatsudharsanam/votingapp-python:10.0.0
Created voting-appp-c7e373b92c0b11bd7c4bb00463658cd9
Scaling up voting-appp-c7e373b92c0b11bd7c4bb00463658cd9 from 0 to 1, scaling down voting-appp from 1 to 0 (keep 1 pods available, don't exceed 2 pods)
Scaling voting-appp-c7e373b92c0b11bd7c4bb00463658cd9 up to 1
Scaling voting-appp down to 0
Update succeeded. Deleting old controller: voting-appp
Renaming voting-appp to voting-appp-c7e373b92c0b11bd7c4bb00463658cd9
replicationcontroller "voting-appp" rolling updated
```

Now go to the browser and reload the page, you will see the below output:


![alt text](images/RC-Vote.png "Vote Page")

Thus the Rolling Update is successfull.
#### Rollback
If we occurd any error when updating we can rollback by:
```
kubectl rolling-update voting-appp --rollback
```

#### Deleting the RC:

To delete a replication controller as well as the pods that it controls, use

```
kubectl delete rc voting-appp
```
