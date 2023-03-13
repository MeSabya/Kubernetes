## Important fields of Persistent volume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
```

## Important fields of Persistent volume claim

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

## What will happend in the above case , where persistent volume and persistent volume claim , access modes does not match ?
If access modes of the PV and PVC does not match then PV and PVC will not be linked.

## Once Persistent volume is created it is immutable. meaning if you have created a PVC, and modified the yaml file again, and try to apply the modified chnages on the PVC, it will fail. We need to delete the created PVC and then reapply again.


## When persistentVolumeReclaimPolicy is retain, what is the significance of it?
When PVC is deleted, then PV will not be deleted but also will not be availble to access.
PV will be in a released state.


