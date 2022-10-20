There are five essential things to understand about networking in Kubernetes

- Communication between containers in the same pod
- Communication between pods on the same node
- Communication between pods on different nodes
- Communication between pods and services
- How does DNS work? How do we discover IP addresses?

## Communication between containers in the same pod
First, if you've got two containers running in the same pod, how do they talk to each other?

This happens via localhost and port numbers. Just like when you’re running multiple servers on your own laptop.

This is possible because containers in the same pod are in the same network namespace – they share networking resources.

### What is a network namespace?
It’s a collection of network interfaces (connections between two pieces of equipment on a network) and routing tables (instructions for where to send network packets).

Namespaces are helpful because you can have many network namespaces on the same virtual machine without collisions or interference.

(You wouldn’t want all your pods to run containers that listen on port 3000 in the same namespace – they’d all collide!)

## Communication between pods on the same node

- Each pod on a node has its own network namespace. Each pod has its own IP address.
- Each pod’s eth0 device is actually connected to a virtual ethernet device in the node.

👉 A virtual ethernet device is a tunnel that connects the pod’s network with the node. This connection has two sides – on the pod’s side, it’s named eth0, and on the node’s side, it’s named vethX.

Why the X? There’s a vethX connection for every pod on the node. (So they’d be veth1, veth2, veth3, etc.)

When a pod makes a request to the IP address of another node, it makes that request through its own eth0 interface. This tunnels to the node’s respective virtual vethX interface.

👉 ***But then how does the request get to the other pod?***

The node uses a **network bridge.**

### What is a Network Bridge?
A network bridge connects two networks together. When a request hits the bridge, the bridge asks all the connected devices (i.e. pods) if they have the right IP address to handle the original request.

(Remember that each pod has its own IP address and it knows its own IP address.)

If one of the devices does, the bridge will store this information and also forward data to the original back so that its network request is completed.

In Kubernetes, this bridge is called cbr0. Every pod on a node is part of the bridge, and the bridge connects all pods on the same node together.

![image](https://user-images.githubusercontent.com/33947539/196868974-8753c2a6-c36c-4134-861a-4d77c09b7157.png)

## Communication between pods on different nodes

Well, when the network bridge asks all the connected devices (i.e. pods) if they have the right IP address, none of them will say yes.

(Note that this part can vary based on the cloud provider and networking plugins.)

After that, the bridge falls back to the default gateway. This goes up to the cluster level and looks for the IP address.

At the cluster level, there’s a table that maps IP address ranges to various nodes. Pods on those nodes will have been assigned IP addresses from those ranges.

![image](https://user-images.githubusercontent.com/33947539/196869406-f3a5664e-6c21-4fc8-8b2e-08c878ecae78.png)

## Communication between pods and services
In Kubernetes, a service lets you map a single IP address to a set of pods. You make requests to one endpoint (domain name/IP address) and the service proxies requests to a pod in that service.

This happens via kube-proxy a small process that Kubernetes runs inside every node.

This process maps virtual IP addresses to a group of actual pod IP addresses.

Once kube-proxy has mapped the service virtual IP to an actual pod IP, the request proceeds as in the above sections.

## How does DNS work? How do we discover IP addresses?
DNS is the system for converting domain names to IP addresses.

Kubernetes clusters have a service responsible for DNS resolution.

Every service in a cluster is assigned a domain name like my-service.my-namespace.svc.cluster.local.

Pods are automatically given a DNS name, and can also specify their own using the hostname and subdomain properties in their YAML config.

So when a request is made to a service via its domain name, the DNS service resolves it to the IP address of the service.

Then kube-proxy converts that service's IP address into a pod IP address. After that, based on whether the pods are on the same node or on different nodes, the request follows one of the paths explained above.

## How to allow two pods running in same/different namespace communicate irrespective of the protocol using a servicename? 
By default, pods can communicate with each other by their IP address, regardless of the namespace they're in.

You can see the IP address of each pod with:

```yaml
kubectl get pods -o wide --all-namespaces
```

However, the normal way to communicate within a cluster is through Service resources.

A Service also has an IP address and additionally a DNS name. A Service is backed by a set of pods. The Service forwards requests to itself to one of the backing pods.

The fully qualified DNS name of a Service is:

```yaml
<service-name>.<service-namespace>.svc.cluster.local
```

This can be resolved to the IP address of the Service from anywhere in the cluster (regardless of namespace).

For example, if you have:

```
Namespace ns-a: Service svc-a → set of pods A

Namespace ns-b: Service svc-b → set of pods B
```

Then a pod of set A can reach a pod of set B by making a request to:

```yaml
svc-b.ns-b.svc.cluster.local
```


