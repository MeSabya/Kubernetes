# What is a headless service, what does it do/accomplish, and what are some legitimate use cases for it?
As we know so far, A stateful application requires pods with a unique identity (for example, hostname). One pod should be able to reach other pods with well-defined names.

For a StatefulSet to work, it needs a Headless Service. A Headless Service does not have an IP address. Internally, 
it creates the necessary endpoints to expose pods with DNS names. The StatefulSet definition includes a reference to the Headless Service, 
but you have to create it separately.
Once the StatefulSet and the Headless Service are created, a pod can access another one by name prefixed with the service name.

***What is a Headless Service?***

Each connection to the service is forwarded to one randomly selected backing pod. **But what if the client needs to connect to all of those pods?** 
What if the backing pods themselves need to each connect to all the other backing pods. Connecting through the service clearly isn’t the way to do this. What is?

For a client to connect to all pods, it needs to figure out the the IP of each individual pod. One option is to have the client call the Kubernetes API server 
and get the list of pods and their IP addresses through an API call, but because you should always strive to keep your apps Kubernetes-agnostic, 
using the API server isn’t ideal

Luckily, Kubernetes allows clients to discover pod IPs through DNS lookups. Usually, when you perform a DNS lookup for a service, 
the DNS server returns a single IP — the service’s cluster IP. But if you tell Kubernetes you don’t need a cluster IP for your service 
(you do this by setting the clusterIP field to None in the service specification ), the DNS server will return the pod IPs instead of the single service IP. 
Instead of returning a single DNS A record, the DNS server will return multiple A records for the service, 
each pointing to the IP of an individual pod backing the service at that moment. Clients can therefore do a simple DNS A record lookup and get the IPs of 
all the pods that are part of the service. The client can then use that information to connect to one, many, or all of them.

Setting the clusterIP field in a service spec to None makes the service headless, as Kubernetes won’t assign it a cluster IP through which clients could connect 
to the pods backing it.

## Some Practical in Action:

**Deployment config**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  labels:
    app: server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```
**Regular Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: regular-svc
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
**Headless Service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-svc
spec:
  clusterIP: None # <= Don't forget!!
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
### Create all resources and run tmp pod.

- k run tmp01 --image=tutum/dnsutils -- sleep infinity

- k exec tmp01 -it -- /bin/sh

- nslookup regular-svc
  
  Server:     10.96.0.10
  
  Address:    10.96.0.10#53

  Name:   regular-svc.moon.svc.cluster.local
  
  Address: 10.109.150.46

- nslookup headless-svc
  
  Server:     10.96.0.10
  
  Address:    10.96.0.10#53

  Name:   headless-svc.moon.svc.cluster.local
  
  Address: 172.17.0.31
  
  Name:   headless-svc.moon.svc.cluster.local
  
  Address: 172.17.0.30
  
  Name:   headless-svc.moon.svc.cluster.local
  
  Address: 172.17.0.32

The DNS server returns three different IPs for the headless-svc.moon.svc.cluster.local FQDN.

**Note 1: with a headless service, clients can connect to its pods by connecting to the service’s DNS name, 
as they can with regular services. But with headless services, because DNS returns the pods’ IPs, clients connect directly to the pods, instead of through the service proxy.**

**Note 2: Headless services still provide load balancing across pods but through the DNS round-robin mechanism instead of through the service proxy.**

