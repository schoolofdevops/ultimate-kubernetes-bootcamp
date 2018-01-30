# Advanced Pod Scheduling

In the Kubernetes bootcamp training, we have seen how to create a pod and and some basic pod configurations to go with it. But this chapter explains some advanced topics related to pod scheduling.

## Adding health checks

Health checks in Kubernetes work the same way as traditional health checks of applications. They make sure that our application is ready to receive and process user requests. In Kubernetes we have two types of health checks,
  * Liveness Probe
  * Readiness Probe
Probes are simply a *diagnostic action* performed by the kubelet. There are three types actions a kubelet perfomes on a pod, which are namely,
  * `ExecAction`: Executes a command inside the pod. Assumed successful when the command **returns 0** as exit code.
  * `TCPSocketAction`: Checks for a state of a particular port on the pod. Considered successful when the state of **the port is open**.
  * `HTTPGetAction`: Performs a GET request on pod's IP. Assumed successful when the status code is **greater than 200 and less than 400**

In cases of any failure during the diagnostic action, kubelet will report back to the API server. Let us study about how these health checks work in practice.

### Liveness Probe
Liveness probe checks the status of the pod(whether it is running or not). If livenessProbe fails, then the pod is subjected to its restart policy. The default state of livenessProbe is *Success*.

### Readiness Probe
Readiness probe checks whether your application is ready to serve the requests. When the readiness probe fails, the pod's IP is removed from the end point list of the service. The default state of readinessProbe is *Success*.

## Resource requests and limits

## Why and how to define pod resource limits

## Defining node/pod affinity and anti-affinity

## Taints and tolerations

## Vertical pod scaling

## Pod termination process
