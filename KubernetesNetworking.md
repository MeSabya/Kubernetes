# The Kubernetes Networking Model

Kubernetes dictates the following requirements on any networking implementation:

- all Pods can communicate with all other Pods without using network address translation (NAT).
- all Nodes can communicate with all Pods without NAT.
- the IP that a Pod sees itself as is the same IP that others see it as.

Given these constraints, we are left with four distinct networking problems to solve:

👉 Container-to-Container networking

👉 Pod-to-Pod networking

👉 Pod-to-Service networking

👉 Internet-to-Service networking

## Container-to-Container Networking

## References:
https://sookocheff.com/post/kubernetes/understanding-kubernetes-networking-model/

