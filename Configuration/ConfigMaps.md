Kubernetes natively supports 2 resources geared towards storing configuration consumed by pods. Both configuration types required data to be represented in a key:value pair format.
- Config Maps: use to store non-confidential data
- Secrets: use to store confidential data (tokens, SSH keys, passwords, certificates, etc)

To configure your apps in Kubernetes, you can use:
- Good old environment variables
- ConfigMap
- Secret

> Config maps are not suited for storing large quantities of data. Please use external storage for this purpose. As a side-note, etcd, key-value store where Kubernetes stores all its state can only hold resources up to 1MN in size.
**Secrets in Kubernetes are actually not encrypted, but rather base64 encoded**, so best for storing critical sensitive information, recommendation is to use key vaults such as Hashicorp Vault with Helm sidecar or native offerings from public cloud providers, like Azure Key Vault.

- ❗ Updates to config maps or secrets reflects in pods automatically applies only to configuration or secrets mounted to pods as volumes
- ❗ If you would like to reflect changes in environmental variables injected from configs or secrets, manual pod restart is required

## Using Environment Variables for configuration
❗Environment variables fit well into distributed systems. They are easy to define, and they are portable. They are the ideal choice for configuration mechanism of new applications.

![image](https://user-images.githubusercontent.com/33947539/141255772-7751813a-6a65-4378-9ac3-4a86a0ab8c4f.png)

To check the environment variables, we will need to execute a command "inside" of the Pod using **kubectl exec** :
![image](https://user-images.githubusercontent.com/33947539/141256217-a4b29421-ae5e-48e8-a756-1585702c954e.png)

in some cases, the configuration might be too complex for environment variables. In such situations, we might need to fall back to files (hopefully YAML). When those cases are combined with legacy applications which are almost exclusively using file-based configuration, it is evident that we cannot rely only on environment variables.

## Getting Started with ConfigMaps
- ConfigMaps allow us to keep configurations separate from application images. Such separation is useful when other alternatives are not a good fit.
- ConfigMap allows us to “inject” configuration into containers. The source of the configs can be files, directories, or literal values. The destination can be files or environment variables.
- ❗**ConfigMap takes a configuration from a source and mounts it into running containers as a volume.**

![image](https://user-images.githubusercontent.com/33947539/141256711-d8e82684-e3f8-44de-9727-0f5faefa4b04.png)

## Creating ConfigMap from manifest file:
It’s possible to create a ConfigMap along with the configuration data stored as key-value pairs in the data section of the definition.
![image](https://user-images.githubusercontent.com/33947539/141665850-4cbc8d31-b8ea-44d7-ae08-dc499af98c05.png)
![image](https://user-images.githubusercontent.com/33947539/141665864-50f4162b-5d7f-46b3-a853-1dd56eddb7c1.png)

In the above manifest:

the ConfigMap named simpleconfig contains two pieces of (key-value) data — hello=world and foo=bar
simpleconfig is referenced by a Pod (pod2; the keys hello and foo are consumed as environment variables HELLO_ENV_VAR and FOO_ENV_VAR respectively.
Note that we have included the Pod and ConfigMap definition in the same YAML separated by a ---

#### How to check for config variables inside pod
:point_right: $ kubectl exec pod2 -it -- env | grep _ENV_

FOO_ENV_VAR=bar
HELLO_ENV_VAR=world


## Configuration data as files
- Another interesting way to consume configuration data is by pointing to a ConfigMap in the spec.volumes section of your Deployment or Pod spec.
- ConfigMap is useless by itself. It is yet another Volume which, like all the others, needs a mount.
- volumes are a way of abstracting your container from the underlying storage system e.g. it could be a local disk or in the cloud such as Azure Disk, GCP Persistent Disk etc.

![image](https://user-images.githubusercontent.com/33947539/141666673-f7a9f18e-68bd-4f31-8521-2e78191b870c.png)
![image](https://user-images.githubusercontent.com/33947539/141666693-8cfd6e7c-3820-4417-8a3f-2297392f7593.png)

> In the above spec, pay attention to the **spec.volumes section** — notice that it refers to an existing ConfigMap. 
> Each key in the ConfigMap is added as a file to the directory specified in the spec **i.e. spec.containers.volumeMount.mountPath** and the value is nothing but the contents of the file.

## Configurations as Json
![image](https://user-images.githubusercontent.com/33947539/141666965-87dfb82e-af47-4249-9192-ac222a3a87f4.png)
![image](https://user-images.githubusercontent.com/33947539/141666974-0f564739-21d3-45f3-b2eb-bfa226109799.png)

## Using --from-literal to seed config data
👉 kubectl create configmap config4 --from-literal=foo_env=bar --from-literal=hello_env=world

## Creating a ConfigMap from a Directory
👉 kubectl create cm my-config --from-file=cm

We created my-config ConfigMap with the directory cm. Let’s describe it, and see what’s inside.
👉 kubectl describe cm my-config

The output is as follows (content of the files is removed for brevity).
![image](https://user-images.githubusercontent.com/33947539/141667724-03198f15-aab3-49d9-84e6-e38db9f677a3.png)

👉 kubectl create -f cm/alpine.yml

> Run the below command separately after the "alpine" container is created

👉 kubectl exec -it alpine --ls /etc/config

> The output of the latter command is as follows:

alpine-env-all.yml alpine.yml      prometheus-conf.yml
alpine-env.yml     my-env-file.yml prometheus.yml

## Reference
- https://github.com/vfarcic/k8s-specs.git
- https://betterprogramming.pub/how-to-use-kubernetes-secrets-for-storing-sensitive-config-data-f3c5e7d11c15
