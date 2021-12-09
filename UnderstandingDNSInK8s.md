# Understanding DNS in Kubernetes

***when a pod requests a Kubernetes service, it must first know the IP address of the service, and how dose this happen?***

👉 The answer is **DNS !!**
DNS is the foundation in K8S for service discovery. Every service defined in your cluster is assigned a DNS name.

## What is the DNS server in your cluster?
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

  




  
