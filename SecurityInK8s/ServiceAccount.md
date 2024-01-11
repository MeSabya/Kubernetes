## What Are Kubernetes Service Accounts?
Whenever you access your Kubernetes cluster with kubectl, you are authenticated by Kubernetes with your user account. User accounts are meant to be used by humans. But when a pod running in the cluster wants to access the Kubernetes API server, it needs to use a service account instead. Service accounts are just like user accounts but for non-humans.

## But, you may wonder, why would pods inside the Kubernetes cluster need to connect to the Kubernetes API at all? 
Well, there are multiple use cases for it. 

- The most common one is when you have a CI/CD pipeline agent deploying your applications to the same cluster. 
- Many cloud-native tools also need access to your Kubernetes API to do their jobs, such as logging or monitoring applications.

An application like Prometheus accessing the cluster to monitor it is a type of service account

# How Does Kubernetes Service Account Works?
## Case 1: When you have an external application trying to access Kubernetes cluster API servers.

Suppose there is a web page: My Web Page which has a list of items to be displayed, this data needs to be fetched from an API server hosted in the Kubernetes cluster as shown above in the figure. To do so, we need to a service account that will be enabled by cluster API servers to authenticate and access the data from the cluster servers.

When you are done creating a service account, a **service account** token also gets generated, this token is what will be required by our My Web Page application to access the data via apis.

We will see how we can connect to API server using this **service account**.

### Default Service Account

your pods have the default service account assigned even when you don’t ask for it. This is because every pod in the cluster needs to have one (and only one) service account assigned to it. What can your pod do with that service account? Well, pretty much nothing. That default service account doesn’t have any permissions assigned to it.

We can validate that as well. Let’s get into our freshly deployed nginx pod and try to connect to a Kubernetes API from there. For that, we’ll need to export a few environment variables and then use the curl command to send an HTTP request to the Kubernetes API.

```shell
# Export the internal Kubernetes API server hostname
$ APISERVER=https://kubernetes.default.svc

# Export the path to ServiceAccount mount directory
$ SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount

# Read the ServiceAccount bearer token
$ TOKEN=$(cat ${SERVICEACCOUNT}/token)

# Reference the internal Kubernetes certificate authority (CA)
$ CACERT=${SERVICEACCOUNT}/ca.crt

# Make a call to the Kubernetes API with TOKEN
$ curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/default/pods
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}#

```
## Creating Your Own Service Accounts
So, if you want your pod to actually be able to talk to the Kubernetes API and do something, you have two options. You either need to assign some permissions to the default service account, or you need to create a new service account. The first option is not recommended. In fact, you shouldn’t use the default service account for anything. Let’s choose the recommended option then, which is creating dedicated service accounts. It’s also worth mentioning here that, just like with user access, you should create separate service accounts for separate needs.

```shell
$ kubectl create serviceaccount nginx-serviceaccount
serviceaccount/nginx-serviceaccount created
```

### Assigning Permissions to a Service Account
OK, you created a new service account for your pod, but by default, it won’t do much more than the default service account (called “default”) that you saw previously. To change that, you can use the standard Kubernetes role-based access control mechanism. This means that you can either use existing role or create a new one (or ClusterRole) and then use RoleBinding to bind a role with your new ServiceAccount.

For the purpose of the demo, you can assign a built-in Kubernetes ClusterRole called “view” that allows viewing all resources. You then need to create a RoleBinding for your Service Account.

```shell
kubectl create rolebinding nginx-sa-readonly \
  --clusterrole=view \
  --serviceaccount=default:nginx-serviceaccount \
  --namespace=default
```

## Specifying ServiceAccount For Your Pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx1
  labels:
    app: nginx1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx1
  template:
    metadata:
      labels:
        app: nginx1
    spec:
      serviceAccountName: nginx-serviceaccount
      containers:
      - name: nginx1
        image: nginx
        ports:
        - containerPort: 80
```

```yaml
$ curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/default/pods
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "52233"
  },
  "items": [
    {
      "metadata": {
        "name": "nginx1-65448895f9-5j6b6",
        "generateName": "nginx1-65448895f9-",
        "namespace": "default",
        "uid": "b09bfa93-a388-4cd9-9495-131f620613d0",
        "resourceVersion": "49536",
(...)
```


## Default service account Path of pod

In a Kubernetes cluster, the default service account tokens for pods are mounted as secrets inside each pod. The path to the default service account token and certificate authority (CA) bundle is typically:

      Token: /var/run/secrets/kubernetes.io/serviceaccount/token
      CA Bundle: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt

These paths are automatically mounted by Kubernetes into each pod's filesystem. The service account token is used by the pod to authenticate itself to the Kubernetes API server, and the CA bundle is used to verify the identity of the API server.


## What is authorization token in Sservice Account


In Kubernetes, service accounts are used to provide an identity for processes that run in a pod. Each service account has an associated token that can be used for authentication and authorization when interacting with the Kubernetes API server.

An authorization token in the context of a service account is a piece of information that represents the identity of the service account. This token is used by the pod to authenticate itself to the Kubernetes API server. When the pod makes requests to the API server, it includes this token in the request headers to prove its identity.

### Token Location:
The authorization token is typically mounted into the pod's filesystem at the path /var/run/secrets/kubernetes.io/serviceaccount/token. This path is automatically created and populated by Kubernetes.

Here's a simplified example of how a pod might use the authorization token to interact with the Kubernetes API server:

```bash
# Get the token from the service account
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

# Make an API request with the token
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/pods
```

## Along with the service account, why role and role binding was needed?

In Kubernetes, a ServiceAccount is an identity that a pod can use to interact with the Kubernetes API server and other resources within the cluster. While a ServiceAccount provides an identity, it does not grant any permissions by itself. To control access to resources, you need to use Role (RBAC Role) and RoleBinding (RBAC RoleBinding) in combination with ServiceAccounts.

Here's a brief explanation of each component:

### ServiceAccount:

A ServiceAccount is a Kubernetes object that represents an identity associated with a set of pods.
Pods use ServiceAccounts to authenticate and authorize their interactions with the Kubernetes API server and other resources.

### Role:

A Role is a set of rules that define a set of permissions for accessing resources within a namespace.
It is a way to grant permissions at the namespace level.

### RoleBinding:

A RoleBinding binds a Role to a ServiceAccount, associating the set of permissions defined in the Role with the ServiceAccount.
It establishes the connection between a ServiceAccount and the set of rules defined in a Role.

Let's see an example to illustrate why Role and RoleBinding are needed along with a ServiceAccount:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: my-namespace
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pod-reader
  namespace: my-namespace
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: my-namespace
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: my-namespace
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
In this example:

- my-service-account is a ServiceAccount in the namespace my-namespace.
- pod-reader is a Role that grants permissions to get and list pods within the namespace.
- read-pods is a RoleBinding that binds the my-service-account ServiceAccount to the pod-reader Role.

## References

https://loft.sh/blog/kubernetes-service-account-what-it-is-and-how-to-use-it

