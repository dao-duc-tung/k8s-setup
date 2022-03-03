# Kubernetes concepts

## Pod

```bash
kubectl get pod -o wide
# IP column: shows the IP assigned to each pod
```

## Service

```bash
kubectl get svc
# CLUSTER-IP column: shows the IP of each Service
# PORT(S) column: shows the port that each service is listening on

kubectl describe svc <svc-name>
# IP field: The assigned IP of the service, same as column CLUSTER-IP of 'kubectl get svc'
# Port field: Same as column PORT(S) of 'kubectl get svc'
# Endpoints: List of IPs of all the pods linked to this service
```
