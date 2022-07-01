# Understanding the KUBECONFIG!

When you develop client applications/tools for Kubernetes, you communicate with the kube-apiserver using clients like client-go or kubectl and behind every client there is a kubeconfig.

Kubeconfig primary use is to store 
1. Kubernetes contexts, built from definitions for accessing the API server (mainly endpoint URL and TLS config), and 
2. user or auth info (credentials, tokens or certificates).

The same configuration file is also used by other tools communicating with Kubernetes, for example, helm. Kubeconfig is read from ~/.kube/config by default, which can be overridden by the KUBECONFIG environment variable.

```yaml
apiVersion: v1
clusters:
- cluster:
    server: https://api.net6.paas.westeurope.rnd.az.myorg.net:6443
  name: api-net6-paas-westeurope-rnd-az-myorg-net:6443
- cluster:
    insecure-skip-tls-verify: true
    server: https://localhost:8443
  name: localhost:8443
- cluster:
    server: https://openshift.nce2.paas.myorg.net:8443
  name: openshift-nce2-paas-myorg-net:8443
contexts:
- context:
    cluster: openshift-nce2-paas-myorg-net:8443
    namespace: apa-int
    user: sa2nayak/openshift-nce2-paas-myorg-net:8443
  name: apa-int/openshift-nce2-paas-myorg-net:8443/sa2nayak
- context:
    cluster: api-net6-paas-westeurope-rnd-az-myorg-net:6443
    namespace: default
    user: sa2nayak/api-net6-paas-westeurope-rnd-az-myorg-net:6443
  name: default/api-net6-paas-westeurope-rnd-az-myorg-net:6443/sa2nayak
- context:
    cluster: localhost:8443
    namespace: default
    user: admin/localhost:8443
  name: default/localhost:8443/admin
- context:
    cluster: localhost:8443
    namespace: one
    user: admin/localhost:8443
  name: one/localhost:8443/admin
- context:
    cluster: api-net6-paas-westeurope-rnd-az-myorg-net:6443
    namespace: rdp
    user: sa2nayak/api-net6-paas-westeurope-rnd-az-myorg-net:6443
  name: rdp/api-net6-paas-westeurope-rnd-az-myorg-net:6443/sa2nayak
- context:
    cluster: localhost:8443
    namespace: sa2nayak
    user: admin/localhost:8443
  name: sa2nayak/localhost:8443/admin
- context:
    cluster: localhost:8443
    namespace: tkt-debug
    user: admin/localhost:8443
  name: tkt-debug/localhost:8443/admin
current-context: rdp/api-net6-paas-westeurope-rnd-az-myorg-net:6443/sa2nayak
kind: Config
preferences: {}
users:
- name: admin/localhost:8443
  user:
    token: GG12WT3nW86o5e-UC-InDVSr_d2etIWYlm2sOdLuNT4
- name: sa2nayak/api-net6-paas-westeurope-rnd-az-myorg-net:6443
  user:
    token: sha256~NP3JjPC6eyzX7AETUXFLBTAMEBOlsOa1v1a9Ak3jJE8
- name: sa2nayak/openshift-nce2-paas-myorg-net:8443
  user:
    token: KfxSmaWL1zFFK_qIQmEhRXMmOxOT4rCFRMSR__4dfLE
    
 ```  
 
 There are three sections in it — Clusters, Users and Contexts. Let’s understand one by one.
 
 ## Clusters:
 The Clusters is a list of cluster objects that holds the information regarding various clusters the user would like to operate upon using this kubeconfig. Each cluster object is composed of details about the server and one of the possible authentication details (few are listed below)-
 
 ```yaml
 clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://kubernetes.docker.internal:6443
```

server: Server is the address of the kubernetes cluster (https://hostname:port).

tls-server-name: TLSServerName is passed to the server for SNI and is used in the client to check server certificates against. If ServerName is empty, the hostname used to contact the server is used.

insecure-skip-tls-verify: InsecureSkipTLSVerify skips the validity check for the server’s certificate. This will make your HTTPS connections insecure.

certificate-authority-data: CAData contains PEM-encoded certificate authority certificates. If empty, system roots should be used.

## Users

The Users is a list of user objects that holds the information regarding different users of the clusters and their authentication details. Users can authenticate themselves by the following ways -

Certificates

```yaml
users:
  - name: admin
    user:
      client-certificate-data: <base64 encoded client cert data>
      client-key-data: <base64 encoded client key>
```

Authentication tokens

```yaml
users:
  - name: admin
    user:
      token: >_
        dGhpcyBpcyBhIHJhbmRvbSBzZW50ZW5jZSB0aGF0IGlzIGJhc2UgZW5jb2R
```

Basic authentication

```yaml
users:
  - name: admin
    user:
      username: teddy-winters
      password: panda-summers
```

## Contexts

Contexts are list of context objects and each context is a triplet — Combination of cluster, user and a namespace.

```yaml
contexts:
- context:
    cluster: production
    namespace: live
    user: admin
  name: production-admin
```

In the above example, the context production-admin means — use the credentials of admin user to access the live namespace of production cluster. It is important to have defined the cluster and user objects under the respective sections of this kubeconfig so that they are successfully referred.

Finally, there is one field in the kubeconfig called current-context that sets the default context to be used.



