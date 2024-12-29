## Must Read Reference:
https://www.cncf.io/blog/2021/08/20/how-to-secure-your-kubernetes-control-plane-and-node-components/

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

## What is the purpose of the NodeRestriction admission plugin, and how does it enhance security?

The NodeRestriction admission plugin in Kubernetes enhances cluster security by ensuring that nodes (and the associated kubelet components) can only 
modify or access resources that they are explicitly authorized to interact with. Here's a detailed explanation of its purpose and how it enhances security:

### Purpose of the NodeRestriction Admission Plugin
The NodeRestriction admission plugin enforces the following key rules:

- Nodes can only modify their own Node objects (e.g., updating status or labels).
- Prevents nodes from modifying other nodes' metadata or status, which could lead to privilege escalation.
- Nodes are restricted to viewing or modifying Pod objects scheduled to run on them.
- Ensures that nodes do not access Pods running on other nodes, preserving data confidentiality and integrity.
- Nodes can only access Secret and ConfigMap objects mounted as volumes in the Pods scheduled to run on them.
