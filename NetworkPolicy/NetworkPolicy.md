# How Does Network Policy Work?
There are unlimited situations where you need to permit or deny traffic from specific or different sources. This is utilized in Kubernetes to indicate how gatherings of pods are permitted to speak with one another and with outside endpoints.

Rules:

- Traffic is allowed unless there is a Kubernetes network policy selecting a pod.
- Communication is denied if policies are selecting the pod but none of them have any rules allowing it.
- Traffic is allowed if there is at least one policy that allows it.

## Network Policy Specification
- PodSelector – Each of these includes a pod selector that selects the grouping of pods to which the policy applies. This selects particular Pods in the same namespace as the Kubernetes Network Policy which should be allowed as ingress sources or egress destinations.

- Policy Types – indicates which sorts of arrangements are remembered for this approach, Ingress, or Egress.

- Ingress – Each Network Policies may include a list of allowed ingress rules. This includes inbound traffic whitelist rules.

- Egress – Each Network Policy may include a list of allowed egress rules. This includes outbound traffic whitelist rules.

## spec.podSelector vs spec.ingress.from.podSelector
The spec.podSelector and spec.ingress[].from[].podSelector fields in a Kubernetes NetworkPolicy serve different purposes. Here’s a detailed explanation of each and how they differ:

spec.podSelector
Purpose: Specifies which pods the NetworkPolicy applies to.
Scope: Defines the target pods within the namespace to which the policy rules (ingress and/or egress) are applied.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: combined-example
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          team: dev
      podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
```
In this example:

- spec.podSelector: The NetworkPolicy targets pods with the label app: myapp.
- Ingress Rule: Allows incoming traffic to myapp pods from:
   - Pods with the label app: frontend in the same namespace.
   - Pods with the label app: backend in namespaces labeled team: dev.
- Egress Rule: Allows outgoing traffic from myapp pods to:
        Pods with the label app: database on TCP port 5432.

This policy ensures that only specific sources can send traffic to the myapp pods and controls where the myapp pods can send traffic, enhancing network security and control.

## References:
https://github.com/ahmetb/kubernetes-network-policy-recipes
