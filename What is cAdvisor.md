## What is cAdvisor? How Does it Work?

cAdvisor stands for container advisor. As it is evident from the name it provides resource usage, performance characteristics & related information about the containers running on the cloud. It is an open-source tool & runs as a daemon process in the background collecting, processing & aggregating useful DevOps information.

In Kubernetes cAdvisor is integrated into the Kubelet binary. It is pretty intelligent to auto-discover all the containers running in the machine & collect CPU, memory, file system & network usage statistics. It also provides a comprehensive overall machine usage by analyzing the root container.

### What is kubelet 
A Kubelet kind of manages things on the cluster. It manages the pods & the containers on a machine. It is responsible for fetching individual container usage statistics from cAdvisor. The collected data is exposed via REST API

As stated the Kubelets, in turn, pull data from the cAdvisor. The entire data is saved in a data store such as InfluxDB & streamed to Grafana for visualization.
