👉 When you type kubectl delete pod, the pod is deleted, and the endpoint controller removes its IP address and port 
(endpoint) from the Services and etcd.

You can observe this with kubectl describe service.

Listing endpoints with kubectl describe

But that’s not enough!

## How to Use Kubernetes Hooks to Track Container Lifecycles

Hooks are commonly used to log container events, implement clean-up scripts, and run asynchronous tasks after a new Pod joins your cluster. In this article, we’ll show how to attach hook handlers to your Pods and gain more control over container lifecycles.

### The Two Available Hooks
Current Kubernetes releases support two container lifecycle hooks:

- PostStart – Handlers for this hook are called immediately after a new container is created.
- PreStop – This hook’s invoked immediately before Kubernetes terminates a container.

They can be handled using two different mechanisms:

- Exec – Runs a specified command inside the container.
- HTTP – Makes an HTTP request to a URL inside the container.

Neither of the hooks provide any arguments to their handlers. Each container supports a single handler per-hook; it’s not possible to call multiple endpoints or combine an exec command with an HTTP request.



## References
https://itnext.io/how-do-you-gracefully-shut-down-pods-in-kubernetes-fb19f617cd67
https://www.datree.io/resources/kubernetes-guide-graceful-shutdown-with-lifecycle-prestop-hook

