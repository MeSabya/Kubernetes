## Cluster Setup and Hardening
1. How would you restrict access to the Kubernetes API server for specific users or groups?
2. What are PodSecurityStandards (PSS), and how do they help in securing workloads?
3. Explain how you would configure a private cluster in Kubernetes. What considerations are there for API server accessibility?
4. How do you ensure that kubelet configurations are secure across all nodes?
5. What is the purpose of the NodeRestriction admission plugin, and how does it enhance security?

## Workload Security
How do you enforce that only signed container images are pulled and run in a cluster?
What are security contexts in Kubernetes, and how can they be used to enforce least privilege for Pods?
How can you restrict Pods from running as root?
Describe how you would isolate sensitive workloads using namespaces, network policies, and RBAC.
What is the purpose of the allowPrivilegeEscalation flag in a security context, and how would you enforce it?

## Access and Authentication
How would you implement role-based access control (RBAC) to limit a user's access to certain namespaces or resources?
What is the difference between ServiceAccount, User, and Group in Kubernetes?
How would you rotate secrets, tokens, or certificates in Kubernetes to ensure secure access?
Explain how OIDC integration works with Kubernetes for authentication.
How can you audit API server requests to detect unauthorized access attempts?

## Network Security
What is a NetworkPolicy, and how can it be used to restrict communication between Pods?
How would you enforce TLS encryption for communication between microservices in a Kubernetes cluster?
How can you restrict external access to Kubernetes services?
What is the purpose of kube-proxy, and what are its security implications?
How would you monitor and prevent data exfiltration from a Kubernetes cluster?

## Image and Supply Chain Security
What tools would you use to scan container images for vulnerabilities?
How can you ensure that developers are only allowed to deploy images from a trusted registry?
Explain the concept of image pull secrets and their security best practices.
How do you verify the integrity of a container image before deployment?
What is the purpose of PodPreset, and how might it introduce a security risk?

## Runtime Security
How do you monitor and detect runtime threats in a Kubernetes cluster?
What is the purpose of Falco, and how would you integrate it into a Kubernetes cluster?
How would you prevent unauthorized container runtime modifications or shell access?
What steps would you take to secure the etcd database in a Kubernetes cluster?
What mechanisms can be used to enforce kernel runtime security in Pods?

## Incident Response and Compliance
How do you create and implement an audit policy for your Kubernetes cluster?
What steps would you take if you suspect a Pod has been compromised?
Explain how to back up and restore a Kubernetes cluster securely.
How do you use RoleBinding and ClusterRoleBinding to investigate potential RBAC misconfigurations?
What are some common compliance frameworks relevant to Kubernetes security, and how would you implement them?

## Advanced/Tricky Questions
What are the risks of enabling the Kubernetes HostPath volume, and how can they be mitigated?
How does Kubernetes handle certificate expiration for control plane components, and how would you renew them?
Explain the differences between the MutatingWebhookConfiguration and ValidatingWebhookConfiguration and their potential security impacts.
How do you mitigate risks associated with privileged containers?
What is the purpose of the seccomp profile, and how can you enforce a custom seccomp policy?
