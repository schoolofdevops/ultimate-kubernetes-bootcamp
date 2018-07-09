# Exercise 2: Volumes
#### 1. Write a Pod manifest with the image nginx which has a volume that mounts /etc/nginx/ directory. Use "hostPath" volume type.
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    volumeMounts:
      xxx
```
`Reference`: [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

#### 2. 
