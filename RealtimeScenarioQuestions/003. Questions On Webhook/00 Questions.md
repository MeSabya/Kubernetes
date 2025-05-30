## List and describe available PATCH methods
- JSON Patch: can be used for both built-in and custom resources.
- JSON Merge Patch: ditto. For the difference from JSON patch, read http://erosb.github.io/post/json-patch-vs-merge-patch/ .
- admission.PatchResponseFromRaw(req.Object.Raw, marshaledPod)

## Describe the problem and solution when multiple mutating webhooks edit the same resource.

There is no way to specify the order of applying mutating webhooks for kube-apiserver.
Suppose we have two mutating webhooks to edit Pods, one is to add a volume mount configuration to all containers, and another is to add a container. 
To make all containers have the volume mount configuration, the first webhook needs to be called after the second.

We can set the reinvocation policy of the first webhook to IfNeeded to make the first called after the second.

### Addressing the Scenario:
When two webhooks need to coordinate in a specific way, such as:

- Webhook A adds a volume mount to all containers in a Pod.
- Webhook B adds a new container to the Pod.

To ensure that Webhook A is reinvoked after Webhook B has modified the resource, Kubernetes provides a mechanism through the reinvocationPolicy field.

### Reinvocation Policy
The reinvocationPolicy setting allows the API server to reinvoke mutating admission webhooks after one or more earlier webhooks have made changes to a resource. This helps to handle cases where webhooks interact indirectly or sequentially.

```bash
reinvocationPolicy: IfNeeded
```
This ensures that a webhook will be reinvoked if earlier webhooks modify the resource in a way that makes the webhook’s logic applicable again.

### webhook-a add-volume-mount
```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: add-volume-mount
webhooks:
  - name: add-volume-mount.k8s.io
    clientConfig:
      service:
        name: add-volume-mount-service
        namespace: default
        path: "/mutate"
      caBundle: <CA_BUNDLE>
    admissionReviewVersions: ["v1"]
    sideEffects: None
    reinvocationPolicy: IfNeeded
    rules:
      - operations: ["CREATE"]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
```
### webhook-b add-container

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: add-container
webhooks:
  - name: add-container.k8s.io
    clientConfig:
      service:
        name: add-container-service
        namespace: default
        path: "/mutate"
      caBundle: <CA_BUNDLE>
    admissionReviewVersions: ["v1"]
    sideEffects: None
    rules:
      - operations: ["CREATE"]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
```

When would you use a Mutating Admission Webhook vs a Validating Admission Webhook?

How do you deploy and scale a webhook in a production cluster?

How do you ensure high availability and minimal latency in a webhook?

How do you secure webhooks using cert-manager or manual TLS?

How do you handle ordering and conflicts when multiple mutating webhooks are present?

Can a webhook cause a cluster outage? How would you prevent that?

How would you write a webhook to block deprecated API usage in the cluster?

How do you test a webhook locally before deploying?

