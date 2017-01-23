## Creating a Deployment

A Deployment simply ensures that a specified number of pod “replicas” are running at any one time. If there are too many, it will kill some. If there are too few, it will start more.

Create a yaml file for the deployment.
```
touch voting-app-deployment.yaml
```
Now paste the below line in the yaml file
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    run: voting-appp
  name: voting-appp
  namespace: default
spec:
  replicas: 1
  template:
    metadata:
      labels:
        run: voting-appp
    spec:
      containers:
      - image: venkatsudharsanam/votingapp-python:8.0.0
        imagePullPolicy: Always
        name: voting-appp
        ports:
        - containerPort: 80
          protocol: TCP
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      securityContext: {}
```

And save the file.

Now create the Deployment
```
kubectl create -f voting-app-deployment.yaml
```

Now the deployment is created. To check it,

```
kubectl get deployment
```
Sample Output
```
kubectl get deployment
NAME          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
voting-appp   1         1         1            1           19s
```

##Creating a Service

Service is necessary since there should be a network configuration for the each pods.

Create a yaml file for the service.
```
touch voting-app-service.yaml
```
Now paste the below line in the yaml file
```
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
And save the file.

Now create the Service
```
kubectl create -f voting-app-service.yaml
```

Now the deployment is created. To check it,

```
kubectl get service
```
Sample Output
```
kubectl get service
NAME          CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes    10.96.0.1        <none>        443/TCP        2d
voting-appp   10.109.118.224   <pending>     80:30410/TCP   11s
```

Now we can see that the port 30410 has been assigned to the pod.

Please check the browser with the hostip and the port number.

In this case it hostip:30410

![alt text](images/Nginx.PNG "Nginx Page")

##Scaling a deployment
To scale a deployment in Kubernetes is done by:

```
kubectl scale deployment/voting-appp --replicas=5
```

Sample output:
```
kubectl scale deployment/voting-appp --replicas=5
deployment "voting-appp" scaled
```

To check, we can see the deployment what has happened:
```
kubectl get deployment
NAME          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
voting-appp   5         5         5            1           28m
```
##Deleting a deployment

To delete a deployment in Kubernetes is very simple,

```
kubectl delete deployment voting-appp
```

To verify:

```
kubectl get deployments
```

##Deleting a service

To delete a service in Kubernetes is very simple,

```
kubectl delete service voting-appp
```
To verify:

```
kubectl get service
```
