# Kubectl common processes

## General

- [Configuration Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
- [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Ports and Protocols](https://kubernetes.io/docs/reference/ports-and-protocols/)
- [Recommended Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)
- [guestbook-all-in-one.yaml](https://github.com/kubernetes/examples/blob/master/guestbook/all-in-one/guestbook-all-in-one.yaml)

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

## Deployments

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
