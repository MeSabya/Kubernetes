## LoadBalancer

load balancers have different characteristics from ingresses. Rather than a standalone object like an ingress, a load balancer is just an extension of a service.

Let's see a simple example of a service with a load balancer:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

Just like our earlier service example, here, we're creating a service that routes traffic to any pod running our API application. In this case, we've included a LoadBalancer configuration.

👉 For this to work, the cluster must be running on a provider that supports external load balancers. All of the major cloud providers support external load balancers using their own resource types:

- AWS uses a Network Load Balancer
- GKE also uses a Network Load Balancer
- Azure uses a Public Load Balancer

Just like we saw with different ingress controllers, different load balancer providers have their own settings. Typically, we manage these settings directly with either a CLI or tool specific to the infrastructure rather than with YAML. Different load balancer implementations will also provide additional features such as SSL termination.

👉 **Because load balancers are defined per service, they can only route to a single service. This is different from an ingress, which has the ability to route to multiple services inside the cluster.**

Also, keep in mind that regardless of the provider, using an external load balancer will typically come with additional costs. This is because, unlike ingresses and their controllers, external load balancers exist outside of the Kubernetes cluster. Because of this, most cloud providers will charge for the additional resource usage beyond the cluster itself.


Many cloud-based Kubernetes deployments prefer LoadBalancer because it supports multiple protocols and multiple ports per service. LoadBalancer works with external network load balancers to distribute traffic according to your preferred load balancing strategy. LoadBalancer works best with large public cloud providers because it can be configured to automatically provision and de-provision external IP addresses and load balancers for your services.

The downside of LoadBalancer is primarily the cost. By default, it assigns an individual external IP address to every service, and then each IP needs its own external load balancer configured in the cloud. This can feel like overkill, especially when you’re running multiple services on every cluster, which is basically the standard in Kubernetes. The costs of a large pool of IP addresses and load balancers will quickly add up as your Kubernetes environment grows, which can limit your scalability.

***Some popular Kubernetes load balancer strategies include:***

![image](https://user-images.githubusercontent.com/33947539/187860028-f1ea40c7-7543-4e4b-88bb-676980a14780.png)

## Ingress

👉 Ingress is actually NOT a type of service. Instead, it sits in front of multiple services and act as a “smart router” or entrypoint into your cluster.

*An ingress is really just a set of rules to pass to a controller that is listening for them.*
You can deploy a bunch of ingress rules, but nothing will happen unless you have a controller that can process them. A LoadBalancer service could listen for ingress rules, if it is configured to do so.

You can also create a NodePort service, which has an externally routable IP outside the cluster, but points to a pod that exists within your cluster. This could be an Ingress Controller.

***There are a few things to keep in mind when creating an ingress.***

👉 First, they are designed to handle web traffic (HTTP or HTTPS). While it is possible to use an ingress with other types of protocols, it typically requires extra configuration.

👉 Second, an ingress can do more than just routing. Some other use cases include load balancing and SSL termination.

Most importantly, the ingress object by itself does not actually do anything. So, for an ingress to actually do anything, we need to have an ingress controller available.

### Ingress Controllers
As with most Kubernetes objects, an ingress requires an associated controller to manage it. However, while Kubernetes provides controllers for most objects like deployments and services, it does not include an ingress controller by default. Therefore, it is up to the cluster administrator to ensure an appropriate controller is available.

Most cloud platforms provide their own ingress controllers, but there are also plenty of open-source options to choose from. Perhaps the most popular is the nginx ingress controller, which is built on top of the popular web server of the same name.

Let's check out a sample configuration using the nginx ingress controller:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
  ```

In this example, we create an ingress that routes any request that starts with /api to a Kubernetes service named api-service.

Notice the annotations field contains values that are specific to nginx. Because all ingress controllers use the same API object, we typically use the annotations field to pass specific configurations into the ingress controller.

There are dozens of available ingress controllers in the Kubernetes ecosystem, and covering them all is well beyond the scope of this article. However, because they use the same API object, they share some common features:

**Ingress Rules**: the set of rules that define how to route traffic to a specific service (typically based on URL or hostname)

**Default Backend**: A default resource that handles traffic that does not match any rule

**TLS**: A secret that defines a private key and certificate to allow TLS termination

Different ingress controllers build on these concepts and add their own functionality and features

Ingress controllers are pods, just like any other application, so they’re part of the cluster and can see other pods.

**Ingress Controllers are susceptible to the same walled-in jail as other Kubernetes pods. You need to expose them to the outside via a Service with a type of either NodePort or LoadBalancer. However, now you have a single entrypoint that all traffic goes through: one Service connected to one Ingress Controller, which, in turn, is connected to many internal pods. The controller, having the ability to inspect HTTP requests, directs a client to the correct pod based on characteristics it finds, such as the URL path or the domain name.**

Consider this example of an Ingress, which defines how the URL path /foo should connect to a backend service named foo-service, while the URL path /bar is directed to a service name bar-service.

![image](https://user-images.githubusercontent.com/33947539/187853817-3866f7f1-3835-40e2-a687-1a3a980ef956.png)

Ultimately, the two paths, /foo and /bar, are served by a common IP address and domain name, such as example.com/foo and example.com/bar. This is essentially the API Gateway pattern. In an API Gateway, a single address routes requests to multiple backend applications.

## References
https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html






