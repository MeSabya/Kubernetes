# Deployments

- A deployment is an object in Kubernetes that helps you to manage a group of identical pods.
- Without a deployment, you’d got to produce, update, and delete a bunch of pods manually.
- Using Kubernetes deployment we can autoscale the applications very easily.
- ❗Every time you create a Deployment, the deployment creates a ReplicaSet and delegates creating (and deleting) the Pods.
- ❗**Deployments are usually used for stateless applications. However, you can save the state of deployment by attaching a Persistent Volume to it and make it stateful, 
  but all the pods of a deployment will be sharing the same Volume and data across all of them will be same.**
  
![image](https://user-images.githubusercontent.com/33947539/141157600-57c6f1d8-4045-4243-bb3a-04855fa46436.png)

- ❗In Deployments, you can also manually roll back to a previous ReplicaSet, if needed in case if your new feature is not working as expected.
   - We will learn this rollback in 

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
- Rolling update
- Recreate

### We are only looking at RollingUpdate below:
- Deployments, as discussed, creates a ReplicaSet which then creates a Pod so whenever you update the deployment using RollingUpdate(default) strategy, 
  a new ReplicaSet is created and the Deployment moves the Pods from the old ReplicaSet to the new one at a controlled rate. 
- ❗**Rolling Update means that the previous ReplicaSet doesn’t scale to 0 unless the new ReplicaSet is up & running ensuring 100% uptime.**

![image](https://user-images.githubusercontent.com/33947539/141145533-f55c0b1a-a4ef-4e2c-900d-d0533418cfb5.png)

Rollback is simply the reverse of rolling update. Kubernetes stores state of previous updates, so it’s very easy to revert to previous revision.

## Rolling Back to previous version


