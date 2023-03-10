### Create a new deployment named blue with the nginx image and 3 replicas.

Name: blue

Replicas: 3

Image: nginx

```yaml
kubectl create deployment blue --image=nginx --replicas=3
```

