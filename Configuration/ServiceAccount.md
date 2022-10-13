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
















## References

https://loft.sh/blog/kubernetes-service-account-what-it-is-and-how-to-use-it

