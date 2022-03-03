# Kubernetes dashboard using kubeadm

```bash
# Install dashboard. Ref: https://github.com/kubernetes/dashboard#install
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml

# Get token to login
# Note that the default created service account 'kubernetes-dashboard' doesn't have permission to read information in the dashboard, so we need to create a new service account.

# Create a new service account in namespace 'default'. Ref: https://www.replex.io/blog/how-to-install-access-and-add-heapster-metrics-to-the-kubernetes-dashboard
kubectl create serviceaccount dashboard-admin-sa
# Bind the service account to the cluster-admin role
kubectl create clusterrolebinding dashboard-admin-sa --clusterrole=cluster-admin --serviceaccount=default:dashboard-admin-sa
# List secrets in namespace 'default'
kubectl get secret
# Find the secret with name 'dashboard-admin-sa-token-xxxxx'
# Get token
kubectl describe secret dashboard-admin-sa-token-xxxxx
# Copy the content of the token

# Now, access the dashboard at
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
# Paste the copied token to login
```
