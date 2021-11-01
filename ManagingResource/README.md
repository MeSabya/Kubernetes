## Understanding CPU Resources
- CPU resources are measured in cpu units. The exact meaning of a cpu unit depends on where we host our cluster. If servers are virtualized, one cpu unit is equivalent to one virtualized processor (vCPU). When running on bare-metal with Hyperthreading, one cpu equals one Hyperthread. For the sake of simplification, 
we’ll assume that one cpu resource is one CPU processor (even though that is not entirely true).

- CPU values can be fractioned. In our example, the db container has the CPU requests set to 0.5 which is equivalent to half CPU. The same value could be expressed as 500m,
which translates to five hundred millicpu. If you take another look at the CPU specs of the api container, you’ll see that its CPU limit is set to 200m and the requests to 100m. They are equivalent to 0.2 and 0.1 CPUs.

- So 500m which is nothing but five hunder mili cpu which also 500/1000 = .5cpu

## Understanding Memory Resources
Memory can be expressed as:

K (kilobyte)
M (Megabyte)
G (Gigabyte)
T (Terabyte)
P (Petabyte)
E (Exabyte).
We can also use the power-of-two equivalents Ki, Mi, Gi, Ti, Pi, and Ei.

## Limits and Requests
### Limits
- A limit represents the amount of resources that a container should not pass. The assumption is that we define limits as upper boundaries which, when reached, indicate that something went wrong, as well as a way to guard our resources from being overtaken by a single rogue container due to memory leaks or similar problems.

- If a container is restartable, Kubernetes will restart a container that exceeds its memory limit. Otherwise, it might terminate it. 
Bear in mind that a terminated container will be recreated if it belongs to a Pod (as all Kubernetes-controlled containers do).

- Unlike memory, CPU limits never result in termination or restarts. Instead, a container will not be allowed to consume more than the CPU limit for an extended period.
- Memory limits control the maximum amount of memory that your container may use independent of contention on the node . If you specify a memory limit, you can constrain the  
  amount of memory the container can use. 
- For example, if you specify a limit of 200Mi, a container will be limited to using that amount of memory on the node. 
  If the container exceeds the specified memory limit, it will be terminated and potentially restarted dependent upon the container restart policy.

To understand the effect of memory limit, let's try two examples : 

- **A memory leak on a container without memory resources limits**
- **A memory leak  on a container with memory resources limits**

### Create container without resources limits
![image](https://user-images.githubusercontent.com/33947539/139714405-5121083d-2e77-4d04-a7aa-d3648a9a9e04.png)

### Create container with resources limits

![image](https://user-images.githubusercontent.com/33947539/139714523-95eece50-f41f-4133-87c3-c4ad9a8e30a9.png)

***There is different behaviour when setting limits and without limits, The general behaviour is that kernel tries to kill the new stress processes on the container. Without limits, these process are requesting the total memory so the kernel protects the system  by killing the container. With memory limits, stress processes are requesting memory up to 256M and no more, kernel detects that the cgroup is out of memory.***

### cpu request exceeding available resources
![image](https://user-images.githubusercontent.com/33947539/139715549-1b7c4a4c-a686-46c3-b3ed-59646a0d473e.png)

![image](https://user-images.githubusercontent.com/33947539/139715659-082c932d-4d52-45c1-8c5a-e86bbb17f113.png)
