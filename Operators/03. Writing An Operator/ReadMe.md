## References
- Basic Ooperator Example: https://github.com/leovct/kube-operator-tutorial?source=post_page-----11eec1492d30--------------------------------
- Operator Theory: https://nakamasato.medium.com/kubernetes-operator-series-2-overview-of-controller-runtime-f8454522a539
- Another Operator Exmple: https://github.com/BackAged/tdset-operator/tree/main

## The operator can have a main loop to spawn and manage multiple application controllers, true or false?

Managing Multiple Application Controllers
In the context of a Kubernetes operator, managing multiple application controllers typically involves:

- Defining Multiple CRDs: Each CRD corresponds to a different type of application or resource.
- Creating Multiple Reconcilers: Each reconciler handles the logic for a specific CRD.
- Managing Reconciler Instances: The operator's main loop (the manager) handles starting and stopping these reconcilers.

## For an operator, can we run with multiple replicas?

Yes, you can run a Kubernetes operator with multiple replicas to achieve high availability and better performance. When an operator runs with multiple replicas, Kubernetes ensures that the custom resources are reconciled by only one replica at a time, using leader election to coordinate which instance is actively performing the reconciliation tasks.

## Some important libraries to remember

```golang
import (
    "context"
    "fmt"

    appsv1 "k8s.io/api/apps/v1"
    corev1 "k8s.io/api/core/v1"
    "k8s.io/apimachinery/pkg/api/errors"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/runtime"
    ctrl "sigs.k8s.io/controller-runtime"
    "sigs.k8s.io/controller-runtime/pkg/client"
    "sigs.k8s.io/controller-runtime/pkg/controller/controllerutil"
    "k8s.io/apimachinery/pkg/runtime"            //
	  "k8s.io/apimachinery/pkg/runtime/serializer"

    appsv1alpha1 "github.com/example/my-operator/api/v1alpha1"
)
```
Here's a breakdown of each import:

- context: Package for handling context in Go, often used for managing cancellation and timeouts.
- fmt: Standard Go package for formatting and printing strings.
- appsv1: Package containing Kubernetes API types for apps/v1 resources (e.g., Deployments).
- corev1: Package containing Kubernetes API types for core/v1 resources (e.g., Pods, Services).
- errors: Package for handling errors in Go.
- metav1: Package containing Kubernetes API types for object metadata.
- runtime: Package for handling Go runtime-related functionality.
- ctrl: Package for building Kubernetes controllers using controller-runtime.
- client: Package containing utilities for interacting with the Kubernetes API server.
- controllerutil: Package containing utilities for working with Kubernetes controller logic.
- appsv1alpha1: Package containing Kubernetes API types for custom resources defined in your operator.
- k8s.io/apimachinery/pkg/runtime: This package provides interfaces and types for working with runtime objects in Kubernetes, such as Object and List.
- k8s.io/apimachinery/pkg/runtime/serializer: This package contains utilities for serializing and deserializing Kubernetes API objects, including various serializers and codecs.

## Do i need to remember these libraries ?
No , not necessarily , but its good if you remember the above libs as mentioned above. Atleast remember for operator writting we need controller-runtime and controllerutil 
For admission controller we  dont need them.

Remember the headers used for pods, deployments etc.

Then while writing the operator or controller start with :

```shell
go get k8s.io/client-go@v0.21.1
go get k8s.io/apimachinery@latest
go mod tidy
```

## So operator is a like daemon which runs always ? i mean when there is a change in configmap how this operator will be invoked ?
Operators are typically implemented as controllers that run continuously inside a Kubernetes cluster. These controllers are usually deployed as Pods managed by Deployments, ensuring they are always running and highly available.

### How Operators Are Triggered
Operators are triggered by changes in Kubernetes resources via a mechanism called informers or watches. Here's how this works in the context of the ConfigMapCopy operator we discussed:

- Setting Up Watches: When you set up your operator, you configure it to watch specific Kubernetes resources (e.g., ConfigMaps). This is done in the 
  SetupWithManager function in Kubebuilder, where you define which resources to watch.

- Event Handling: Kubernetes provides an event-based system where changes to resources (such as creation, updates, or deletions) generate events. The controller 
  watches for these events and enqueues them for processing.

- Reconcile Loop: The controller's Reconcile function is called whenever an event is processed. This function contains the logic to handle the changes, such as 
  copying a ConfigMap from one namespace to another.

### Example with Kubebuilder
Let’s look at how this works in our Kubebuilder example:

Watch Setup
In the ConfigMapCopyReconciler struct, the SetupWithManager method specifies that the controller should watch for changes to ConfigMap resources:

```go
func (r *ConfigMapCopyReconciler) SetupWithManager(mgr ctrl.Manager) error {
    return ctrl.NewControllerManagedBy(mgr).
        For(&corev1.ConfigMap{}).
        Complete(r)
}
```

### Continuous Operation
Since the operator is running inside a Deployment, Kubernetes ensures it is always running. If the operator Pod crashes or is terminated, Kubernetes restarts it. The operator continuously watches for changes and processes events as they occur.

### Summary
- Always Running: Operators run as long-lived processes, typically managed by Deployments.
- Event-Driven: Operators watch for events related to the resources they manage.
- Reconcile Loop: When changes are detected, the Reconcile function is called to handle the changes.

This event-driven model allows operators to react promptly to changes in the cluster, ensuring that the desired state specified by the custom resources is maintained.


Then try to restart and compile the program , IDE will suggest some more libs to download.
