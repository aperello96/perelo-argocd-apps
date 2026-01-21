# Crate ceph-csi driver for kubernetes/okd

## Create app on argocd with values.yaml

## Create pool and pool credentials to ceph access
```bash
$ ceph osd pool create k8s-pool 64 64 
$ rbd pool init k8s-pool
```