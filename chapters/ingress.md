# Ingress


## Pre Requisites

  * Ingress controller such as Nginx, Trafeik needs to be deployed before creating ingress resources.
  * On GCE, ingress controller runs on the master. On all other installations, it needs to be deployed, either as a deployment, or a daemonset. In addition, a service needs to be created for ingress.
  * Daemonset will run ingress on each node. Deployment will just create a highly available setup, which can then be exposed on specific nodes using ExternalIPs configuration in the service.

##### Reading

  * [Trafeik's Guide to Kubernetes Ingress Controller](https://docs.traefik.io/user-guide/kubernetes/)
  * [Annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)
  * [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

**References**

  * [Online md5 generator](http://www.md5.cz/) 

**Keywords**
  * trafeik on kubernetes
  * kubernetes ingress
  * kubernetes annotations
  * daemonsets
