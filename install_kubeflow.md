# Kubeflow

[General instruction from a Medium post](https://medium.com/@andrei.benea/setting-up-a-kubeflow-deployment-on-a-kubernetes-cluster-f1f5dc35d46a)

## Part 1: Install kustomize

[We need a compatible kustomize version for kubeflow.](https://github.com/kubeflow/manifests#installation).

```bash
# Install kustomize. Ref: https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/
# Check released versions here: https://github.com/kubernetes-sigs/kustomize/tags.
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" > 3.10.0

# Move binary file to bin
mv kustomize /usr/bin

# Verify
kustomize version
```

## Part 2: Install Kubeflow

## Other links

[Kubeflow install tutorial 01](https://medium.com/@andrei.benea/setting-up-a-kubeflow-deployment-on-a-kubernetes-cluster-f1f5dc35d46a)
[Kubeflow install tutorial 02](https://gist.github.com/muka/f91aa4afbbbe5cfb7cbbb7e19109b896)
[Kubeflow install tutorial - Part 1](https://thenewstack.io/kubeflow-where-machine-learning-meets-the-modern-infrastructure/)
[Kubeflow install tutorial - Part 2](https://thenewstack.io/tutorial-install-kubernetes-and-kubeflow-on-a-gpu-host-with-nvidia-deepops/)
[Kubeflow install tutorial - Part 3](https://thenewstack.io/a-closer-look-at-kubeflow-components/)
[Kubeflow install tutorial - Part 4](https://thenewstack.io/how-i-built-an-on-premises-ai-training-testbed-with-kubernetes-and-kubeflow/)
[Tutorial: Build Custom Container Images for a Kubeflow Notebook Server](https://thenewstack.io/tutorial-build-custom-container-images-for-a-kubeflow-notebook-server/)
[Configure Storage Volumes for Kubeflow Notebook Servers](https://thenewstack.io/tutorial-configure-storage-volumes-for-kubeflow-notebook-servers/)
