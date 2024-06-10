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
