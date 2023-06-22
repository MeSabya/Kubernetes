## Understanding the KUBECONFIG!
- When you develop client applications/tools for Kubernetes, you communicate with the kube-apiserver using clients like client-go or kubectl and behind every client there is a kubeconfig.
- By default it is located in $HOME/.kube directory as file config. If there is any other kubeconfig file then you can refer it by setting its path to the environment variable KUBECONFIG.
- A kubeconfig file would typically look as shown

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://kubernetes.docker.internal:6443
  name: docker-desktop
users:
- name: docker-desktop
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
contexts:
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-desktop
current-context: docker-desktop
preferences: {}
```
There are three sections in it — Clusters, Users and Contexts. 
Let’s understand one by one.

### Clusters
The Clusters is a list of cluster objects that holds the information regarding various clusters the user would like to operate upon using this kubeconfig. Each cluster object is composed of details about the server and one of the possible authentication details (few are listed below)-

```yaml
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://kubernetes.docker.internal:6443
```

- server: Server is the address of the kubernetes cluster (https://hostname:port).
- tls-server-name: TLSServerName is passed to the server for SNI and is used in the client to check server certificates against. If ServerName is empty, the hostname used to contact the server is used.
- insecure-skip-tls-verify: InsecureSkipTLSVerify skips the validity check for the server’s certificate. This will make your HTTPS connections insecure.
- certificate-authority-data: CAData contains PEM-encoded certificate authority certificates. If empty, system roots should be used.

### Users 
The Users is a list of user objects that holds the information regarding different users of the clusters and their authentication details.

- Certificates
```yaml
users:
  - name: admin
    user:
      client-certificate-data: <base64 encoded client cert data>
      client-key-data: <base64 encoded client key>
```

- Authentication tokens
```yaml
users:
  - name: admin
    user:
      token: >_
        dGhpcyBpcyBhIHJhbmRvbSBzZW50ZW5jZSB0aGF0IGlzIGJhc2UgZW5jb2R
```
- Basic authentication
```yaml
users:
  - name: admin
    user:
      username: teddy-winters
      password: panda-summers
```

### Contexts
Contexts are list of context objects and each context is a triplet — Combination of cluster, user and a namespace.

```yaml
contexts:
- context:
    cluster: production
    namespace: live
    user: admin
  name: production-admin
```




