## Workflow
- Namespace Creation: When a container starts, Docker creates a new network namespace for it.
- veth Pair Creation: Docker creates a veth pair, linking one end to the container's namespace and the other to the host's network namespace.
- Bridge Connection: The host end of the veth pair is connected to the Docker bridge (docker0), enabling inter-container communication.
- IP Assignment: Docker assigns an IP address from a private subnet to the container’s end of the veth pair.
- iptables Rules: Docker configures iptables rules to enable NAT, allowing containers to access external networks.

![image](https://github.com/MeSabya/Kubernetes/assets/33947539/562b64ab-deea-45c4-9a3a-c653adbf6b0e)

## Suppose I ran a docker on linux machine , what is docker host here linux machine is the docker host both are same is it ?

Yes, the terms "Docker host" and "Linux machine" refer to the same entity in this context.

The Docker host is the physical or virtual machine where the Docker daemon runs, and where Docker containers are executed. When you run Docker on a Linux machine, that machine acts as the Docker host, 
providing the necessary environment and resources for Docker containers to run.

## What is docker-0 and etho0.
In the context of Docker networking on a Linux machine:

### docker0
docker0 is the default bridge network created by Docker.
When Docker is installed, it automatically creates a virtual bridge named docker0.
All Docker containers connected to this default bridge can communicate with each other.
Docker assigns an IP address from a private subnet to this bridge and connects the container's network interfaces to it.

### eth0
eth0 is a common name for the first Ethernet network interface on a Linux machine.
This interface connects the Linux host to its local network or the internet.
eth0 typically represents the physical or virtual network interface of the host system itself, allowing it to send and receive network traffic.

### How They Work Together
- docker0 facilitates container-to-container communication within the Docker host.
- eth0 handles external network traffic for the host, and containers can use eth0 indirectly to reach external networks, typically via Network Address Translation (NAT).

These interfaces enable both isolated and external communication for Docker containers running on the host machine.

![image](https://github.com/MeSabya/Kubernetes/assets/33947539/a024521d-fed5-492b-8acd-81f1603c314a)

![image](https://github.com/MeSabya/Kubernetes/assets/33947539/f26dbe00-768c-432e-966d-c3201d357633)

# Most important One
## From the external world how request reaches to the docker container?
So we got the IP address of the container for example 172.17.0.3 and we know that container is running on port 80.
Two questions:

1. What will happen if we do a curl http://172.17.0.3:80 from host? 
2. What will happen if we do a curl http://172.17.0.3:80 from external world?

###  What will happen if we do a curl http://172.17.0.3:80 from host? 
We can access the container using http://172.17.0.3:80 from the host without port forwarding if the following conditions are met:

- Network Bridge: Docker containers are connected to a bridge network by default, allowing them to communicate with each other and the host.
- Firewall Rules: No firewall rules are blocking traffic to the container’s IP and port.
- Routing: The Docker bridge network (usually docker0) allows the host to route traffic to the container’s IP address.
- This setup is typical in a default Docker installation, enabling direct access to container IPs from the host.

###  What will happen if we do a curl http://172.17.0.3:80 from external world?
We cannot access the container using http://172.17.0.3:80 from external without port forwarding.

When an external request reaches a Docker container, it follows this general flow:

- External Request: An external client sends a request to the Docker host's IP address on a specified port.
- Host to Container Port Mapping: The Docker engine on the host maps the request to the appropriate container based on port mapping configurations. This mapping is specified when the container is started with the -p option, e.g., docker run -p 8080:80 mycontainer.
- Docker Bridge: The request is routed from the host through the Docker bridge network (typically docker0) to the container's network namespace.
- Container Network Namespace: Inside the container, the request is received on the mapped port by the container's application.

***When you run a command like docker run -p 8080:80 nginx, Docker takes care of setting up the necessary iptables rules for you. Here's how it works:***

- Bridge Network: Docker creates a virtual bridge network (usually docker0) to which all containers are connected. This allows containers to communicate with each other and the host.
- Network Namespace: Each container gets its own network namespace, isolating its networking stack from other containers and the host.
- Virtual Ethernet Pairs: Docker creates a pair of virtual Ethernet interfaces (veth pair) for each container. One end is attached to the container's network namespace (usually named eth0 inside the container), and the other end is connected to the docker0 bridge.
- IP Address Assignment: Docker assigns an IP address to the container's eth0 interface from the bridge network's IP range.
- NAT and Port Forwarding: Docker sets up NAT rules using iptables to forward traffic from the host's port 8080 to the container's port 80.

#### Behind the Scenes
Here’s what Docker does with iptables:

```sh
iptables -t nat -A DOCKER -p tcp --dport 8080 -j DNAT --to-destination <container-ip>:80
iptables -t nat -A POSTROUTING -s <container-ip>/32 -j MASQUERADE
```

These rules handle forwarding traffic from the host to the container and masquerading the source IP address so that return traffic is correctly routed back to the original requester.
By handling this automatically, Docker simplifies the process of managing network connectivity for containers, allowing you to focus on your applications.
