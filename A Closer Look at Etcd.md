# A Closer Look at Etcd: The Brain of a Kubernetes Cluster

Etcd is a crucial component for Kubernetes as it stores the entire state of the cluster: its configuration, specifications, and the statuses of the running workloads.

👉 In the Kubernetes world, etcd is used as the backend for service discovery and stores the cluster’s state and its configuration.

Etcd is deployed as a cluster, several nodes whose communications are handled by the Raft algorithm. In a production environment, a cluster contains an odd number of nodes and at least three are required.


## Etcd in Kubernetes

In the context of a Kubernetes cluster, etcd instances can be deployed as Pods on the masters (this is the example we will use in this post).

![image](https://user-images.githubusercontent.com/33947539/177320898-7dec58f0-67a1-49a5-8f1a-d6dd2abdfa3b.png)

To add an additional level of security and resiliency it can also be deployed as an external cluster.

![image](https://user-images.githubusercontent.com/33947539/177320939-289fcda5-9612-4631-a3aa-60b1db7dd732.png)

The components involved during a simple Pod creation process. It’s a great illustration of the API Server and etcd interaction.

![image](https://user-images.githubusercontent.com/33947539/177321000-47fc810c-6847-4ba8-8872-25bee416d853.png)
