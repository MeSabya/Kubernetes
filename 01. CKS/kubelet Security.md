## Must Read Reference:
https://www.cncf.io/blog/2021/08/20/how-to-secure-your-kubernetes-control-plane-and-node-components/


## How does kubelet authentication/authorization works?

The kubelet is a critical Kubernetes component that runs on each node in the cluster.
It is responsible for:

- Registering the node with the Kubernetes API server.
- Maintaining the Node object to reflect the node's current status.
- Managing Pods scheduled on the node by the control plane.

### Authentication with Certificates:
- Each kubelet typically authenticates with the Kubernetes API server using a client certificate.
- During the kubelet bootstrap process (e.g., using the kubeadm tool or other mechanisms), the kubelet is issued a certificate where:
- The Subject field in the certificate contains the username system:node:<nodeName> (in this case, system:node:controller-0).
- The certificate also includes the group system:nodes.

### Role and Permissions:
- Kubernetes uses this username (system:node:<nodeName>) and group (system:nodes) to determine what the kubelet is allowed to do.
- The ClusterRoleBinding binds the system:node ClusterRole (which has predefined permissions for kubelets) to the kubelet running on controller-0.

Tasks the Kubelet Performs: With this role, the kubelet can:

- Manage its own Node object in the API (e.g., update its status, advertise its health).
- Modify Pod objects that are assigned to the node (e.g., update readiness status).
- Retrieve resources it needs (e.g., ConfigMaps, Secrets) for workloads running on the node.

## How to verify that the kubelet on a node (e.g., controller-0) is authenticating as the user system:node:controller-0 and is part of the system:nodes group?

### Inspect the Kubelet Client Certificate
Kubelets authenticate with the Kubernetes API server using client certificates. You can check the certificate being used by the kubelet:

On the Node (e.g., controller-0):
Locate the Certificate: The kubelet's client certificate is usually stored under /var/lib/kubelet/pki/. The file to inspect is typically kubelet-client-current.pem.

```bash
sudo openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout
```

Check the Certificate Subject: Look for the Subject field in the certificate. It should show:

```perl
Subject: O=system:nodes, CN=system:node:controller-0
O=system:nodes: This indicates the kubelet belongs to the system:nodes group.
CN=system:node:controller-0: This indicates the kubelet's username.
```

### View the system:node Role and Binding
Verify that the system:node ClusterRole grants the expected permissions and is correctly bound to the system:node:controller-0 user.

Check the system:node ClusterRole:

```bash
kubectl get clusterrole system:node -o yaml
```

Check the system:node ClusterRoleBinding:

```bash
kubectl get clusterrolebinding system:node -o yaml
```
Ensure the subjects field includes the user system:node:controller-0 or the group system:nodes.

## How do you ensure that kubelet configurations are secure across all nodes

### How to Locate the Kubelet Configuration File, is it inside /etc/kubernetes/manifets where other manifests reside? How to find it? 

#### Option1: Check the Kubelet Service File:
- The kubelet service file usually specifies the configuration file path via the --config flag.
- Location: /etc/systemd/system/kubelet.service.d/10-kubeadm.conf.

Example:
ExecStart=/usr/bin/kubelet --config=/var/lib/kubelet/config.yaml

#### Option2: Command to Check Kubelet Options: Run the following command to verify how the kubelet is started:

```bash
ps -ef | grep kubelet | grep config
```

### How to secure Kubelet 
- Set --anonymous-auth argument to false to disallow unauthenticated requests.
- Set --rotate-certificates to true. If you’re using a kubelet configuration file, set RotateKubeletServerCertificate to true.
- If you aren’t doing so already, ensure that your kubelets get their certificates from the API Server.
  Set --tls-cert-file and --tls-private-key-file appropriately. If you’re using a kubelet configuration file, set tlsCertFile and tlsPrivateKeyFile appropriately
  so that all connections on the kubelets happen over TLS.
- Use the NodeRestriction admission plugin to limit what kubelets can access and modify.
  Set --authorization-mode=Webhook and --authentication-token-webhook=true to ensure kubelets authenticate and are authorized securely by the API server.

## What is the purpose of the NodeRestriction admission plugin (default admission plugin in api-server  onfig file), and how does it enhance security?

The NodeRestriction admission plugin in Kubernetes enhances cluster security by ensuring that nodes (and the associated kubelet components) can only 
modify or access resources that they are explicitly authorized to interact with. Here's a detailed explanation of its purpose and how it enhances security:

### Purpose of the NodeRestriction Admission Plugin
The NodeRestriction admission plugin enforces the following key rules:

- Nodes can only modify their own Node objects (e.g., updating status or labels).
- Prevents nodes from modifying other nodes' metadata or status, which could lead to privilege escalation.
- Nodes are restricted to viewing or modifying Pod objects scheduled to run on them.
- Ensures that nodes do not access Pods running on other nodes, preserving data confidentiality and integrity.
- Nodes can only access Secret and ConfigMap objects mounted as volumes in the Pods scheduled to run on them.
