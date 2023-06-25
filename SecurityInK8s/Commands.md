## How to identify the authorization modes configured on the cluster?

```yaml
kubectl describe pod kube-apiserver-controlplane -n kube-system
```
look for --authorization-mode.

## Commands to create Roles and Role Binding 

```yaml
To create a Role:- kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods

To create a RoleBinding:- kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user
```

