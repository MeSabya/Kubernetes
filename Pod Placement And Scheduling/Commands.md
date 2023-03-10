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
