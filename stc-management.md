Remove a storagecontroller completely. This is destructive!
```
# REMOVE STC
# Get the namespace where portworx is installed.
export pxnamespace=$()

# Get the stc resource, and patch the deleteStrategy.
export STC=`kubectl -n $pxnamespace get stc | grep -v NAME | cut -f1 -d' '`
kubectl -n $pxnamespace patch stc ${STC} --type=merge --patch '{"spec":{"deleteStrategy":{"type":"UninstallAndWipe"}}}'

# Delete configmaps out of kube-system after uninstall-wipe. 
# Ignoring kdmp-config because I'm not sure if there could be other systems using it.
for cm in $(kubectl -n kube-system get cm | grep px | awk '{print $1}'); do kubectl -n kube-system delete cm $cm;done
for cm in $(kubectl -n kube-system get cm | grep stork | awk '{print $1}'); do kubectl -n kube-system delete cm $cm;done
kubectl -n kube-system delete cm vsphere-ds-lock
```
Patching the storagecontroller 

```
# Get the namespace where portworx is installed.
export pxnamespace=$(kubectl get stc -A | grep -v NAMESPACE | awk '{print $1}')
# Get the stc resource.
export STC=`kubectl -n $pxnamespace get stc | grep -v NAME | cut -f1 -d' '`

# Various useful patches

# Enable telemetry
kubectl -n $pxnamespace patch stc $STC --type=merge --patch '{"spec":{"monitoring":{"telemetry":{"enabled":true}}}}'
# Disable telemetry
kubectl -n $pxnamespace patch stc $STC --type=merge --patch '{"spec":{"monitoring":{"telemetry":{"enabled":false}}}}'

# Disable creation of PX built-in storageclasses. Note that this will remove exisiting ones. 
kubectl -n $pxnamespace patch stc <mystoragecluster> --type=merge --patch '{"metadata":{"annotations":{"portworx.io/disable-storage-class":"true"}}}'
```