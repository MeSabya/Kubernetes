# Graceful shutdown in Kubernetes

## What is the problem we are trying to solve here ?

- *If we don’t stop our application correctly, we are wasting resources like DB connections and we may also break ongoing requests. An HTTP request doesn’t recover automatically — if we fail to serve it, then we simply missed it and it can lead to unexpected ELB 5XXs on our load balancers.*
- It’s important that your application handle termination gracefully so that there is minimal impact on the end user and the time-to-recovery is as fast as possible!
- In practice, this means your application needs to handle the SIGTERM message and begin shutting down when it receives it. This means saving all data that needs to be saved, closing down network connections, finishing any work that is left, and other similar tasks.
- 

## Understand with an example

An increased number of sporadic HTTP failures with response code 502 — Bad Gateway when communicating with a legacy service "svcA" from one of the microservices "svcB".
And the "svcB" microservice is not implemented with exponential backoff retry stratergy and the usecase failed.

**We found that:**

          Requests were failing during a deployment or when HorizontalPodAutoscaler auto-scales pods.
          
          During that time the pods were in Terminatingor Terminated state.

>This made us suspect that the pods are not gracefully terminated.

## Theory: Pod deletion in Kubernetes:
When a pod is deleted, Kubernetes has to:

        Remove the IP and port of the corresponding pod from the Endpointobject,
        Notify kubelet which terminates the containers associated with the pod,
        Notify kube-proxy which removes the IP and port of the corresponding pod from the iptables.

![image](https://user-images.githubusercontent.com/33947539/155293433-40a45d25-8b4d-4c8d-8891-0cdccbc5d9ce.png)

- When the API Server receives the request to delete the pod, it updates the state of the pod to Terminating. 
- The kubelet is notified of this event, which in turn sends SIGTERM to the corresponding containers associated with the pod.
- When the Endpoint object is updated in the control plane, the event is published to kube-proxy, Ingress Controller, DNS and Service Mesh.
- These components remove the pod IP and port from their internal state and stop routing traffic to the pod. 
- *Since each of the individual components is independent and each component might be busy executing other tasks, there is no guarantee on how long it will take to remove the pod IP and port and stop routing traffic to the pod.*

👉 The processes of removing the pod from the Endpoint object and sending SIGTERM to containers occur concurrently rather than sequentially. If the pod is terminated after the pod IP is removed from kube-proxy, we don’t have a problem.

   ![image](https://user-images.githubusercontent.com/33947539/155294888-0a965e6c-0e6b-478b-a017-d5ce45fa2c3f.png)

👉 However, if the pod is terminated before the pod IP is removed, the terminated pod would still receive traffic during that short period of time. As a result, the clients calling the terminated service would get a 502 error.

![image](https://user-images.githubusercontent.com/33947539/155295010-d8dd99b9-9cc6-4640-b217-d0d400aab6e6.png)


### Solution to the above problem:
Because this behavior is highly inconsistent and depends on how busy the kube-proxy or Ingress Controller are, it has to be handled properly. 
Kubernetes provides **Container Lifecycle Hooks** which are executed by kubelet on the corresponding containers when it receives the events. 

👉 *Before the kubelet sends SIGTERM to the container, it executes preStop lifecycle hook. The hook is blocking and synchronous, so only after the command in the hook is executed does the container receive SIGTERM*. 

The total duration of **preStop hook** should not be more than the **terminationGracePeriodSeconds** configured for the pod, which is 30 seconds by default. 
If the pod takes more than 30 seconds to terminate, a higher value for terminationGracePeriodSeconds should be set in the pod definition YAML.

![image](https://user-images.githubusercontent.com/33947539/155296072-d4881f48-6258-4918-9ac7-612912b48040.png)

👉 If the preStop hook adds a delay before terminating the container, we get a grace period till the pod IP is removed from the Endpoint object which in turn updates kube-proxy, Ingress Controller, DNS, etc.

### Example to the above statement:
The pod of the PHP service is composed of 2 containers: nginx which is the reverse-proxy web server and php-fpm which is responsible for handling the request.

![image](https://user-images.githubusercontent.com/33947539/155296525-06d010b4-fbd2-4e8e-8f11-84fdf48ce062.png)

K8 pod yaml file for the above:

![image](https://user-images.githubusercontent.com/33947539/155296835-4a50cf92-5e04-489b-9d91-9ce13eb3b6df.png)


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: php-pod
spec:
  containers:
    - name: fpm
      image: php-fpm
      ports:
        - name: web
          containerPort: 80
      lifecycle:
        preStop:
          exec:
            command:
              - sh
              - '-c'
              - sleep 5 && kill -SIGQUIT 1
    - name: nginx
      image: nginx
      ports:
        - name: http
          containerPort: 80
      lifecycle:
        preStop:
          exec:
            command:
              - sh
              - '-c'
              - sleep 5 && /usr/sbin/nginx -s quit
  terminationGracePeriodSeconds: 30
```

☝️ Setting a value for php-fpm lower than nginx means that when the pod receives traffic after the php-fpm container is terminated and nginx container is still within the sleep duration, the request would fail with a 5xx error. Setting a sleep value for the php-fpm container higher than nginx is safer and ensures that no requests will fail.



