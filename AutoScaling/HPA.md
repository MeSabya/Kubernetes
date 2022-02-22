# Kubernetes Autoscaling

Instead of manually allocating resources, you can create automated processes that save time, let you respond quickly to peaks in demand, and conserve costs by scaling down when resources are not needed. It can be used alongside the cluster autoscaler by allocating only the resources that are needed.

The Kubernetes autoscaling mechanism uses two layers:

**Pod level**: The HPA and VPA methods take place at the pod level. Both HPA and VPA will scale the available resources or instances of the container.

**Cluster level**: The Cluster Autoscaler falls under the Cluster level, where it scales up or down the number of nodes inside your cluster.

## Horizontal Pod Autoscaler (HPA) 

HPA is a form of autoscaling that increases or decreases the number of pods in a replication controller, deployment, replica set, or stateful set based on CPU utilization—the scaling is horizontal because it affects the number of instances rather than the resources allocated to a single container.

If the load decreases, and the number of Pods is above the configured minimum, the HorizontalPodAutoscaler instructs the workload resource (the Deployment, StatefulSet, or other similar resource) to scale back down.

HPA can make scaling decisions based on custom or externally provided metrics and works automatically after initial configuration. All you need to do is define the MIN and MAX number of replicas

Once configured, the Horizontal Pod Autoscaler controller is in charge of checking the metrics and then scaling your replicas up or down accordingly. By default, HPA checks metrics every 15 seconds.

To check metrics, HPA depends on another Kubernetes resource known as the Metrics Server. The Metrics Server provides standard resource usage measurement data by capturing data from “kubernetes.summary_api” such as CPU and memory usage for nodes and pods. It can also provide access to custom metrics (that can be collected from an external source) like the number of active sessions on a load balancer indicating traffic volume.

### How does HPA work?

![image](https://user-images.githubusercontent.com/33947539/155082179-31a79830-6324-46c3-9813-34cf13de27fc.png)


In simple terms, HPA works in a “check, update, check again” style loop. Here’s how each of the steps in that loop work.

- HPA continuously monitors the metrics server for resource usage.
- Based on the collected resource usage, HPA will calculate the desired number of replicas required.
- Then, HPA decides to scale up the application to the desired number of replicas.
- Finally, HPA changes the desired number of replicas.
- Since HPA is continuously monitoring, the process repeats from Step 1.

#### Limitations of HPA

- One of HPA’s most well-known limitations is that it does not work with DaemonSets.
- If you don’t efficiently set CPU and memory limits on pods, your pods may terminate frequently or, on the other end of the spectrum, you’ll waste resources.
- If the cluster is out of capacity, HPA can’t scale up until new nodes are added to the cluster. Cluster Autoscaler (CA) can automate this process. We have an article dedicated to CA; however, below is a quick contextual explanation.

## What is Metrics Server?

- Early on, Kubernetes introduced Heapster as a tool that enables Container Cluster Monitoring and Performance Analysis for Kubernetes.
- It collects and interprets various metrics like resource usage, events, and so on. Heapster has been an integral part of Kubernetes and enabled it to schedule Pods appropriately.
- Right now, even though Heapster is still in use, it is considered deprecated, **Metrics Server is currently used now**.

- *A simple explanation is that it collects information about used resources (memory and CPU) of nodes and Pods. It does not store metrics, so do not think that you can use it to retrieve historical values and predict tendencies.*
- Metrics Server's goal is to provide an API that can be used to retrieve current resource usage.the Metrics Server collects cluster-wide metrics and allows us to retrieve them through its API.

### Why do I need the Metrics Server?
Although the Metrics Server misses all the glitter and glamour compared the rest of Monitoring tools offer with visualization and dashboard, if you use features such as Horizontal Pod AutoScaler, Vertical Pod AutoScaler, kubectl top, or Kubernetes Dashboard then you do need the Metrics Server to provide resource utilization metrics for them.

### How to view the Metrics Server?
There are two ways to capture the information in the Metrics Server.

        kubectl top command 
        Metrics endpoints /apis/metrics.k8s.io/v1beta1



### References:
https://www.kubecost.com/kubernetes-autoscaling/kubernetes-hpa/#how-does-hpa-work-3


