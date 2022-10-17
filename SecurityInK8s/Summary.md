👉 *There are couples of ways to authenticate an ordinary user:*

1. Client-side X509 Client Certs
2. HTTP request
3. Bearer token

👉 *X509 Client Certs*:

The workflow goes as follows: you create a certificate request, sign it through a Certificate Authority (CA) and present it to the API server during the authentication phase. 
The API server consults the CA server to validate the certificate and, accordingly, approves or denies the request.

👉 *Bearer token*:

Bearer token is a static token verify method, to enable which, you need to start APIServer with token-auth-file=authfile.

👉 *HTTP Login*:

It’s basically a username and password login method. To enable it, you need to start the APIServer with basic-auth-file=authfile.

👉 *Service Account*:

In the Kubernetes cluster, any processes or applications in the container which resides within the pod can access the cluster by getting authenticated by the API server, using a service account.

👉 *The RBAC Components*:

- Rules
- Roles
- Subjects
- RoleBindings

👉 *Rule*:

A Rule is a set of operations (verbs), resources, and API groups.

👉 *Role*:

A Role is a collection of Rules. It defines one or more Rules that can be bound to a user or a group of users.

👉 *Subjects*:

Subjects define entities that are executing operations. A Subject can be a User, a Group, or a Service Account.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

