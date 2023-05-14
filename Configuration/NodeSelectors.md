Let us start with a simple example.  You have a three-node cluster of which two are smaller nodes  with lower hardware resources, and one of them  is a 
larger node configured with higher resources.  You have different kinds of workloads  running in your cluster. 

### Requirements
You would like to dedicate the data processing workloads  that require higher horsepower to the larger node  as that is the only node that will not run out of resources  in case the job demands extra resources.  However, in the current default setup,  any pods can go to any nodes.  
So Pod C in this case may very well end up  on nodes two or three, which is not desired.  To solve this, we can set a limitation on the pods so that they only run on particular nodes.  

There are two ways to do this.  

- The first is using nodeSelectors  which is the simple and easier method.
  For this, we look at the pod definition file  we created earlier.  This file has a simple definition to create a pod  with a data processing image. 
To limit this pod to run on the larger node,  we add a new property called **nodeSelector  to the spec section and specify the size as large.** 

👉 But wait a minute, where did the size large come from  and how does Kubernetes know which is the large node?  

The key value pair of size and large  are in fact labels assigned to the node.  The scheduler uses these labels to match and identify  the right node to place the pods on.  Labels and selectors are a topic we have seen many times  throughout this Kubernetes course,  such as with services, replica sets, and deployments.

To use labels in a nodeSelector like this,  you must have first labeled your nodes  prior to creating this pod.  

So let us go back  and see how we can label the nodes.  To label a node, use the command kube-control label nodes  followed by the name of the node and the label in a key value pair format.  

In this case, it would be kube-control label nodes node one  followed by the label in a key value format 
such as size equals large.  Now that we have labeled the node,  we can get back to creating the pod,  this time with the nodeSelector set to a size of large.  
When the pod is now created,  it is placed on node one, as desired.  NodeSelectors served our purpose, but it has limitations.

### NodeSelector Limitations

We used a single label and selector  to achieve our goal here.  But what if our requirement is much more complex?  For example, we would like to say something like  
"place the pod on a large or medium node"  or something like  "place the pod on any nodes that are not small".  
You cannot achieve this using nodeSelectors  For this, node affinity and anti-affinity features  were introduced, and we will look at that next.  
