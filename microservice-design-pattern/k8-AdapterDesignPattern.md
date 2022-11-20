## Adapter pattern

A container that transform output of the main container.

Prometheus works by querying an endpoint exposed by the target application. The endpoint must return the diagnostic data in a format that Prometheus expects. A possible solution is to configure each application to output its health data in a Prometheus-friendly way. However, You may need to switch your monitoring solution to another tool that expects another format. Changing the application code each time you need a new health-status format is largely inefficient. Following the Adapter Pattern, we can have a sidecar container in the same Pod as the app’s container. The only purpose of the sidecar (the adapter container) is to “translate” the output from the application’s endpoint to a format that Prometheus (or the client tool) accepts and understands.

![image](https://user-images.githubusercontent.com/33947539/202900667-e7433260-b460-4f6f-a98f-f56623b731ec.png)

## References:
https://www.weave.works/blog/kubernetes-patterns-the-adapter-pattern

