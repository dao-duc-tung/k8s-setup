# Kubernetes

## Part 1: Install kubectl, kubeadm, kubelet

[General instruction from Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

```bash
# Install docker

# Verify containerd service. Ref: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime
sudo systemctl status containerd

# Disable swap permanently. Ref: https://www.geeksforgeeks.org/how-to-permanently-disable-swap-in-linux/
sudo swapoff -a
vi /etc/fstab # to comment out the line with swap
sudo reboot
free -h # Verify swap = 0G

# Install kubectl. Ref: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install kubelet kubeadm. Ref: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
# Verify version
kubectl version # 1.23.4
kubeadm version # 1.23.4
kubelet --version # 1.23.4

# Config cgroup driver for docker (definition: https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cgroup-drivers)
# Open or create /etc/docker/daemon.json, add in
# Ref: https://stackoverflow.com/a/68722458
{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
# Then run
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet
# Verify
docker info # Verify cgroup Driver = systemd
systemctl docker # Verify Actice = active
systemctl kubelet # Verify Actice = active
```

## Part 2: Init Kubernetes cluster

[General instruction from Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

```bash
# Init k8s cluster: Ref: Section Cluster Creation at https://medium.com/@andrei.benea/setting-up-a-kubeflow-deployment-on-a-kubernetes-cluster-f1f5dc35d46a
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
# If getting error, run this to revert, then run the above cmd again
sudo kubeadm reset

# Create config file
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Set iptables value to 1 (Flannel's requirement)
sudo sysctl net.bridge.bridge-nf-call-iptables=1

# Install Flannel as the Pod netword add-on
# Go to this link to copy the content to a file in your local machine: https://github.com/flannel-io/flannel/blob/v0.16.3/Documentation/kube-flannel.yml
vi kube-flannel.yml # paste content into this file
# Using github link won't work
kubectl apply -f kube-flannel.yml

# Schedule Pods on the control-plane node. Ref: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#control-plane-node-isolation
kubectl taint nodes --all node-role.kubernetes.io/master-

# Verify all pods, services, etc
sudo kubectl get all -A
# Verify nodes
sudo kubectl get nodes
```

## Part 3: Install common tools

```bash
# Install krew. Ref: https://krew.sigs.k8s.io/docs/user-guide/setup/install/

# Install tree
kubectl krew install tree
# To use tree, enable metrics server. Refer to kubeadm_metrics_server.md
# Use tree to show all the child components of a deployment
kubectl tree deploy <deployment-name>

# Install ctx, ns
kubectl krew install ctx
kubectl krew install ns
# List down all context
kubectl ctx
# List down all ns
kubectl ns
# Get current ns
kubectl ns -c
```
