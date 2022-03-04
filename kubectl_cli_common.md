# Kubectl common cli

## General

- [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

```bash
# List all k8s resources, with shortnames, api version, etc
kubectl api-resources
kubectl api-resources | grep apps
kubectl api-resources | grep apps | less

# Get common resources
kubectl get pod
kubectl get svc
kubectl get deploy
kubectl get ing
kubectl get job
kubectl get ep
kubectl get ds
kubectl get apiservice

# watch the change of deploy
watch kubectl get pod
watch kubectl get deploy
watch kubectl get rs

# Compare difference
kubectl diff -f <yaml-file>

# Show tree structure of deploy
kubectl tree deploy <deploy-name>
```

## Namespace

- [Namespace document](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- Namespace is just for grouping resources, like a folder. It can be useful when using with NetworkPolicy to grant permission for a group of resources.
- When deleting a namespace, all of its resources will be deleted also.
- Some resources can't be put in a namespace like nodes, persisten volumes, etc. They're called cluster resources.

```bash
# Create ns
kubectl create namespace team1
# permanently save the namespace for all subsequent kubectl commands in that context
kubectl config set-context --current --namespace=team1
# Use kubectl ctx to get contexts
# Use kubectl ns to get namespaces
```

## Pod

```bash
kubectl get pod -o wide
# IP column: shows the IP assigned to each pod
```

## Service

[Service document](https://kubernetes.io/docs/concepts/services-networking/service/)

```bash
kubectl get svc
# CLUSTER-IP column: shows the IP of each Service
# PORT(S) column: shows the port that each service is listening on

kubectl describe svc <svc-name>
# IP field: The assigned IP of the service, same as column CLUSTER-IP of 'kubectl get svc'
# Port field: Same as column PORT(S) of 'kubectl get svc'
# Endpoints: List of IPs of all the pods linked to this service
```

## Deployment

[Deployment document](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

```bash
# rollout
kubectl rollout status deploy/<deploy-name>
kubectl rollout pause deploy/<deploy-name>
kubectl rollout resume deploy/<deploy-name>
kubectl rollout history deploy/<deploy-name>
kubectl rollout undo deploy/<deploy-name>
kubectl rollout undo deploy/<deploy-name> --to-revision=<number>
# You cannot rollback a paused Deployment until you resume it.
```

## ConfigMap

[ConfigMap document](https://kubernetes.io/docs/concepts/configuration/configmap/)

```bash
# Create configmap from file by using imperative command, useful for big files
kubectl create configmap <configmap-name> --from-file=<path-to-file> --dry-run -o yaml > <filename>.yaml
```
