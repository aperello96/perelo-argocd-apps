# Crate ceph-csi driver for kubernetes/okd

## Create app on argocd with values.yaml
Check the values.yml. You have to update the cluserId and mons

## Create pool and pool credentials to ceph access
```bash
$ ceph osd pool create k8s-pool 64 64 
$ rbd pool init k8s-pool
$ ceph auth get-or-create-key client.k8s-user mds 'allow *' mgr 'allow *' mon 'allow *' osd 'allow * pool=k8s-pool' | tr -d '\n' | base64
```

encode user in base64:
```bash
$ echo "k8s-user" | tr -d '\n' | base64
```

## Create secrets resource:
Create resource ceph-admin-secret.yaml with userID and userKey in base64 from above

```bash
apiVersion: v1
kind: Secret
metadata:
  name: ceph-admin
  namespace: ceph-csi
type: kubernetes.io/rbd
data:
  userID: azhzLXVzZXI=
  userKey: QVFBV1AzRnBWVXhhQ0JBQUpZUGQ1bVJvbXhwZEM5OEpyK3XXXXXXXX== 
```
apply with
```bash
kubectl apply -f ceph-admin-secret.yaml
```

## Create storageclass:
Create resource ceph-rbd-sc.yaml

```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ceph-rbd-sc
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rbd.csi.ceph.com
parameters:
   clusterID: d756e630-5310-48f8-b5ba-3d190d661622
   pool: k8s-pool
   imageFeatures: layering
   csi.storage.k8s.io/provisioner-secret-name: ceph-admin
   csi.storage.k8s.io/provisioner-secret-namespace: ceph-csi
   csi.storage.k8s.io/controller-expand-secret-name: ceph-admin
   csi.storage.k8s.io/controller-expand-secret-namespace: ceph-csi
   csi.storage.k8s.io/node-stage-secret-name: ceph-admin
   csi.storage.k8s.io/node-stage-secret-namespace: ceph-csi
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
   - discard
```
apply with
```bash
kubectl apply -f ceph-rbd-sc.yaml
```

## Test provisioning:
Create a resource create-ceph-pvc.yaml

```bash
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-ceph-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: ceph-rbd-sc 
```
apply with
```bash
kubectl apply -f create-ceph-pvc.yaml
```
Check if the resource is ok:
```bash
kubectl get pvc
```

You can find more information (here:)[https://medium.com/@satishdotpatel/ceph-integration-with-kubernetes-using-ceph-csi-c434b41abd9c]