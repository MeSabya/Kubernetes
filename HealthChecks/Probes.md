# Kubernetes' probes
- The probes are under responsibility of the Probe Manager that is running inside the Kubelet. Probes are defined and exercised at container level. They are optional.
- There are 2 types of probes Readiness and Liveness. Each probe has its worker and they are not correlated.
- The readiness status of the pod is the result of a logical AND of all the readiness status of its container. A container with no probe is by default "Ready".

In a nutshell, the probe's Handler can be of 3 types:
- Exec: specific command to execute inside the container. Return value of 0 expected for healthy container.
- HTTP/GET: define an URI and port to be used to perform a HTTP query. Return value of 200  (HTTP/OK) expected for healthy container.
- TCPSocket: define a port. Ability to setup connect is expected for healthy container.

## Attributes in Kubernets Probes:
**InitialDelaySeconds**: To avoid stressing the containers immediately with the probe, you can define this. This is useful in case you know that your container need a warm-up period.

**PeriodSeconds**:Defines the frequency of polling. By default it is 10 seconds.

**TimeoutSeconds**: Depending on the handler and its implementation you may need to adjust the TimeoutSeconds value to give the handler some time to respond. The default value is 1 seconds. In practice a reasonable value should be smaller than the PeriodSeconds; if not you may end-up queuing you probe request in the container depending on your implementation.

**FailureThreshold**: If your probe is likely to return some false positive, or if you plan some kind of grace period for your container before declaring it not ready,default value of 3.

**SuccessThreshold**: Finally to bring back your container to a ready state you may want to call for more than 1 successful probe. Default value is 1.

***The probe stops when a pod enter a phase "failed" or "succeed". The probe is played only on running containers once the InitialDelaySeconds as expired. This means that probe can run on some containers while the pod is in "pending" phase.***

## Readiness Probe or Liveness Probe:

**Liveness probe**: Will periodically probe the application and kill the container and recreate it in case it is not responding / reporting failure.

**Readiness probe**: Will periodically probe the application and allow traffic if and only if it is responding / not reporting failure. It is also checked during the rolling update process to validate that the new pod is ready to receive traffic. For instance, say your application depends on a database and memcached. If both of these need to be up and running for your app to serve traffic, then you could say that both are required for your app's "readiness".

If the readiness probe for your app fails, then that pod is removed from the endpoints that make up a service. This makes it so that pods that are not ready will not have traffic sent to them by Kubernetes' service discovery mechanism. This is really helpful for situations where a new pod for a service gets started; scale up events, rolling updates, etc. Readiness probes make sure that pods are not sent traffic in the time between when they start up, and and when they are ready to serve traffic.

- If readiness time < liveness time then all is ok.
- if liveness time <= readiness time then crash loop backoff.

### Readiness Probe/Liveness Probe usage

Use a **Liveness probe** if you want to ask kubernetes to restart a container (a container not the pod). This can be useful is you suspect that your process cannot recover (case of deadlock, case of leak, ...)

Use a **Readiness probe** if you want to ask kuberntes to stop (or not start to) sending you traffic while you recover/get ready. Useful for startup check and warm-up, as well as for reducing workload in case of unexpected queuing for example. Use it also if you are able to detect that you won't be able to process incoming queries, for example you are missing a dependency and you cannot deliver a degraded response (caution read next sections about dependencies and probes).


# References:
https://alibaba-cloud.medium.com/kubernetes-configure-liveness-and-readiness-probes-588aa2cab90d
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/?spm=a2c41.12911419.0.0.530a2c3ezn4w48#define-a-liveness-command

