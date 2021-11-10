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

## Kubernetes Pod Affinity and Anti-Affinity
- Affinity and anti-affinity allow you to control on which nodes the pods in your deployment can be scheduled. While this feature is not specific to deployments, it can be very useful for many applications.


When configuring affinity or anti-affinity , There are two options:
- **requiredDuringSchedulingIgnoredDuringExecution**: the pod cannot be scheduled on a node unless it matches the affinity configuration, even if there are no nodes that match
- **preferredDuringSchedulingIgnoredDuringExecution**: the scheduler will attempt to schedule the pod on a node matching the affinity configuration, but if it is unable to do so the pod will still be scheduled on another node.

**Pod Affinity**:Pod affinity is used to schedule pods onto certain nodes. You would normally want to do this if you know that a pod has a specific resource requirement that can be met by a specific set of nodes.
**Pod AntiAffinity**:Pod anti-affinity is useful for ensuring that pods in a Deployment are not scheduled all on one node. 

### References:
- https://learnk8s.io/kubernetes-rollbacks
- https://www.bluematador.com/blog/kubernetes-deployments-rolling-update-configuration
