# Deployments vs StatefulSets vs DaemonSets
In this post, will be discussing three different ways to deploy your application(pods) on Kubernetes using different Kubernetes resources. 
Below are 3 different resources that Kubernetes provides for deploying pods.
  - Deployments
  - StatefulSets
  - DaemonSets
  
There is one other type ReplicationController but Kubernetes now favors Deployments as Deployments configure ReplicaSets to support replication.


## Deployments
Deployments are usually used for stateless applications. owever, you can save the state of deployment by attaching a Persistent Volume to it and make it stateful, 
but all the pods of a deployment will be sharing the same Volume and data across all of them will be same.


**Step1**: To deploy the pod use the command below:
> kubectl apply -f deployment.yaml

**Step2**: To get the number of persistent volumes created :
> kubectl get pv 

**Step3**: To get into the container shell and check into the file:
> kubectl exec -it <container name> -- bash


Suppose we have created two pods , pod-xyzA, pod-xyzB . Then in **step2**: one pod must be printing logs starting from 1 and another pod must be starting from where pod1 has left.
they both are sharing the same file and volume and data is shared across all pods of a Deployment. Also if you check the Persistent Volume Claims(PVCs), only one PVC will be 
created that both the pods will be sharing so it can cause Data Inconsistency. If we check the **step3** this can be verified that both pods sharing the same file.

![image](https://user-images.githubusercontent.com/33947539/136932732-9a63b5ff-5d29-4d5b-b058-9f6b86a2b4ab.png)
  
**Deployment Strategy**: Rolling Update
  
***What is Rolling Update***
Rolling Update means that the previous ReplicaSet doesn’t scale to 0 unless the new ReplicaSet is up & running ensuring 100% uptime. If an error occurs while updating,
the new ReplicaSet will never be in Ready state, so old ReplicaSet will not terminate again ensuring 100% uptime in case of a failed update.
Deployments, as discussed, creates a ReplicaSet which then creates a Pod so whenever you update the deployment using RollingUpdate(default) strategy, 
a new ReplicaSet is created and the Deployment moves the Pods from the old ReplicaSet to the new one at a controlled rate.

**In Deployments, you can also manually roll back to a previous ReplicaSet, if needed in case if your new feature is not working as expected.**

 
## Statefulsets
ReplicaSet, Deployments, and DaemonSet work seamlessly with stateless applications but they are not suitable for stateful applications like MySQL, 
Kafka etc. This is where StatefulSets comes to the picture. Imagine a scenario where you want to **deploy your containers in order**, this is possible with StatefulSets.

### Understanding the container creation and scaling:
For example, when you initially create a StatefulSet, the first replica is created, and Kubernetes waits for it to become healthy and available before creating the second replica. This means that when the second replica is being created, you can depend on the fact that the zeroth index (1st replica) is available for you to connect to. It can point back to the original member of the StatefulSet

StatefulSets in Kubernetes are used for Stateful applications. The pods in StatefulSets are named with unique Identities and stable hostname. 
It stores the state information and other resilient data in a persistent volume. This persistent volume is attached to Cloud-based storage. 
It is going to be responsible for maintaining the state of your application.
A persistent volume claim in the context of your StatefulSet, so that when that StatefulSet is created and each replica is created, 
Kubernetes will go ahead and create a different disk for each replica in the StatefulSet.

### Complexities associated with ReplicaSet for Stateful application 
Now, when Kubernetes started, the only sort of way that you could do replication was using a ReplicaSet. With a replica set, every single replica was 
treated entirely identically. They have random hashes on the end of their application names. And if a scaling event happens, for example, a scaled-down, 
a container is chosen at random and deleted. These characteristics make ReplicaSet very hard to map to stateful applications. Many stateful applications expect 
their hostnames to be constant. So, those complexities of using ReplicaSet and stateful applications led to the eventual development of StatefulSets. 
**A StatefulSets in Kubernetes is similar to a ReplicaSet, but it adds some guarantees that make it easier to manage stateful applications inside of Kubernetes.**
  
So, for example, if there are 3 pods of MySQL, the names would be mysql-0 , mysql-1 and mysql-2. And, if any of these pod fail, a new pod with the same name 
will be deployed by StatefulSets. A headless service is required by StatefulSets to manage unique identities. 
The diagram below shows the architecture of StatefulSets:
  ![image](https://user-images.githubusercontent.com/33947539/136938636-88a10df7-5190-483c-bb6a-d4efb075c222.png)



### Pod creation and naming in Statefulsets
StatefulSet is also a Controller but unlike Deployments, it doesn’t create ReplicaSet rather itself creates the Pod with a unique naming convention. 
e.g. If you create a StatefulSet with name counter, it will create a pod with name counter-0, and for multiple replicas of a statefulset, 
their names will increment like counter-0, counter-1, counter-2, etc

**Every replica of a stateful set will have its own state, and each of the pods will be creating its own PVC(Persistent Volume Claim). 
So a statefulset with 3 replicas will create 3 pods, each having its own Volume, so total 3 PVCs.**

 ![image](https://user-images.githubusercontent.com/33947539/136935095-81e63b9a-7cd2-4891-92d9-e62f03ee2703.png)
  
**Deployment Strategy**: Rolling Update
StatefulSets don’t create ReplicaSet or anything of that sort, **so you cant rollback a StatefulSet to a previous version**. 
You can only delete or scale up/down the Statefulset. 


## DaemonSets

A DaemonSet is a controller that ensures that the pod runs on all the nodes of the cluster. If a node is added/removed from a cluster, 
DaemonSet automatically adds/deletes the pod.

### Some typical use cases of a DaemonSet is to run cluster level applications like:
**Monitoring Exporters**: You would want to monitor all the nodes of your cluster so you will need to run a monitor on all the nodes of the cluster like NodeExporter.
**Logs Collection Daemon**: You would want to export logs from all nodes so you would need a DaemonSet of log collector like Fluentd to export logs from all your nodes.

![image](https://user-images.githubusercontent.com/33947539/136935755-f5481a1b-e8ee-4d15-a9f7-1657c14e3df7.png)

 **Deployment Strategy** : Rolling Update
 If you update a DaemonSet, it also performs RollingUpdate i.e. one pod will go down and the updated pod will come up, then the next replica 
 pod will go down in same manner e.g. If I change the image of the above DaemonSet, one pod will go down, and when it comes back up with the updated image, 
 only then the next pod will terminate and so on. If an error occurs while updating, so only one pod will be down, all other pods will still be up, 
 running on previous stable version. Unlike Deployments, you cannot roll back your DaemonSet to a previous version
  
# **So after all these discussions we should able to answer the question:**
  
***Why StatefulSets? Can't a stateless Pod use persistent volumes?***
Yes, a regular pod can use a persistent volume. However, sometimes you have multiple pods that logically form a "group". 
**Examples** of this would be database replicas, ZooKeeper hosts, Kafka nodes, etc. In all of these cases there's a bunch of servers and they work together and talk to each other. What's special about them is that each individual in the group has an identity. For example, for a database cluster one is the master and two are followers and each of the followers communicates with the master letting it know what it has and has not synced. So the followers know that "db-x-0" is the master and the master knows that "db-x-2" is a follower and has all the data up to a certain point but still needs data beyond that.
  
**In such situations you need a few things you can't easily get from a regular pod:**

**A predictable name**: you want to start your pods telling them where to find each other so they can form a cluster, elect a leader, etc. but you need to know their names in advance to do that. Normal pod names are random so you can't know them in advance.

**A stable address/DNS name**: you want whatever names were available in step (1) to stay the same. If a normal pod restarts (you redeploy, the host where it was running dies, etc.) on another host it'll get a new name and a new IP address.

 **A persistent link between an individual in the group and their persistent volume**: if the host where one of your database master was running dies it'll get moved to a new host but should connect to the same persistent volume as there's one and only 1 volume that contains the right data for that "individual". So, for example, if you redeploy your group of 3 database hosts you want the same individual (by DNS name and IP address) to get the same persistent volume so the master is still the master and still has the same data, replica1 gets it's data, etc.
  
StatefulSets solve these issues because they provide:
- Stable, unique network identifiers.
- Stable, persistent storage.
- Ordered, graceful deployment and scaling.
- Ordered, graceful deletion and termination.

As some have noted, you can indeed can some of the same benefits by using regular pods and services, but its much more work. For example, if you wanted 3 database instances you could manually create 3 deployments and 3 services. Note that you must manually create 3 deployments as you can't have a service point to a single pod in a deployment. Then, to scale up you'd manually create another deployment and another service. This does work and was somewhat common practice before PetSet/PersistentSet came along. Note that it is missing some of the benefits listed above (persistent volume mapping & fixed start order for example).
