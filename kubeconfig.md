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

By default there is one user, one context, one namespace defined in the default config file.

### Commands for kube config 

#### How to check current config file that is in use 
```yaml
kubectl config view
```
#### How to change the current context 

```yaml
kubectl config use-context <context name>
```
#### I would like to use the dev-user to access test-cluster-1. Set the current context to the right one so I can do that.

Note that : We should not use the default config here , rather we should use the another context And some part of the current config is:

```yaml
contexts:
- name: test-user@development
  context:
    cluster: development
    user: test-user

- name: aws-user@kubernetes-on-aws
  context:
    cluster: kubernetes-on-aws
    user: aws-user

- name: test-user@production
  context:
    cluster: production
    user: test-user

- name: research
  context:
    cluster: test-cluster-1
    user: dev-user

users:
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key
```

##### Answer is

```yaml
kubectl config --kubeconfig=/root/my-kube-config use-context research

To know the current context, run the command: 
kubectl config --kubeconfig=/root/my-kube-config current-context
```











