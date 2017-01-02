## Replication Controller Operations

A replication controller ensures that a specified number of pod “replicas” are running at any one time. If there are too many, it will kill some. If there are too few, it will start more.

#### Creating a replication controller

Follow the below command to create a RC:

```
touch replication.yaml
vi replication.yaml
```

Paste the below code in it:
```
{
  "kind": "ReplicationController",
  "apiVersion": "v1",
  "metadata": {
    "name": "frontend-controller",
    "labels": {
      "state": "serving"
    }
  },
  "spec": {
    "replicas": 2,
    "selector": {
      "app": "frontend"
    },
    "template": {
      "metadata": {
        "labels": {
          "app": "frontend"
        }
      },
      "spec": {
        "volumes": null,
        "containers": [
          {
            "name": "php-redis",
            "image": "redis",
            "ports": [
              {
                "containerPort": 80,
                "protocol": "TCP"
              }
            ],
            "imagePullPolicy": "IfNotPresent"
          }
        ],
        "restartPolicy": "Always",
        "dnsPolicy": "ClusterFirst"
      }
    }
  }
}
```

```
kubectl create -f replication.yaml
```

To list replication controllers on a cluster

```
kubectl get rc
```

To view detailed information about a specific replication controller

```
kubectl describe rc frontend-controller
```



#### Deleting the RC:

To delete a replication controller as well as the pods that it controls, use

```
kubectl delete rc NAME
```
