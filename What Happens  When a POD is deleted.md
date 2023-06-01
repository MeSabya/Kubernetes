👉 When you type kubectl delete pod, the pod is deleted, and the endpoint controller removes its IP address and port 
(endpoint) from the Services and etcd.

You can observe this with kubectl describe service.

Listing endpoints with kubectl describe



But that’s not enough!

## References
https://itnext.io/how-do-you-gracefully-shut-down-pods-in-kubernetes-fb19f617cd67
