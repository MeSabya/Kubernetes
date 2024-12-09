## You are an administrator of a Kubernetes cluster running a couple of existing Pods. It's your job to inspect the containers defined by the Pods for immutability. Delete all Pods that do not follow typical immutability best practices.

A container can be considered mutable if it exhibits characteristics that allow it to change during its lifecycle.
***Below characteristics will make a container mutable***

### Writable Filesystem:

- Filesystem is not read-only (readOnlyRootFilesystem: false or omitted)
- Applications write to internal container paths instead of external storage.

### Ephemeral Runtime Configuration:
Runtime configurations are hardcoded or modified within the container instead of being injected via environment variables, ConfigMaps, or Secrets.

### Writable Volume Mounts:
Volume mounts (e.g., /tmp, /var) are writable instead of being explicitly read-only.

### hostNetwork: true:
Container shares the host’s network namespace, exposing it to potential security risks and host-level mutability.

### securityContext.runAsUser: 0 (Root User):
The container runs as the root user, increasing the risk of privilege escalation and unintended modifications.

### securityContext.privileged: true:
Privileged containers have full access to the host, allowing modifications to the host system.

### Absence of readOnlyRootFilesystem:
The root filesystem is writable, enabling modifications at runtime.

### Capabilities Added:
Elevated capabilities (e.g., SYS_ADMIN) are added, granting the container more privileges than necessary.

### Host Path Mounts:
The container mounts host paths directly, allowing modification of host files or access to sensitive directories.

## Your cluster is configured to allow anonymous API requests.Update the cluster configuration to disable anonymous access.
Locate the Kubernetes API Server Configuration:

Configuration is typically managed via flags in the API server manifest file, often found at /etc/kubernetes/manifests/kube-apiserver.yaml.

1. Modify the API Server Configuration:

Locate the --anonymous-auth flag.
Set it to false to disable anonymous access.

```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --anonymous-auth=false
```

Restart the API Server:
If running in static pods, the kubelet automatically restarts the API server pod after changes to its manifest.

## Write a NetworkPolicy that denies all incoming and outgoing traffic to Pods in the dev namespace, except for traffic from the monitoring namespace.

- It denies all incoming and outgoing traffic for Pods in the dev namespace.
- It allows incoming traffic only from Pods in the monitoring namespace.

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-dev-traffic
  namespace: dev
spec:
  podSelector: {} # Applies to all Pods in the 'dev' namespace
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring # Allow traffic only from the 'monitoring' namespace
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: monitoring # Allow traffic only to the 'monitoring' namespace
```

### Key Points:
#### Namespace Labeling:

Ensure the monitoring namespace has the label name=monitoring. You can add it with:

``` sh
kubectl label namespace monitoring name=monitoring
```
#### PodSelector:
The podSelector: {} applies the policy to all Pods in the dev namespace.

#### Ingress and Egress Rules:
The from and to rules allow traffic to and from the monitoring namespace while blocking all other traffic.

#### Deny by Default:
Kubernetes NetworkPolicy applies a default "deny all" behavior for Pods covered by the policy when no explicit rules match the traffic.

## One of the Pods in your cluster has an outdated image with known vulnerabilities. How can you automate image vulnerability scanning in Kubernetes?

- Trivy: An open-source scanner for container images.
- Portieris is a Kubernetes admission controller for the enforcement of image security policies. You can create image security policies for each Kubernetes namespace or at the cluster level, and enforce different rules for different images.

Trivy combined with Portieris is a robust solution for automating image vulnerability scanning and enforcement in your Kubernetes cluster. Here’s how they complement each other:

- Trivy scans images for vulnerabilities during the build and deployment process (either in CI/CD or as part of the Kubernetes deployment).
- Portieris enforces security policies that ensure only trusted, scanned images are deployed, blocking vulnerable images from running in the cluster.

Together, they provide:

- Pre-deployment image scanning (with Trivy).
- Runtime policy enforcement (with Portieris).
- This combination is a comprehensive solution for securing your Kubernetes deployments and preventing the deployment of vulnerable container images.

## Inspect the audit logs of your cluster to identify unauthorized access attempts. What tools or configurations can help monitor and prevent such activities?

### Enable Kubernetes Audit Logging
https://github.com/bmuschko/cks-crash-course/blob/master/exercises/12-audit-logging/solution/solution.md

### KubeAudit
KubeAudit is a tool for auditing your Kubernetes cluster's security posture. It performs a security audit of your cluster's configuration and helps identify misconfigurations that could allow unauthorized access.

### Falco
Falco is an open-source tool for detecting unexpected behavior in your Kubernetes cluster. It monitors the kernel and container activity and can detect unauthorized access and malicious activities by alerting you in real-time.

### Kubernetes RBAC (Role-Based Access Control)
RBAC helps in controlling access to the Kubernetes API based on the roles and permissions granted to users and service accounts. Review and configure your RBAC policies to limit unauthorized access attempts.

Inspect Current RBAC Configurations:

kubectl get roles,rolebindings --all-namespaces
kubectl get clusterroles,clusterrolebindings
Best Practices:

Follow the principle of least privilege (POLP), ensuring users and service accounts have only the minimum required permissions.
Use kubectl auth can-i to check what actions a user can perform.

## Some Pods in your cluster are using default service accounts. Explain why this is risky and configure unique, least-privileged service accounts for these Pods.

The default service account typically has broad permissions to access resources within the cluster. If a Pod uses this account, any vulnerability in that Pod could potentially be exploited to access sensitive data or perform unauthorized actions on the cluster. This increases the risk of privilege escalation attacks.

If the default service account is used by a compromised Pod, it might be able to access the Kubernetes API server and interact with other resources, including secrets, ConfigMaps, and other critical infrastructure components.


### How to Configure Unique, Least-Privileged Service Accounts
To mitigate these risks, configure unique and least-privileged service accounts for Pods. Here’s how:

#### Step 1: Create a Custom Service Account
Create a new service account for your Pod with limited permissions.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-custom-service-account
```

#### Step 2: Define Roles and RoleBindings
Create a Role or ClusterRole to define the specific permissions required by the Pod, ensuring it follows the least-privilege principle.

Role Definition Example (in a namespace):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-namespace
  name: my-custom-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
RoleBinding Example (binding the role to the service account):
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-custom-rolebinding
  namespace: my-namespace
subjects:
- kind: ServiceAccount
  name: my-custom-service-account
  namespace: my-namespace
roleRef:
  kind: Role
  name: my-custom-role
  apiGroup: rbac.authorization.k8s.io
```

#### Step 3: Assign the Service Account to Your Pod

Modify the Pod specification to use the newly created service account instead of the default one.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  serviceAccountName: my-custom-service-account
  containers:
  - name: my-app
    image: my-app-image
```

## Detect and fix any containers that are exposing privileged host ports (ports below 1024). Why should such ports be avoided in containerized applications?

Privileged host ports, i.e., ports below 1024, are often called well-known ports. These are reserved by the Internet Assigned Numbers Authority (IANA) for specific services such as:

- Port 22 for SSH
- Port 80 for HTTP
- Port 443 for HTTPS

Exposing containers to these privileged ports on the host system introduces several risks:

Port Conflicts: These ports are often in use by critical services. Using them in containers may cause conflicts with existing services on the host.

Escalation of Privileges: On many systems, only privileged users (root or users with elevated privileges) can bind to ports below 1024. Running containers with elevated privileges to bind to such ports can open up the system to privilege escalation vulnerabilities.

### Detect Containers Exposing Privileged Ports (Below 1024)

```bash
kubectl get pods --all-namespaces -o json | jq -r '.items[] | select(.spec.containers[].ports[]?.hostPort < 1024) | .metadata.namespace + "/" + .metadata.name'
```



## Identify Pods Without Resource Limits and risks of not defining resource limits
### Identify Pods Without Resource Limits
To identify Pods that don't specify resources.limits for CPU and memory, you can use kubectl to inspect the resource configurations of the Pods. You can use the following command to find Pods missing resource limits:

```bash
kubectl get pods --all-namespaces -o=jsonpath='{range .items[?(!@.spec.containers[0].resources.limits)]}{.metadata.namespace}/{.metadata.name}{"\n"}{end}'
```

This command looks through all namespaces and lists Pods that don't have resource limits set for CPU or memory.

### Risks of Missing Resource Limits
The absence of resource limits can lead to several issues:

- Resource Contention: If Pods don't have limits, they can potentially consume more resources than necessary, starving other Pods of CPU or memory.
- Node Instability: Pods without limits can lead to excessive resource consumption, causing nodes to run out of resources. This can trigger the node to kill Pods, potentially affecting your application availability.
- Unexpected Failures: Pods without limits might cause unpredictable behavior in your application due to the lack of resource boundaries.
- Difficult Debugging: Resource exhaustion might not be immediately apparent, making it hard to troubleshoot issues.

