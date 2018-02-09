# Services

Services are a way to expose your backend pods to outside world. Services can also be used for internal networks.

## Picking up the right type of a service

We have four type of services to choose from. Each has a unique use case for which it can be used. Let us see about the types of services below.

### Types:

  * ClusterIp
    * Internal network.
    * Can not accessed from outside world.
    * Best suited for backend services which should not be exposed (ex: DB).
  * NodePort
    * Can be accessed from outside world.
    * NodePort is best suited for development environment(debugging).
    * Not suited for production use.
  * LoadBalancer
    * Normal cloud load balancer.
    * Exposes service to outside world.
  * ExternalName
    * Used for entities which are outside the cluster.
    * CNAME record for an external service.
    * No ports are defined.

We have one more service configuration, which is not exactly a service type. That is .,
  * ExternalIP

## Cluster IP and DNS

ClusterIP is internal to the cluster. ClusterIP is provided by default. If you do not define the type of the service, then the service is configured with a clusterIP. A typical service type of clusterIP looks like,

`File: frontend-svc.yml`

```
apiVersion: v1
kind: Service
metadata:
  name: front-end
  namespace: instavote
  labels:
    app: front-end
    env: dev
spec:
  selector:
    app: front-end
  ports:
    - protocol: TCP
      port: 8079
  type: ClusterIP
```

## NodePort

Exposes a port on node within 30000 to 32767 range. If a service is defined as nodePort, we can access the service on <ANY-NODE-IP:NODEPORT>. Let us change the frontend service type to NodePort and see what happens.

`File: frontend-svc.yml`

```
apiVersion: v1
kind: Service
metadata:
  name: front-end
  namespace: instavote
  labels:
    app: front-end
    env: dev
spec:
  selector:
    app: front-end
  ports:
    - protocol: TCP
      port: 8079
  type: NodePort
```

Lets get the node port created for the front-end service.

```
kubectl get svc front-end


```

Visit this node port on any of node's IP. In my case,


## ExternalIP

ExternalIP is not exactly a service type. It can be coupled with any other service type. For example a service with NodePort can be used with ExternalIP as well. Only cache here is, the external IP has to one of the node/master's IP. It is used to route the traffic to one or more cluster nodes.

`File: frontend-svc.yml`

```
apiVersion: v1
kind: Service
metadata:
  name: front-end
  namespace: instavote
  labels:
    app: front-end
    env: dev
spec:
  selector:
    app: front-end
  ports:
    - protocol: TCP
      port: 8079
  type: NodePort
  externalIPs:
  - 10.40.1.26
  - 10.40.1.11
```

## LoadBalancer

Service type LoadBalancer is only supported on AWS,GCE,Azure and Openstack. If you have you cluster running on any of these clouds, give it a try. It will create a load balancer for us with a ephemeral IP. We can also specify a loadBalancerIP. Mostly not recommended to use. Using service type LoadBalancer will rise your cloud provider spendings. Think about launching 10 load balancers for 10 different services. Thats where **ingress** come into picture (explained in the later part).

```
apiVersion: v1
kind: Service
metadata:
  name: front-end
  namespace: instavote
  labels:
    app: front-end
    env: dev
spec:
  selector:
    app: front-end
  ports:
    - protocol: TCP
      port: 8079
  type: LoadBalancer
```

## Headless services with Endpoints/External Names

## Labels and Selectors

## Service API specs

## Traffic Policy

## Selectors

## Ingress

## Services Vs Ingress

## Creating service resources for applications and backing services
