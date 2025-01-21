## Cluster Setup and Hardening
1. How would you restrict access to the Kubernetes API server for specific users or groups?
2. What are PodSecurityStandards (PSS), and how do they help in securing workloads?
3. Explain how you would configure a private cluster in Kubernetes. What considerations are there for API server accessibility?
4. How do you ensure that kubelet configurations are secure across all nodes?
5. What is the purpose of the NodeRestriction admission plugin, and how does it enhance security?

## Workload Security
1. How do you enforce that only signed container images are pulled and run in a cluster?
2. What are security contexts in Kubernetes, and how can they be used to enforce least privilege for Pods?
3. How can you restrict Pods from running as root?
4. Describe how you would isolate sensitive workloads using namespaces, network policies, and RBAC.
5. What is the purpose of the allowPrivilegeEscalation flag in a security context, and how would you enforce it?

## Access and Authentication
1. How would you implement role-based access control (RBAC) to limit a user's access to certain namespaces or resources?
2. What is the difference between ServiceAccount, User, and Group in Kubernetes?
3. How would you rotate secrets, tokens, or certificates in Kubernetes to ensure secure access?
4. Explain how OIDC integration works with Kubernetes for authentication.
5. How can you audit API server requests to detect unauthorized access attempts?

## Network Security
1. What is a NetworkPolicy, and how can it be used to restrict communication between Pods?
2. How would you enforce TLS encryption for communication between microservices in a Kubernetes cluster?
3. How can you restrict external access to Kubernetes services?
4. What is the purpose of kube-proxy, and what are its security implications?
5. How would you monitor and prevent data exfiltration from a Kubernetes cluster?

## Image and Supply Chain Security
1. What tools would you use to scan container images for vulnerabilities?
2. How can you ensure that developers are only allowed to deploy images from a trusted registry?
3. Explain the concept of image pull secrets and their security best practices.
4. How do you verify the integrity of a container image before deployment?
5. What is the purpose of PodPreset, and how might it introduce a security risk?

## Runtime Security
1. How do you monitor and detect runtime threats in a Kubernetes cluster?
2. What is the purpose of Falco, and how would you integrate it into a Kubernetes cluster?
3. How would you prevent unauthorized container runtime modifications or shell access?
4. What steps would you take to secure the etcd database in a Kubernetes cluster?
5. What mechanisms can be used to enforce kernel runtime security in Pods?

## Incident Response and Compliance
1. How do you create and implement an audit policy for your Kubernetes cluster?
2. What steps would you take if you suspect a Pod has been compromised?
3. Explain how to back up and restore a Kubernetes cluster securely.
4. How do you use RoleBinding and ClusterRoleBinding to investigate potential RBAC misconfigurations?
5. What are some common compliance frameworks relevant to Kubernetes security, and how would you implement them?

## Advanced/Tricky Questions
1. What are the risks of enabling the Kubernetes HostPath volume, and how can they be mitigated?
2. How does Kubernetes handle certificate expiration for control plane components, and how would you renew them?
3. Explain the differences between the MutatingWebhookConfiguration and ValidatingWebhookConfiguration and their potential security impacts.
4. How do you mitigate risks associated with privileged containers?
5. What is the purpose of the seccomp profile, and how can you enforce a custom seccomp policy?
