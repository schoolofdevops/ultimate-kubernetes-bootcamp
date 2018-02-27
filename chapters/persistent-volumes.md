# Steps to set up nfs based Persistent Volumes

## Set up NFS Server

Install NFS Server package
```
sudo apt-get update
sudo apt-get install nfs-kernel-server -y
```

Create and share NFS directory
```
sudo mkdir -p /var/nfs/kubernetes
sudo chown nobody:nogroup /var/nfs/kubernetes
sudo chmod 666 /etc/exports
sudo cat > /etc/exports <<EOF
/var/nfs/kubernetes    192.168.12.0/24(rw,sync,no_subtree_check)
EOF
sudo chmod 644 /etc/exports
```
Change **<192.168.12.0/24>** with your IP space.

Restart NFS server to use the configuration
```
sudo systemctl restart nfs-kernel-server
```

## Set up NFS Provisioner in kubernetes

Clone the `External Storage` repository from github.
```
git clone https://github.com/kubernetes-incubator/external-storage.git
```

We are going to use nfs-client provisioner.
```
cd external-storage/nfs-client
```

Update the deployment with your NFS server's data
```
vim deploy/deployment.yaml

kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: 192.168.12.10
            - name: NFS_PATH
              value: /var/nfs/kubernetes
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.12.10
            path: /var/nfs/kubernetes
```
Change `NFS_SERVER` and `NFS_PATH` values accordingly. 

**Tip**: You can also change `PROVISIONER_NAME` data to something like `sod.com/nfs`. If you do that be sure to change `provisioner` value in `class.yaml` as well.

Deploy nfs-client provisioner.

```
kubectl apply -f deploy/deployment.yaml
```

Create Storage Class.

```
kubectl apply -f deploy/class.yaml
```

## Authorization

Your cluster might be using RBAC. So you need to create Service Accounts and Cluster Roles to provide appropriate permissions to nfs-provisioner.
The manifests inside `auth` directory has not been updated in a while. So please change `apiVersion` from `rbac.authorization.k8s.io/v1alpha1` to `rbac.authorization.k8s.io/v1` in all the files inside that directory.

Apply RBAC...
```
kubectl create -f deploy/auth/serviceaccount.yaml
kubectl create -f deploy/auth/clusterrole.yaml
kubectl create -f deploy/auth/clusterrolebinding.yaml
kubectl patch deployment nfs-client-provisioner -p '{"spec":{"template":{"spec":{"serviceAccount":"nfs-client-provisioner"}}}}'
```

## Test

Let us test the if our NFS provisioner works.

Create a PVC
```
kubectl apply -f deploy/test-claim.yaml
```

Check the PVC Status
```
kubectl get pvc

[output]
NAME         STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
test-claim   Bound     pvc-17e37502-1b9f-11e8-ab45-02ab3bf0d4f7   1Mi        RWX            managed-nfs-storage   52s
```

Let us create a pod to use this PVC
```
kubectl apply -f deploy/test-pod.yaml
```

Now check the contents of our shared NFS directory
```
ls /var/nfs/kubernetes

[output]
default-test-claim-pvc-17e37502-1b9f-11e8-ab45-02ab3bf0d4f7

ls /var/nfs/kubernetes/default-test-claim-pvc-17e37502-1b9f-11e8-ab45-02ab3bf0d4f7

[output]
SUCCESS
```
There you go. Thats how you create PV and PVC using NFS in Kubernetes
