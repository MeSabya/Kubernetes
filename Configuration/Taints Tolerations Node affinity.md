## Let us start with a simple cluster with three worker nodes.

The nodes are named one, two, and three.  We also have a set of pods  that are to be deployed on these nodes.  
Let's call them A, B, C, and D.  

When the pods are created  Kubernetes scheduler tries to place these pods  on the available worker nodes. 
As of now, there are no restrictions or limitations  and as such, the scheduler places the pods  across all of the nodes to balance them out equally.  

### Problem Statement
Now, let us assume that we have dedicated resources  on node one for a particular use case, or application.  
So we would like only those pods  that belong to this application  to be placed on node one.  

First, we prevent all pods from being placed on the node  by placing a taint on the node.  Let's call it blue.  
By default, pods have no tolerations,  which means unless specified otherwise  none of the pods can tolerate any taint.  So in this case, 
none of the pods can be placed on node one,  as none of them can tolerate the taint blue.  This solves half of our requirement.  

No unwanted pods are going to be placed on this node.  The other half is to enable certain pods  to be placed on this node.  
For this, we must specify which pods are tolerant  to this particular taint.  In our case,  we would like to allow only pod D to be placed on this node.  
So we add a toleration to pod D.  Pod D is now tolerant to blue.  So when the scheduler tries to place this pod on node one,  
it goes through.  Node one can now only accept pods  that can tolerate the taint blue.  So with all the taints and tolerations in place  
this is how the pods would be scheduled.  The scheduler tries to place pod A on node one,  but due to the taint it is thrown off  and it goes to node two.  
The scheduler then tries to place pod B on node one,  but again, due to the taint it is thrown off  and is placed on node three  which happens to be the next
free node.  The scheduler then tries to place pod C to the node one.  It is thrown off again and ends up on node two.  And finally, 
the scheduler tries to place pod D on node one.  Since the pod is tolerant to node one, it goes through.  

### Important points 
👉 **So remember, taints are set on nodes  and tolerations are set on pods.  

👉 **So remember, taints and tolerations does not tell the pod to go to a particular node.Instead, it tells the node to only accept pods
with certain tolerations.**

let us also take a look at an interesting fact.  
So far we have only been referring to the worker nodes  but we also have master nodes in the cluster,  which is technically just another node  that has all the capabilities of hosting a pod,  plus it runs all the management software.  Now, I'm not sure if you noticed  the scheduler does not schedule any pods on the master node.  Why is that?  When the Kubernetes cluster is first set up,  a taint is set on the master node automatically  that prevents any pods from being scheduled on this node.  You can see this,  as well as modifies this behavior if required.  However, a best practice is to not deploy  application workloads on a master server.  To see this taint run a cube control  describe node command with Q master as the node name  and look for the taint section.  You will see a taint set to not schedule any pods  on the master node.  Well, that's it for this lecture.  Head over to the coding exercises section  and practice working with taints and tolerations.  information
