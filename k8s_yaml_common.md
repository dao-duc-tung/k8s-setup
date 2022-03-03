# Kubernetes common yaml

## General

- [Configuration Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
- [Ports and Protocols](https://kubernetes.io/docs/reference/ports-and-protocols/)
- [Recommended Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)
- [guestbook-all-in-one.yaml](https://github.com/kubernetes/examples/blob/master/guestbook/all-in-one/guestbook-all-in-one.yaml)

## Pod

## Service

## Deployment

[Deployment document](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

```yaml
# livenessProbe: to check health status
# This is used as the signal to restart the container
containers:
  - name: envbin
    image: mtinside/envbin:latest
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 1
      periodSeconds: 1
```

```yaml
# readinessProbe: to check ready status
# This is not used to restart the container, but to not assign IP to the pod
# Use case 1: when the container is not dead, but it's not ready due to some reasons
# e.g: lose connection with database/servers, something inside is broken temporarily
# This error will persist forever without restarting the container until we do something
# Use case 2: when the container takes time to initialize some tasks
containers:
  - name: envbin
    image: mtinside/envbin:latest
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 0
      periodSeconds: 1
      timeoutSeconds: 1
      successThreshold: 1
      failureThreshold: 1
```
