# Understanding DNS in Kubernetes

***when a pod requests a Kubernetes service, it must first know the IP address of the service, and how dose this happen?***

👉 The answer is **DNS !!**
DNS is the foundation in K8S for service discovery. Every service defined in your cluster is assigned a DNS name.

## What is the DNS server in your cluster (till k8s 1.11 kube-dns is the default DNS, kube-dns is replaced by core-dns ?
- kube-dns is a built-in DNS service automatically launched in your cluster.
- kube-dns runs as a deployment that scheduling kube-dns pods to nodes.
- kube-dns is typically exposed as a native K8S service and the service type is ClusterIP, which means it is assigned a virtual IP address.
- List the kube-dns deployment in your cluster:
   >kubectl get deployment -l k8s-app=kube-dns -nkube-system

The output is something like:

```yaml
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
...
kube-dns   2/2     2            2           ...
```

List the kube-dns service:

```yaml
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kube-dns   ClusterIP   10.0.32.10   <none>        53/UDP,53/TCP
```

## How does my pod find the DNS server?
Kubelet on the node automatically configures the DNS server in the pod’s /etc/resolv.conf with the — cluster-dns=<kube-dns-virtual-ip>
You can execute nslookup in your pod to print the DNS server information:
  
```yaml  
kubectl exec -it <Pod-Name> -- nslookup kubernetes.default 
  
Server:    10.0.32.10
Address 1: 10.0.32.10
Name:      kubernetes.default
Address 1: 10.0.32.1  
```

## What happens when my pod requests a service?
- When a pod requests a service, it must first lookup for the virtual IP of the service using DNS queries.
- Every service is assigned a DNS name, in the form of “servicename.namespace.svc.cluster-domain.example”. 
  For example, if you create a service “foo” in the default namespace, it will be assigned the following DNS name.
  ```yaml
  foo.default.svc.cluster.local
  ```
✳️ The DNS queries from your pod that don’t specify a namespace will use the pod’s namespace.
For example, consider your pod in “test” namespace. A foo service in the prod namespace.
A query for “foo” is implicitly expanded to “foo.test.svc.cluster.local”. The DNS query returns no results, because there is no foo service in test namespace *
  
## Then how can I get the IP address of a service outside of my pod’s namespace?
Simple, you just need to explicitly specify the namespace in the DNS query.
Using the above example, the query should be “foo.prod.svc.cluster.local”
  
## Can I customize the DNS service in my cluster?
You can modify the ConfigMap for kube-dns to set stub domains (aka DNS private zone) and/or add Upstream NameServers.
The following example ConfigMap manifest for kube-dns includes a stub domain and an upstream server:
  
```yaml  
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-dns
  namespace: kube-system
data:
  stubDomains: |
    {“example.com”: [“1.2.3.4”]}
  upstreamNameservers: |
    [“8.8.4.4”]
```
All DNS queries with the “.example.com” suffix will be forwarded to the server “1.2.3.4”.
All other queries not ending in “.cluster.local” or “.example.com”, will be forwarded to the external DNS server at “8.8.4.4”.  

## Can you give me an example of how core DNS is used in k8s service and pod discovery. 

Certainly! In Kubernetes, CoreDNS is used for DNS-based service and pod discovery within the cluster. Here's a simple example to illustrate how CoreDNS works for service and pod discovery:

### Step1: Create a Deployment and a Service:

Let's create a simple Nginx deployment and expose it using a Service.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

### Step2: Apply these YAML files to create the deployment and service:

```bash
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```

### Step3: Verify CoreDNS Configuration:

By default, CoreDNS in a Kubernetes cluster is configured to handle DNS resolution for services and pods. You don't need to explicitly configure CoreDNS for basic service discovery.

### Step4: Service Discovery:

You can now use DNS to discover the Nginx service. The service DNS format is <service-name>.<namespace>.svc.cluster.local.

```bash
nslookup nginx-service.default.svc.cluster.local
```

The output should show the ClusterIP address of the nginx-service. This is how DNS is used to resolve the service name to its IP address.

### Step5: Pod Discovery:

Each pod in Kubernetes gets a DNS subdomain based on its name and namespace. The format is <pod-name>.<namespace>.pod.cluster.local.

Let's find the DNS name for one of the Nginx pods:

```bash
kubectl get pods -l app=nginx
```

Replace <pod-name> and <namespace> with the actual pod name and namespace from the output.

```bash
nslookup <pod-name>.<namespace>.pod.cluster.local
```

The output should show the IP address of the Nginx pod. This demonstrates how DNS is used to resolve the pod name to its IP address.

In this example, CoreDNS automatically handles the DNS resolution for the Nginx service and pods. Kubernetes applications and services can use these DNS names to communicate with each other, making service discovery simpler and more abstracted from specific IP addresses.

## Configuration for CoreDNS in a Kubernetes cluster
The configuration for CoreDNS in a Kubernetes cluster is typically managed using ConfigMaps. The ConfigMap contains the CoreDNS configuration file, and you can customize it based on your requirements. The configuration file is named Corefile.

In a standard Kubernetes installation, you can find the ConfigMap for CoreDNS in the kube-system namespace. The ConfigMap's name is usually coredns. You can retrieve it using the following command:

```bash
kubectl get configmap coredns -n kube-system -o yaml
```

  
