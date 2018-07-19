# RBAC Reference

### kubernetes Instances Configuration
## GCP

|NUMBER OF NODE-SIZE |INSTANCE TYPE   |CPU|MEMORY   
|--------------------   
| 1-5                | n1-standard-1  | 1 |3.75  |  
|6-10                | n1-standard-2  | 2 |7.50  |  
|11-100              | n1-standard-4  | 4 |15    |
|101-250             | n1-standard-8  | 8 |30    |
|251-500             | n1-standard-16 | 16|60    |
|more than 500       | n1-standard-32 | 32|120   |

## AWS
| NUMBER OF NODE_SIZE   |INSTANCE TYPE   |CPU|MEMORY
|--------------------|-----------------|----|---------
| 1-5                | m3.medium      | 1 | 3.75 |
|6-10                | m3.large       | 2 | 7.50 |
|11-100              | m3.xlarge      | 4 | 15   |
|101-250             | m3.2xlarge     | 8 | 30   |
|251-500             | c4.4xlarge     | 8 | 30   |
|more than 500       | c4.8xlarge     | 16| 60   |

## api groups and resources

| apiGroup     | Resources     |
| :------------- | :------------- |
| apps      |   daemonsets, deployments, deployments/rollback, deployments/scale, replicasets, replicasets/scale, statefulsets, statefulsets/scale     |
|core|configmaps, endpoints, persistentvolumeclaims, replicationcontrollers, replicationcontrollers/scale, secrets, serviceaccounts, services,services/proxy, namespace, volume, resourceQuota
|autoscaling|horizontalpodautoscalers
| batch|cronjobs, jobs
|policy| poddisruptionbudgets
|networking.k8s.io|networkpolicies
|authorization.k8s.io|localsubjectaccessreviews
|rbac.authorization.k8s.io|rolebindings,roles, clusterRole, clusterRoleBinding
|storage.k8s.io|volumeAttachment, stroageClass
|extensions|podSecurityPolicy, 	Ingress

## verbs
||
|---
|GET| get
|MODIFY| update, patch
|READ| watch, list, get
|DELETE| delete, deletecollection
|CREATE| create
