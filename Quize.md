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


