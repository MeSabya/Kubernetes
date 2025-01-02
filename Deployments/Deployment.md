# Deployments

- A deployment is an object in Kubernetes that helps you to manage a group of identical pods.
- Without a deployment, you’d got to produce, update, and delete a bunch of pods manually.
- Using Kubernetes deployment we can autoscale the applications very easily.
- ❗Every time you create a Deployment, the deployment creates a ReplicaSet and delegates creating (and deleting) the Pods.
- ❗**Deployments are usually used for stateless applications. However, you can save the state of deployment by attaching a Persistent Volume to it and make it stateful, 
  but all the pods of a deployment will be sharing the same Volume and data across all of them will be same.**
  
![image](https://user-images.githubusercontent.com/33947539/141157600-57c6f1d8-4045-4243-bb3a-04855fa46436.png)

- ❗In Deployments, you can also manually roll back to a previous ReplicaSet, if needed in case if your new feature is not working as expected.
   - [rollback in K8s](https://github.com/MeSabya/Kubernetes/blob/main/Deployments/Deployment.md#rolling-back-to-previous-version) 

Deployments are very flexible and can be used in many ways. Below I have selected most important problems that deployments help solve.
- **Scalability**: enable up and down-scaling of pods
- **Configuration**: enables changing of pods state and configuration on the flight
- **Deployment**: enables zero-downtime updates of pods to new versions
- **Delivery Control**: enables high degree of control over delivery process by using rollouts and rollbacks

Typical deployment resource consists of following objects:

![image](https://user-images.githubusercontent.com/33947539/141118126-148d24cd-8d5f-4755-ba61-8c82a5be9eac.png)

❗As usual **apiVersion, kind, metadata and spec** are mandatory fields in every Kubernetes resource. 
Deployment adds following important fields under spec:
- replicas: number of pods replicated via the deployment
- selector: tells Kubernetes how deployment should find pods to act on
- template: fields under this section refer to pod specification that deployment acts on

## Deployment Stratergies:
Kubernetes supports two types of deployment strategies:
- **RollingUpdate**: New pods are added gradually, and old pods are terminated gradually
- **Recreate**: All old pods are terminated before any new pods are added

### RollingUpdate below:
- Deployments, as discussed, creates a ReplicaSet which then creates a Pod so whenever you update the deployment using RollingUpdate(default) strategy, 
  a new ReplicaSet is created and the Deployment moves the Pods from the old ReplicaSet to the new one at a controlled rate. 
- ❗**Rolling Update means that the previous ReplicaSet doesn’t scale to 0 unless the new ReplicaSet is up & running ensuring 100% uptime.**

![image](https://user-images.githubusercontent.com/33947539/141145533-f55c0b1a-a4ef-4e2c-900d-d0533418cfb5.png)

Rollback is simply the reverse of rolling update. Kubernetes stores state of previous updates, so it’s very easy to revert to previous revision.
When using the RollingUpdate strategy, there are two more options that let you fine-tune the update process:

- **maxSurge**: The maximum number of new pods that will be created at a time.
- **maxUnavailable**: The maximum number of old pods that will be deleted at a time.

## Rolling Back to previous version
- The ReplicaSet can hold only a single type of Pod.So you can't have version 1 and version 2 of the Pods in the same ReplicaSet.
- **When you upgrade your Pods from version 1 to version 2, the Deployment creates a new ReplicaSet and increases the count of replicas while the previous count goes to zero.**
- **So what happens when another rolling update from version 2 to version 3** ?
   - you might notice that at the end of the upgrade, you have two ReplicaSets with a count of 0.
- ❗After the rolling update, the previous ReplicaSet is not deleted — not immediately at least.Why ?
  - **keeping the previous ReplicaSets around is a convenient mechanism to roll back to a previously working version of your app.**
  - By default Kubernetes stores the last 10 ReplicaSets and lets you roll back to any of them.
  - But you can change how many ReplicaSets should be retained by changing the spec.revisionHistoryLimit in your Deployment.
#### Could you list all the previous Replicasets that belong to a Deployment?
:point_right:kubectl rollout history deployment/app
#### And how can you rollback to a specific version with?
:point_right:kubectl rollout undo deployment/app --to-revision=2
#### How to check that the rolling update proceeds as expected?
:point_right:kubectl rollout status deployment myapp
 - If the deployment fails, the command exits with a non-zero return code to indicate a failure.

## How does minReadySeconds affect readiness probe?
Let's say I have a deployment template like this:
![image](https://user-images.githubusercontent.com/33947539/141172964-a831a3cc-55c1-4fa8-982c-d476906d00ce.png)

❗**.spec.minReadySeconds** is an optional field that specifies the minimum number of seconds for which a newly created Pod should be ready without any of its containers crashing, for it to be considered available. This defaults to 0 (the Pod will be considered available as soon as it is ready).

**initialDelaySeconds**: Number of seconds after the container has started before liveness or readiness probes are initiated.

❗So initialDelaySeconds comes before minReadySeconds.

❗Lets say, container in the pod has started at t seconds. Readiness probe will be initiated at t+initialDelaySeconds seconds. Assume Pod become ready at t1 seconds(t1 > t+initialDelaySeconds). So this pod will be available after t1+minReadySeconds seconds.

## Deployment Usecase Analysis:
For example, say you had a deployment running 5 pods that read from an event stream, process events, and save them to a database. It takes each pod about 60 seconds to warm up and process events at full speed. In the default configuration, the pods would be replaced and immediately become ready, but they would be slow to process events for the first minute. Immediately after your update is finished, this event processing system will have fallen behind and will need to catch up since all of the pods had to warm up at the same time. 
### Solution
you can set your maxSurge to 1, maxUnavailable to 0, and **[minReadySeconds](https://github.com/MeSabya/Kubernetes/blob/main/Deployments/Deployment.md#how-does-minreadyseconds-affect-readiness-probe)** to 60. This would ensure new pods would be created one at a time, a minute would pass between pods being added, and old pods would only be removed once new pods have already warmed up. This way you update all your pods over the course of ~5 minutes, and your processing times remain stable.

## Kubernetes Pod Affinity/Anti-Affinity and taints and tolerations
- Affinity and anti-affinity allow you to control on which nodes the pods in your deployment can be scheduled. While this feature is not specific to deployments, it can be very useful for many applications.


When configuring affinity or anti-affinity , There are two options:
- **requiredDuringSchedulingIgnoredDuringExecution**: the pod cannot be scheduled on a node unless it matches the affinity configuration, even if there are no nodes that match
- **preferredDuringSchedulingIgnoredDuringExecution**: the scheduler will attempt to schedule the pod on a node matching the affinity configuration, but if it is unable to do so the pod will still be scheduled on another node.

### 1. Node Affinity
Node Affinity is a set of rules that influence which nodes a Pod can be scheduled on. It’s similar to nodeSelector, but more expressive and flexible. With Node Affinity, you can specify conditions based on node labels, allowing you to place Pods on nodes that match certain criteria.

### There are two types of Node Affinity:

#### a. RequiredDuringSchedulingIgnoredDuringExecution
This means that the Pod will only be scheduled on nodes that satisfy the conditions specified in the nodeAffinity field. If no node satisfies the conditions, the Pod will remain unscheduled.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "kubernetes.io/hostname"
            operator: In
            values:
            - "node-1"
            - "node-2"
  containers:
    - name: example-container
      image: busybox
```
This Pod will only be scheduled on node-1 or node-2.

#### b. PreferredDuringSchedulingIgnoredDuringExecution
This is a soft constraint. It tells Kubernetes that, if possible, it should schedule the Pod on a node that satisfies the conditions. However, if no node meets the criteria, Kubernetes can still schedule the Pod on another node that doesn’t meet the condition.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
            - key: "kubernetes.io/zone"
              operator: In
              values:
              - "us-east-1a"
  containers:
    - name: example-container
      image: busybox
```
Kubernetes will prefer to schedule the Pod in the us-east-1a zone, but it will still schedule it elsewhere if necessary.

#### Common Use Cases for Node Affinity
- Spread Pods Across Zones or Regions: To ensure high availability, you can schedule Pods across different availability zones or regions.
- Node Specific Requirements: Certain workloads may require specific hardware resources or configurations that are only available on certain nodes (e.g., GPUs, large memory nodes).
- Isolation of Workloads: Use Node Affinity to isolate workloads to specific nodes (e.g., separating batch jobs from web servers).

### 2. Taints and Tolerations
Taints and Tolerations provide a way to repel Pods from being scheduled onto certain nodes unless they explicitly tolerate the taint on the node. This helps ensure that only specific Pods are allowed to run on nodes with special configurations or requirements.

#### a. Taints
A taint is a property added to a node that prevents Pods from being scheduled onto it unless the Pods tolerate the taint. Taints have three parts:

- Key: A key for the taint (e.g., key1).
- Value: An optional value associated with the key (e.g., value1).
- Effect: What happens to Pods that do not tolerate the taint. There are three possible effects:

    - NoSchedule: The Pod will not be scheduled on the node.
    - PreferNoSchedule: Kubernetes will try to avoid scheduling the Pod on this node.
    - NoExecute: The Pod is not only unscheduled but will also be evicted if it’s already running on the node.

Example of adding a taint to a node:

```bash
kubectl taint nodes node-1 key1=value1:NoSchedule
```

This command adds a taint to node-1, preventing Pods without the matching toleration from being scheduled on it.

#### b. Tolerations
Tolerations allow Pods to be scheduled on nodes with matching taints. A toleration is added to a Pod specification, and it defines which taints the Pod is willing to tolerate.

Example of a Pod with a toleration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
  containers:
    - name: example-container
      image: busybox
This Pod can be scheduled on nodes that have the taint key1=value1:NoSchedule.
```
### Combining Node Affinity with Taints and Tolerations
In some scenarios, it’s useful to combine Node Affinity and Taints/Tolerations to ensure more precise control over where Pods are scheduled. Here’s an example:

Node Affinity could specify that a Pod must be scheduled on nodes in a particular zone.
Taints could prevent most Pods from being scheduled on a node with high load or specific hardware needs.
Tolerations would then allow only certain Pods (e.g., critical workloads or specialized workloads) to tolerate the taint and be scheduled on the tainted node.
Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "kubernetes.io/zone"
            operator: In
            values:
            - "us-west-1a"
  tolerations:
  - key: "critical"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
    - name: example-container
      image: busybox
```

This Pod will only be scheduled on us-west-1a and can only be scheduled on nodes that have the taint critical=true:NoSchedule.

## Commands used in Deployment:
#### List deployments:
:point_right:kubectl get deploy

#### Update a deployment with a manifest file:
:point_right:kubectl apply -f test.yaml

#### Scale a deployment “test” to 3 replicas:
:point_right:kubectl scale deploy/test --replicas=3

#### Watch update status for deployment “test”:
:point_right:kubectl rollout status deploy/test

#### Pause deployment on “test”:
:point_right:kubectl rollout pause deploy/test

#### Resume deployment on “test”:
:point_right:kubectl rollout resume deploy/test

#### View rollout history on “test”:
:point_right:kubectl rollout history deploy/test

#### Undo most recent update on “test”:
:point_right:kubectl rollout undo deploy/test

#### Rollback to specific revision on “test”:
:point_right:kubectl rollout undo deploy/test --to-revision=1
 

## References:
- https://learnk8s.io/kubernetes-rollbacks
- https://www.bluematador.com/blog/kubernetes-deployments-rolling-update-configuration
