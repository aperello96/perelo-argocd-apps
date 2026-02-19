aplicamos:

oc apply -f crds.yaml
oc apply -f common.yaml
oc apply -f operator.yaml

oc adm policy add-scc-to-user privileged system:serviceaccount:rook-ceph:rook-ceph-system
oc adm policy add-scc-to-user privileged system:serviceaccount:rook-ceph:rook-csi-rbd-plugin-sa
oc adm policy add-scc-to-user privileged system:serviceaccount:rook-ceph:rook-csi-rbd-provisioner-sa
oc adm policy add-scc-to-user privileged system:serviceaccount:rook-ceph:rook-csi-cephfs-plugin-sa
oc adm policy add-scc-to-user privileged system:serviceaccount:rook-ceph:rook-csi-cephfs-provisioner-sa

Creamos el secreto (editamos el fichero segun necesitamos)
external-cluster-secret.yaml
aplicamos: oc apply -f external-cluster-secret.yaml

Creamos el cephcluster externo:
Editamos el fichero: external-cluster.yaml
aplicamos: oc apply -f external-cluster.yaml

Creamos la storageclass: storageclass.yaml
oc apply -f storageclass.yaml

Hay que editar el configmap manualmente con:
oc -n rook-ceph edit configmap rook-ceph-csi-config
y cambiamos "clusterID":"rook-ceph" => "clusterID":"rook-ceph-external"

Check storageclass:
oc -n rook-ceph get cephcluster
oc get storageclass
oc create -f test-pvc.yaml #check if pvc on bound: oc get pvc test-rook-ceph

Convertir a sc por defecto:
oc patch storageclass rook-ceph-block -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'