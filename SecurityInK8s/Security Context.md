## What Is Kubernetes Security Context?
In Kubernetes, a security context defines privileges for individual pods or containers. You can use security context to grant containers or pods permissions such as the right to access an external file or run in privileged mode.

## Internal vs. External Security Contexts
Kubernetes security context is a bit complicated in the sense that some of the rules that you can define are enforced internally via Kubernetes itself, whereas others integrate with external security context tools – namely, **AppArmor and SELinux.**

Thus, you can think of Kubernetes security context as a way to define certain permissions for pods and containers, as well as to integrate Kubernetes with external security tools that run on the host rather than in Kubernetes itself.

## Security Contexts vs. RBAC
Security context is similar to, but distinct from, Kubernetes Role-Based Access Control, or RBAC. The key differences are as follows:

👉 **Resource scope**: 
RBAC can be applied to a variety of Kubernetes resources, such as pods, Kubernetes nodes, and even entire clusters. Security context assigns permissions only to pods.

👉 **Actions**:
RBAC can grant a variety of permissions based on “verbs” that admins can define within RBAC policies. Security context is more restrictive in that it only allows admins to assign specific types of predefined capabilities, like running in privileged mode (although security contexts are more flexible if you define rules using SELinux or AppArmor).

👉 **Extensibility**: 
As noted above, security contexts can be extended via integrations with external frameworks, including SELinux and AppArmor. Kubernetes RBAC can’t use external tools to define policies.

So, think of security context as a way to define additional types of security permissions for containers and pods that RBAC can’t manage. For most Kubernetes environments, you’ll want to use security context and RBAC at the same time because they complement each other.

Unlike RBAC, security context doesn’t require you to define multiple types of files (like Roles and RoleBindings) in order to enforce a security rule. You just need to add the requisite security context code when you declare a deployment, and Kubernetes will automatically enforce the rules from there.


## Security Contexts vs. Pod Security Policies

Many of the security rules that you can define using security contexts can also be configured via pod security policies, which are a different tool.

Why would Kubernetes provide support for both security contexts and pod security policies? The answer is that security contexts are essentially a replacement for pod security policies. Pod security policies, which can be used to configure permission for all pods running in a cluster, provide less granular control than security contexts, which can be applied to individual pods.


## Limitations of Security Contexts

### No Windows Support
For one, the security context tooling currently addresses only privileges and permissions that are valid on a Linux-based server. If you run Windows containers in Kubernetes, security contexts won’t be useful to you.

### Limited to Pod/Container-Level Security
Security context can only define permissions for pods or containers. It can’t control privileges at other layers of your stack.

Of course, most of the rules that you can apply with security contexts would only make sense when applied to containers or pods. It wouldn’t make sense to tell a node to run in unprivileged mode, for instance.

Still, the point here is that security context is a tool only for addressing security issues at the pod or container level. You’ll need other tools (like RBAC) to secure nodes, users, service accounts, and the like.

## Quizes

```shell
Create a Pod named secure-pod. Use redis image. Run pod as user 1000 and group 2000.
```

```yaml
cat << EOF > secure-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secure-pod
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 2000
  containers:
  - image: redis
    name: secure-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF
```
#### Explanation:

runAsUser field specifies that for any Containers in the Pod, all processes run with user ID 1000. The runAsGroup field specifies the primary group ID of 3000 for all processes within any containers of the Pod.

### Example2:

```shell
Run pod ubuntu-sleeper to run as Root user and with the SYS_TIME capability.
```

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
```

N:B 
**Capabilities can only be added per container , not per pod.**

