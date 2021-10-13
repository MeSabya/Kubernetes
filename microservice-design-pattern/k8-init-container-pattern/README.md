# Init Container Pattern

## What are Init Containers?
In Kubernetes, an init container is the one that starts and executes before other containers in the same Pod. 
It’s meant to perform initialization logic for the main application hosted on the Pod. For example, create the necessary user accounts, 
perform database migrations, create database schemas and so on.


## Init Containers Design Considerations
There are some considerations that you should take into account when you create init containers:

- They always get executed before other containers in the Pod. So, they shouldn’t contain complex logic that takes a long time to complete. 
  Startup scripts are typically small and concise.
- Init containers are started and executed in sequence. An init container is not invoked unless its predecessor is completed successfully.
- If any of the init containers fail, the whole Pod is restarted (unless you set restartPolicy to Never). 
  Restarting the Pod means re-executing all the containers again including any init containers.
- An init container is a good candidate for delaying the application initialization until one or more dependencies are available. 
  For example, if your application depends on an API that imposes an API request-rate limit, you may need to wait for a certain time period to be able 
  to receive responses from that API. Implementing this logic in the application container may be complex; as it needs to be combined with health and readiness probes. 
  A much simpler way would be creating an init container that waits until the API is ready before it exits successfully. 
  The application container would start only after the init container has done its job successfully.
- Init containers cannot use health and readiness probes as application containers do. The reason is that they are meant to start and exit successfully, 
  much like how Jobs and CronJobs behave.
- All containers on the same Pod share the same Volumes and network. You can make use of this feature to share data between the application and its init containers.
- Init containers are restarted when they fail. Hence, their code must be idempotent.
- 
