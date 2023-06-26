## Set Node Affinity to the deployment to place the pods on node01 only.

Name: blue

Replicas: 3

Image: nginx

NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution

Key: color

value: blue

### Answer

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
```
### Create a taint on node01 with key of spray, value of mortein and effect of NoSchedule

```shell
kubectl taint nodes node01 spray=mortein:NoSchedule
```
### Create POD which can scheduled on node01 , the pod should able to tolerate the taint 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
  tolerations:
  - key: spray
    value: mortein
    effect: NoSchedule
    operator: Equal
```

### How to untaint a node 
```shell
kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```
### How to know on which node the pod has been scheduled 

```shell
kubectl get pods -o wide
```
