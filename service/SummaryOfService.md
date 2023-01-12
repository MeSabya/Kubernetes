- **I would like you to imagine that if you create a NodePort service it also creates a ClusterIP one. And if you create a LoadBalancer it creates a NodePort which then creates a ClusterIP**
- Services point to pods. Services do not point to deployments or replicasets. Services point to pods directly using labels.

- Services are not scheduled on any specific Node, lets just say they are “available everywhere in the cluster”.

- 

This one is the best to learn:
https://medium.com/swlh/kubernetes-services-simply-visually-explained-2d84e58d70e5

https://medium.com/@wuestkamp/kubernetes-ingress-simply-visually-explained-d9cad44e4419
