# Kubectl common cli

This document describes common `kubectl` commands.

## General

- [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

```bash
# List all k8s resources, with shortnames, api version, etc
kubectl api-resources
kubectl api-resources | grep apps # to look up
kubectl api-resources | grep apps | less # to show less
kubectl api-resources | grep apps | less -S # to chop long lines

# Get common resources
kubectl get all
kubectl get pod
kubectl get svc
kubectl get deploy
kubectl get ing
kubectl get job
kubectl get ep
kubectl get ds
kubectl get apiservice
kubectl get crd

# debug pod error/status
kubectl describe pod <pod-name> -n <namespace>

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

# Add annotation to a namespace
# Eg: kubectl annotate namespace default linkerd.io/inject=enabled
kubectl annotate namespace <ns-name> <annotation>
```

## Pod

```bash
kubectl get pod -o wide
# READY column: shows number of ready containers/all containers
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
kubectl get deploy
# READY column: shows number of ready pods/all replicas

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

## Debug running pod

- [Debug running pod document](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/)

### 1. Debug with container exec

```bash
kubectl exec -it <pod-name> -- sh
# or attach to a pod
kubectl attach <pod-name> -it -c <container-name>
# List down all process
ps faux
```

### 2. Debug with an ephemeral debug container

[Ephemeral containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/) are useful for interactive troubleshooting when kubectl exec is insufficient because a container has crashed or a container image doesn't include debugging utilities (shell), such as with distroless images. One problem is that, like regular containers, you may not change or remove an ephemeral container after you have added it to a Pod.

```bash
# process-namespace is the name of the main process which runs in the pod, usually the main container name
kubectl debug <pod-name> -it --image=busybox --target=<process-namespace>
# check added ephemeral containers
kubectl describe pod <pod-name>
```

### 3. Debug using a copy of the pod

Sometimes Pod configuration options make it difficult to troubleshoot in certain situations. For example, you can't run kubectl exec to troubleshoot your container if your container image does not include a shell or if your application crashes on startup. In these situations you can use kubectl debug to create a copy of the Pod with configuration values changed to aid debugging.

#### 3.1. Copy a pod while adding a new container

Adding a new container can be useful when your application is running but not behaving as you expect and you'd like to add additional troubleshooting utilities to the Pod.

```bash
# -i flag causes 'kubectl debug' to attach to the new container by default
# you can prevent this by specifying --attach=false
# --share-processes allows the containers in this pod to see processes
# from the other containers in the pod
kubectl debug <pod-name> -it --image=ubuntu --share-processes --copy-to=<debug-pod-name>
# delete debugging pod
kubectl delete pod <debug-pod-name>
```

#### 3.2. Copy a pod while changing its command

Sometimes it's useful to change the command for a container, for example to add a debugging flag or because the application is crashing.

```bash
# To change the command of a specific container, you must specifying its name using --container
kubectl debug <pod-name> -it --copy-to=<debug-pod-name> --container=<container-name> -- sh
# delete debugging pod
kubectl delete pod <debug-pod-name>
```

#### 3.3. Copy a pod while changing container images

In some situations you may want to change a misbehaving Pod from its normal production container images to an image containing a debugging build or additional utilities.

```bash
# --set-image uses the same 'container_name=image' syntax as kubectl set image . *=ubuntu
# This changes the image of all containers to ubuntu
kubectl debug <pod-name> -it --copy-to-<debug-pod-name> --set-image=*=ubuntu
# delete debugging pod
kubectl delete pod <debug-pod-name>
```

### 4. Debug via a shell on the node

If none of these approaches work, you can find the Node on which the Pod is running and create a privileged Pod running in the host namespaces.

```bash
# To create an interactive shell on a node using kubectl debug:
kubectl debug node/<node-name> -it --image=ubuntu
# delete debugging pod
kubectl delete pod <debug-pod-name>
```

### Debug trick: Using nixery.dev image

```bash
# Instead of using busybox, ubuntu, etc with limited tools, use nixery.dev as the debug image
# This image weighs ~225MB
# the list of tools behind nixery.dev will be installed on the fly
kubectl debug <pod-name> -it --image=nixery.dev/shell/curl/wget/htop --target=<process-namespace>
kubectl debug <pod-name> -it --image=nixery.dev/shell/curl/wget/htop --share-processes --copy-to=<debug-pod-name>
```
