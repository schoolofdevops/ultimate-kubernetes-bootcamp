#### 1. From the following manifest, change the service type to Load Balancer. Try to access the service by accessing the load balancer URL. (This excercise assumes you already have vote application deployed in your environment.)
```
apiVersion: v1
kind: Service
metadata:
  name: vote
  namespace: instavote
  labels:
    role: vote
spec:
  selector:
    role: vote
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30100
  type: NodePort
```

#### 2. How would you access your service on Port 80 from internet for the following?
```
apiVersion: v1
kind: Service
metadata:
  name: vote
  namespace: instavote
  labels:
    role: vote
spec:
  selector:
    role: vote
  ports:
    - port: 80
      targetPort: 80
```

#### 3. The following service is created in `instavote` namespace. Use kubectl command to change its namespace to `default`.
```
apiVersion: v1
kind: Service
metadata:
  name: vote
  namespace: instavote
  labels:
    role: vote
spec:
  selector:
    role: vote
  ports:
    - port: 80
      targetPort: 80
```

#### 4. 
