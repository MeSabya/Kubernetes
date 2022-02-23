Kubernetes has one of the most sophisticated schedulers that handles the pod placement strategy. Based on the resource requests mentioned in the pod spec, Kubernetes scheduler automatically chooses the most appropriate node to run the pod.

But there are scenarios where we may have to intervene in the scheduling process to enable matchmaking between the pod and a node or two specific pods. Kubernetes offers a powerful mechanism to take control of the pod placement logic.

Let’s explore the key techniques that influence the default scheduling decisions in Kubernetes.

## Node Affinity/Anti-Affinity

Below are the examples of using node affinity/anti-affinity with hard and soft rules.
👉 *The expressions, **requiredDuringSchedulingIgnoredDuringExecution and preferredDuringSchedulingIgnoredDuringExecution** enforce the hard and soft rules respectively.*

```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
          - key: "failure-domain.beta.kubernetes.io/zone"
            operator: In
            values: ["asia-south1-a"]
```

The above rule will instruct Kubernetes scheduler to try and place the pod on a node running in the “asia-south1-a” zone of a GKE cluster. If there are no nodes available, the scheduler is free to apply the standard placement logic.

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
          - key: "failure-domain.beta.kubernetes.io/zone"
            operator: NotIn
            values: ["asia-south1-a"]
 ```
 
 The above rule imposes anti-affinity by using the NotIn operator. This is a hard rule which ensures that no pod is placed in a GKE node running in the asia-south1-a zone.
 
 ## Pod Affinity/Anti-Affinity
 
 - There are scenarios where we need to ensure that pods are co-located or no two pods are running on the same node. Pod affinity/anti-affinity helps us in applying rules that enforce granular placement logic.
 - Similar to the expressions in node affinity/anti-affinity, pod affinity/anti-affinity can impose and hard and soft rules through requiredDuringSchedulingIgnoredDuringExecution and preferredDuringSchedulingIgnoredDuringExecution. It is also possible to mix and match node affinity with pod affinity to define complex placement logic.

We start by deploying cache with an anti-affinity rule that prevents more than one pod running on a node.

```yaml
affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - redis
            topologyKey: "kubernetes.io/hostname"
```

### Reference:

https://medium.com/the-programmer/working-with-node-affinity-in-kubernetes-40bc79d16f2f
https://thenewstack.io/implement-node-and-pod-affinity-anti-affinity-in-kubernetes-a-practical-example/
 
https://www.magalix.com/blog/influencing-kubernetes-scheduler-decisions

 
