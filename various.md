Quick exec into one of the portworx pods:
```
# Get the namespace where portworx is installed.
export pxnamespace=$(kubectl get stc -A | grep -v NAMESPACE | awk '{print $1}')
PX_POD=$(kubectl get pods -l name=portworx -n $pxnamespace -o jsonpath='{.items[0].metadata.name}')
kubectl -n portworx exec -ti $PX_POD -- bash
```

Various others:
```
# For older system with in-tree storage classes, copy them to an identical storageclass using our CSI volume provider.
# requires yq

for sc in $(kubectl get sc | grep kubernetes.io/portworx-volume | awk '{print $1}')
    do kubectl get sc ${sc} -o yaml | yq e 'del(.metadata.resourceVersion)' - |yq e 'del(.metadata.creationTimestamp)' - | yq e 'del(.metadata.uid)' - | sed 's/kubernetes.io\/portworx-volume/pxd.portworx.com/g'| sed "s/${sc}/${sc}-csi/g" > ${sc}-csi.yaml
    kubectl create -f ${sc}-csi.yaml
done
```