## Problem1
Deployment naboo is created. Make sure the replicas autoscale with minimum 2 and maximum 5 when at 80% CPU. Use naboo as the name of HPA resource.

### Ans:
kubectl autoscale deploy naboo --name=naboo --min=2 --max=5 --cpu-percent=80

## Problem2
Create a Cron job bespin that runs every 5 minutes(*/5 * * * *) and runs command date. Use alpine image.

### Ans
kubectl create cronjob bespin --image=alpine --schedule="*/5 * * * *" -- date

## Problem3
Label node node01 with shuttle=true.

Remove annotation flagship=finalizer form node01.

### Ans
kubectl label node node01 shuttle=true

kubectl annotate node node01 flagship-

## Problem4
Get the name and image of all pods in skywalker namespace having label jedi=true. Write the output to /root/jedi-true.txt.
Output should be in the following format. Use jsonpath.

podname,image

podname2,image

### Ans

```yaml
kubectl get po  -n skywalker --selector=jedi=true -o jsonpath="{range .items[*]}{.metadata.name},{.spec.containers[0].image}{'\n'}{end}" > /root/jedi-true.txt
```

## Problem5
Create a pod myredis with image redis. Define a liveness probe and readiness probe with an initial delay of 5 seconds and command redis-cli PING.

### Ans

```yaml
cat << EOF > myredis.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: myredis
  name: myredis
spec:
  containers:
  - image: redis
    name: myredis
    livenessProbe:
      exec:
        command:
        - redis-cli
        - PING
      initialDelaySeconds: 5
    readinessProbe:
      exec:
        command:
        - redis-cli
        - PING
      initialDelaySeconds: 5
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF
```

## Problem 
Create a Pod named myenv with command sh -c "printenv && sleep 1h". Use alpine image.

Save the logs of the pod to /root/myenv.log file.

Delete myenv pod.

### Ans
kubectl run myenv --image=alpine -- sh -c "printenv && sleep 1h"

Wait few seconds for pod to run.

kubectl logs myenv > /root/myenv.log

kubectl delete po myenv

## Problem 
Create a pod httptest with image kennethreitz/httpbin. Define a readiness probe at path /status/200 on port 80 of the container.

### Ans
```yaml
cat << EOF > httptest.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: httptest
  name: httptest
spec:
  containers:
  - image: kennethreitz/httpbin
    name: httptest
    readinessProbe:
      httpGet:
        path: /status/200
        port: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF
```

## Problem
Find the name of pod which is using most CPU across all namespaces. Enter the name of pod in /root/high-cpu.yaml.

### Ans
Run kubectl top po -A --sort-by=cpu and put the name of first pod to /root/high-cpu.yaml file.

## Problem
Pod and Service geonosis is created for you. Create a network policy geonosis-shield which allows only pods with label empire=true to access the service. Use appropriate labels.

### Ans
```yaml
cat << EOF > geonosis-shield.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: geonosis-shield
spec:
  podSelector:
    matchLabels:
      sector: arkanis
  ingress:
  - from:
    - podSelector:
        matchLabels:
          empire: "true"
EOF
```

