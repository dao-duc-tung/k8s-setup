# Kubectl common processes

```bash
# List all k8s resources, with shortnames, api version, etc
kubectl api-resources
kubectl api-resources | grep apps
kubectl api-resources | grep apps | less

# Get common resources
kubectl get pod -A
kubectl get svc -A
kubectl get deploy -A
kubectl get ing -A
kubectl get ds -A
kubectl get apiservice
```
