# Exploring the Effects by Violating Quotas

## Exploring the Effects#
Now let’s create the objects and explore the effects as we defined the resource quotas in the previous lesson.

## Creating the dev Namespace#
Let’s get started by creating the dev Namespace as per our plan.

```shell
kubectl create ns dev

kubectl create \
    -f dev.yml \
    --record --save-config
```
We can see from the output that the namespace "dev" was created as well as the resourcequota "dev". To be on the safe side, we’ll describe the newly created devquota.

```shell
kubectl --namespace dev describe \
    quota dev
```

The output is as follows.

```shell
Name:               dev
Namespace:          dev
Resource            Used  Hard
--------            ----  ----
limits.cpu          0     1
limits.memory       0     1Gi
pods                0     10
requests.cpu        0     800m
requests.memory     0     500Mi
services.nodeports  0     0
```

We can see that the hard limits are set and that there’s currently no usage. That was to be expected since we’re not running any objects in the dev Namespace.

## Creating Resources #
Let’s spice it up a bit by creating the already too familiar go-demo-2 objects.

```shell
kubectl --namespace dev create \
    -f go-demo-2.yml \
    --save-config --record

kubectl --namespace dev \
    rollout status \
    deployment go-demo-2-api
```

We created the objects from the go-demo-2.yml file and waited until the go-demo-2-api Deployment rolled out.

## Looking into the Description #
Now we can revisit the values of the dev quota.

```shell
kubectl --namespace dev describe \
    quota dev
```

The output is as follows.

```shell
Name:              dev
Namespace:         dev
Resource           Used  Hard
--------           ----  ----
limits.cpu         400m  1
limits.memory      130Mi 1Gi
pods               4     10
requests.cpu       40m   800m
requests.memory    65Mi  500Mi
services.nodeports 0     0
```

Judging from the Used column, we can see that we are, for example, currently running 4 Pods and that we are still below the limit of 10. One of those Pods was created through the go-demo-2-db Deployment, and the other three with the go-demo-2-api. If you summarize resources we specified for the containers that form those Pods, you’ll see that the values match the used limits and requests.

## Violating the Number of Pods#
So far, we did not reach any of the quotas. Let’s try to break at least one of them go-demo-2-scaled.yml.

```yaml
...
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: go-demo-2-api
spec:
  replicas: 15
  ...
```
The definition of the go-demo-2-scaled.yml is almost the same as the one in go-demo-2.yml. The only difference is that the number of replicas of the go-demo-2-api Deployment is increased to fifteen. As you already know, that should result in fifteen Pods created through that Deployment.

## Applying the Definition #
I’m sure you can guess what will happen if we apply the new definition. We’ll do it anyway.

```shell
kubectl --namespace dev apply \
    -f go-demo-2-scaled.yml \
    --record
```
We applied the new definition. We’ll give Kubernetes a few moments to do the work before we take a look at the events it’ll generate. So, take a deep breath and count from one to the number of processors in your machine.

```shell
kubectl --namespace dev get events
```
The output of a few of the events generated inside the dev Namespace is as follows.

```shell
...
... Error creating: pods "go-demo-2-api-..." is forbidden: exceeded quota: dev, requested: limits.cpu=100m,pods=1, used: limits.cpu=1,pods=10, limited: limits.cpu=1,pods=10
13s         13s          1         go-demo-2-api-6bd767ffb6.150f51f4b3a7ed3f         ReplicaSet                          Warning ... Error creating: pods "go-demo-2-api-..." is forbidden: exceeded quota: dev, requested: limits.cpu=100m,pods=1, used: limits.cpu=1,pods=10, limited: limits.cpu=1,pods=10
...
```

We can see that we reached two of the limits imposed by the Namespace quota. We reached the maximum amount of CPU (1) and Pods (10). As a result, ReplicaSet controller was forbidden from creating new Pods.

## Analyzing the Error #
We should be able to confirm which hard limits were reached by describing the dev Namespace.

![image](https://user-images.githubusercontent.com/33947539/185915757-1828056c-de58-47d5-92af-5ee76ff18285.png)

As the events showed us, the values of limits.cpu and pods resources are the same in both User and Hard columns. As a result, we won’t be able to create any more Pods, nor will we be allowed to increase CPU limits for those that are already running.

Finally, let’s take a look at the Pods inside the dev Namespace.

![image](https://user-images.githubusercontent.com/33947539/185915846-583531cd-257f-4213-91a7-f758fbef3410.png)

The go-demo-2-api Deployment managed to create nine Pods. Together with the Pod created through the go-demo-2-db, we reached the limit of ten.

## Reverting Back to Previous Definition #
We confirmed that the limit and the Pod quotas work. We’ll revert to the previous definition (the one that does not reach any of the quotas) before we move onto the next verification.

```shell
kubectl --namespace dev apply \
    -f go-demo-2.yml \
    --record

kubectl --namespace dev \
    rollout status \
    deployment go-demo-2-api
```
The output of the latter command should indicate that the deployment "go-demo-2-api" was successfully rolled out.

## Violating the Memory Quota#
Let’s take a look at yet another slightly modified definition of the go-demo-2 objects go-demo-2-mem.yml.

```yaml
...
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-2-db
spec:
  ...
  template:
    ...
    spec:
      containers:
      - name: db
        image: mongo:3.3
        resources:
          limits:
            memory: "100Mi"
            cpu: 0.1
          requests:
            memory: "50Mi"
            cpu: 0.01
...
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-2-api
spec:
  replicas: 3
  ...
  template:
    ...
    spec:
      containers:
      - name: api
        ...
        resources:
          limits:
            memory: "200Mi"
            cpu: 0.1
          requests:
            memory: "200Mi"
            cpu: 0.01
...
```
Both memory request and limit of the api container of the go-demo-2-api Deployment is set to 200Mi while the database remains with the memory request of 50Mi. Knowing that the requests.memory quota of the dev Namespace is 500Mi, it’s enough to do simple math and come to the conclusion that we won’t be able to run all three replicas of the go-demo-2-api Deployment.

## Applying the Definition #

![image](https://user-images.githubusercontent.com/33947539/185916214-247898ef-0646-4556-8764-ce7fa6d6b994.png)

We reached the quota of the requests.memory. As a result, creation of at least one of the Pods is forbidden. We can see that we requested creation of a Pod that requests 200Mi of memory. Since the current summary of the memory requests is 455Mi, creating that Pod would exceed the allocated 500Mi.

## Analyzing the Error #
Let’s take a closer look at the Namespace.

![image](https://user-images.githubusercontent.com/33947539/185917241-f65d1e41-3f5a-433c-8cd1-d3aa7b1b8f8f.png)

Indeed, the amount of used memory requests is 455Mi, meaning that we could create additional Pods with up to 45Mi, not 200Mi.

### Reverting Back to the Previous Definition #
We’ll revert to the go-demo-2.yml one more time before we explore the last quota we defined.

```shell
kubectl --namespace dev apply \
    -f go-demo-2.yml \
    --record

kubectl --namespace dev \
    rollout status \
    deployment go-demo-2-api
```

## Violating the Services Quota #
The only quota we did not yet verify is services.nodeports. We set it to 0 and, as a result, we should not be allowed to expose any node ports. Let’s confirm that is indeed true.

```shell
kubectl expose deployment go-demo-2-api \
    --namespace dev \
    --name go-demo-2-api \
    --port 8080 \
    --type NodePort
```

The output is as follows.

```shell
Error from server (Forbidden): services "go-demo-2-api" is forbidden: exceeded quota: dev, requested: services.nodeports=1, used: services.nodeports=0, limited: services.nodeports=0
```

All our quotas work as expected. But, there are others. We won’t have time to explore examples of all the quotas we can use. Instead, we’ll list them all for future reference.






