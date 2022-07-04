# Kubernetes API Concepts

The Kubernetes API is a resource-based (RESTful) programmatic interface provided via HTTP. It supports retrieving, creating, updating, and deleting primary resources via the standard HTTP verbs (POST, PUT, PATCH, DELETE, GET).

👉 Kubernetes supports efficient change notifications on resources via watches. 

👉 Kubernetes also provides consistent list operations so that API clients can effectively cache, track, and synchronize the state of resources.

## Kubernetes API terminology

Kubernetes generally leverages common RESTful terminology to describe the API concepts:

- A resource type is the name used in the URL (pods, namespaces, services)
- All resource types have a concrete representation (their object schema) which is called a kind
- A list of instances of a resource is known as a collection
- A single instance of a resource type is called a resource, and also usually represents an object
- For some resource types, the API includes one or more sub-resources, which are represented as URI paths below the resource

## Resource URIs

You can also access collections of resources (for example: listing all Nodes). The following paths are used to retrieve collections and resources:

👉 **Cluster-scoped resources:**

GET /apis/GROUP/VERSION/RESOURCETYPE - return the collection of resources of the resource type
GET /apis/GROUP/VERSION/RESOURCETYPE/NAME - return the resource with NAME under the resource type

👉 **Namespace-scoped resources:**

GET /apis/GROUP/VERSION/RESOURCETYPE - return the collection of all instances of the resource type across all namespaces
GET /apis/GROUP/VERSION/namespaces/NAMESPACE/RESOURCETYPE - return collection of all instances of the resource type in NAMESPACE
GET /apis/GROUP/VERSION/namespaces/NAMESPACE/RESOURCETYPE/NAME - return the instance of the resource type with NAME in NAMESPACE

## Efficient detection of changes
The Kubernetes API allows clients to make an initial request for an object or a collection, and then to track changes since that initial request: a watch. Clients can send a list or a get and then make a follow-up watch request.

## References
https://kubernetes.io/docs/reference/using-api/api-concepts/#efficient-detection-of-changes



