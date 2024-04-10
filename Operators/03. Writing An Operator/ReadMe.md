## References
- Basic Ooperator Example: https://github.com/leovct/kube-operator-tutorial?source=post_page-----11eec1492d30--------------------------------
- Operator Theory: https://nakamasato.medium.com/kubernetes-operator-series-2-overview-of-controller-runtime-f8454522a539
- Another Operator Exmple: https://github.com/BackAged/tdset-operator/tree/main

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
Then try to restart and compile the program , IDE will suggest some more libs to download.
