# Exercise 2: Pods

#### 1. Create a Pod manifest which uses influxdb image and open port 8086.

#### 2. Create a Pod with ubuntu:trusty image and a command to echo “YOUR_NAME” which overrides the default CMD/ENTRYPOINT of the image.
`Reference`: [Define command argument in a Pod](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)

#### 3. Apply the following Pod manifest and read the error. Fix it by editing it.
```
              apiVersion: v1apps/beta1
              kind: Pod
              metadata:
                name: mogambo-frontend
                label:
                  role: frontend
              spec:
                containers:
                  - name: frontend
                    image: schoolofdevops/frontend:orange
                    ports:
                      - containerName: web
                        Port: 8079
                        protocol: TCP
```
`Reference`: [Debugging a unscheduled Pod](https://stackoverflow.com/questions/37302776/kubectl-get-pods-kubectl-get-pods-status-imagepullbackoff)

#### 4. A Pod with the following pod always crashes with CrashLoopBackOff error. How would you fix it?

```
              image: schoolofdevops/nginx:break
              ports: 80
```
`Reference`: [Debugging a crashed Pod] (https://sysdig.com/blog/debug-kubernetes-crashloopbackoff/)

#### 5. You are running a Pod with hostPort option. Your Pod status stays “pending”. What could be the issue for the Pod not being scheduled?
`Reference`: [Debugging a unscheduled Pod](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/#my-pod-stays-waiting)

#### 6. The given manifest for multi-container pod is not working as intended. It does not sync the content between containers like expected. What could be the issue? Find the issue just by reading the manifest.

```
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    tier: front
    app: nginx
    role: ui
spec:
  containers:
    - name: nginx
      image: nginx:stable-alpine
      ports:
        - containerPort: 80
          protocol: TCP
      volumeMounts:
        - name: data
          mountPath: /var/www/html-sample-app

    - name: sync
      image: schoolofdevops/sync:v2
      volumeMounts:
        - name: datanew
          mountPath: /var/www/app

  volumes:
    - name: data
      emptyDir: {}
```

#### 7. For the above given manifest, the following command is not working. What could be the issue?
```
kubeclt exec -it web -sh -c synch
```

