# Microservice Architecture: Sidecar Pattern
Sidecar concept in Kubernetes is getting popular and .Its a common principle in container world, that container should address a single concern & it should do it well. 
The Sidecar pattern achieves this principle by decoupling the core business logic from additional tasks that extends the original functionality.

Sidecar pattern is a single node pattern made up of two containers.
The first is the application container which contains the core logic of the application (primary application). Without this container, application wouldn’t exist.
In addition, there is a Sidecar container used to extend/enhance the functionalities of the primary application by running another container in parallel on the 
same container group (Pod).
Since sidecar runs on the same Pod as the main application container it shares the resources — filesystem, disk, network etc.

## Usecases 

### Adding HTTPS to a Legacy Service
In addition to the main container (legacy app) we can add Nginx Sidecar container which runs in the same network namespace as the main container 
so that it can access the service running on localhost. At the same time Nginx terminate HTTPS traffic on the external IP address of the pod and 
delegate that traffic to the legacy application.

![image](https://user-images.githubusercontent.com/33947539/136905704-bf5672d9-582c-42ed-a1b0-8dd189e9ee4f.png)


### Dynamic Configuration with Sidecars
When the legacy app starts, it loads its configuration from the filesystem.
When configuration manager starts, it examines the differences between the configuration stored on the local file system and the configuration stored 
on the cloud. If there are differences, then the configuration manager downloads the new configuration to the local filesystem & notify legacy app to 
re-configure itself with the new configuration 

![image](https://user-images.githubusercontent.com/33947539/136906109-ad71ed01-983a-44fa-9df8-19421cc3437e.png)

### Log Aggregator with Sidecar
Another classic example is we can implement the Sidecar pattern by deploying a separate container to capture and transfer the access/error logs from 
the web server to log aggregator. Web server performs its task well to serve client requests & Sidecar container handle access/error logs. Since containers 
are running on the same pod, we can use a shared volume to read/write logs.

![image](https://user-images.githubusercontent.com/33947539/136906276-f000c770-3f01-4b83-8638-c1c868fc27ff.png)
