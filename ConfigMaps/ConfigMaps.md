To configure your apps in Kubernetes, you can use:
- Good old environment variables
- ConfigMap
- Secret

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

## Mounting the ConfigMap


## Injecting Configuration from a Single File:

