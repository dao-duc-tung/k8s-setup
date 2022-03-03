# Kubectl tree tool

```bash
# Install krew. Ref: https://krew.sigs.k8s.io/docs/user-guide/setup/install/

# Install tree
kubectl krew install tree

# Enable metrics server. Refer to kubeadm_metrics_server.md

# Use tree to show all the child components of a deployment
kubectl tree deploy <deployment-name>
```
