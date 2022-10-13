👉 **Kubernetes (K8s) namespaces are a way to isolate, group, and organize resources within a Kubernetes cluster.**

![image](https://user-images.githubusercontent.com/33947539/195406231-7bc1f569-822d-4eab-b585-91cbd1fb13bf.png)

## When to use Kubernetes namespaces
With that in mind, let’s take a look at some common use cases for Kubernetes namespaces.

- Large teams can use namespaces to isolate their microservices. Teams can re-use the same resource names in different workspaces without conflicts. Additionally, taking action on items in one workspace never affects other workspaces.
- Organizations that use a single cluster for development, testing, and production can use namespaces to isolate environments. This practice ensures production code is not affected by changes that developers make in their own namespaces.
- Namespaces enable the use of RBAC, so teams can define roles that group lists of permissions. RBAC can ensure that only authorized users have access to resources in a given namespace.
- Users can set resource limits on namespaces by defining resource quotas. These quotas can ensure that every project has the resources it needs to run and that one namespace is not hogging all available resources.
- Namespaces can improve performance by limiting API search items. If a cluster is separated into multiple namespaces for different projects, the Kubernetes API will have fewer items to search when performing operations. As a result, teams can see performance gains within their Kubernetes clusters.

## The default namespaces in Kubernetes

Here is a breakdown of what each of those automatically created namespaces is:

**default**- used by user apps by default, until there are other custom namespaces
**kube-public**- used by public Kubernetes resources, not recommended to be used by cluster users
**kube-system**- used by Kubernetes control plane, and must not be used by cluster users

## Namespaces and Resource Quotas
You can implement Resource Quotas on a namespace so that cluster resources are allocated the way you need them to be. You can define a resource quota object on your desired namespace. For example, this manifest will create a CPU quota for the namespace demo.

![image](https://user-images.githubusercontent.com/33947539/195408321-a2e62dff-f121-4513-9f24-fa6c108dee81.png)

This means that once the quota is created, pods created and running inside the demo namespace will be limited to the above requests and limits. Once the limit is reached, no more pods can be created inside the demo namespace.

