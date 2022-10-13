## Problem1
Create a secret db-secret with value MYSQL_ROOT_PASSWORD=YoYoSecret and MYSQL_PASSWORD=XoXoPassword.

Create a configmap db-config with value MYSQL_USER=k8s and MYSQL_DATABASE=newdb.

Create a pod named mydb with image mysql:5.7 and expose all values of db-secret and db-config as environment variable to pod.


```yaml
kubectl create secret generic db-secret --from-literal='MYSQL_ROOT_PASSWORD=YoYoSecret' --from-literal='MYSQL_PASSWORD=XoXoPassword'

kubectl create configmap db-config --from-literal='MYSQL_USER=k8s' --from-literal='MYSQL_DATABASE=newdb'

```

```yaml

cat << EOF > mydb.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mydb
  name: mydb
spec:
  containers:
  - image: mysql:5.7
    name: mydb
    envFrom:
    - configMapRef:
        name: db-config
    - secretRef:
        name: db-secret
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF
```

