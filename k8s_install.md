# Kubernetes

This document describes steps to install k8s and setup helper tools.

## Part 0: Personal and Production clusters

### Personal cluster

- [minikube](https://minikube.sigs.k8s.io/docs/)
- Kubernetes through Docker desktop
- [kind](https://kind.sigs.k8s.io/): design for k8s team own internal development and testing. It's meant to run in the k8s CI system.
- [microk8s](https://microk8s.io/docs)
- [k3s](https://rancher.com/docs/k3s/latest/en/)

**Note**: personal cluster is likely to be 2 or 3 versions, or 6-month ahead of the production cluster.

### Production cluster

- Self-mananed (DIY): setup everything from scratch. Check out [Kubernetes The Hard Way by Kelsey Hightower]()
- Managed install: deploy cluster on a Linux system. Eg: [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
- Managed deploy: deploy cluster on cloud. Eg: [kOps](https://kops.sigs.k8s.io/)
- Fully-managed: cloud provider installs and operates k8s for you with the help of automated scripts and experts. Eg: GKE, AKS, EKS, IKS.

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

### Helper tools

```bash
# Install jq, envsubst (from GNU gettext utilities) and bash-completion
sudo yum -y install jq gettext bash-completion moreutils
# Install yq for yaml processing
echo 'yq() {
  docker run --rm -i -v "${PWD}":/workdir mikefarah/yq "$@"
}' | tee -a ~/.bashrc && source ~/.bashrc
# Enable kubectl bash_completion
kubectl completion bash >>  ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
```

### Kubernetes dashboard

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

### Ingress using kubeadm

[Ingress document](https://kubernetes.io/docs/concepts/services-networking/ingress/)

#### Install

```bash
# Install nginx ingress controller using yaml. Ref: https://kubernetes.github.io/ingress-nginx/deploy/
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/cloud/deploy.yaml

# Local testing. Ref: https://kubernetes.github.io/ingress-nginx/deploy/
# Create deployment
kubectl create deployment demo --image=httpd --port=80
# Create service to expose port 80
kubectl expose deployment demo
# Create ingress with rule to point to port 80 of "demo" and class=nginx
# because the above nginx ingress controller uses the IngressClass named "nginx"
kubectl create ingress demo-localhost --class=nginx \
  --rule=localdev.me/*=demo:80

# Test
# Forward port 80 from ingress ngxin controller to localhost:8080
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
curl http://localdev.me:8080/

# Cleanup
kubectl delete ing demo-localhost
kubectl delete svc demo
kubectl delete deployment demo
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/cloud/deploy.yaml
kubectl
# Check the section 'Useful tools' to delete the namespace 'ingress-nginx'
```

#### Customize ingress

Create `green.yaml` with content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: green
  labels:
    app: blue-green
    colour: green
spec:
  containers:
    - name: green # This image listens on port 8080
      image: docker.io/mtinside/blue-green:green
---
apiVersion: v1
kind: Service
metadata:
  name: green
spec:
  ports:
    - port: 80 # Incoming port
      targetPort: 8080 # Map incoming port to this port
  selector:
    app: blue-green
    colour: green
```

Create `nginx.yaml` to be the default backend for ingress

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.18.0
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: nginx
```

Create `ingress.yaml` with content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx # important
  defaultBackend:
    service:
      name: nginx
      port:
        number: 80
  rules:
    - host: "localdev.me"
      http:
        paths:
          - path: /green
            pathType: Prefix
            backend:
              service:
                # Forward requests from any port to port 80 of 'green' service
                name: green
                port:
                  number: 80
```

Test:

```bash
# Create resources
kubectl apply -f .
# Check
kubectl describe ing minimal-ingress

# Test 1
# Forward incoming requests at localhost:8080 to ingress ngxin controller:80
# nginx ingress controller listens on port 80
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
curl http://localdev.me:8080/green

# Test 2
# Forward incoming requests at localhost:8080 to pod green at port 8080
kubectl port-forward pod/green 8080:8080 # pod green is listening on port 8080
curl localhost:8080 # This should return green page

# Test 3
# Forward incoming requests at localhost:8080 to service green at port 80
kubectl port-forward svc/green 8080:80 # service green is listening on port 80
curl localhost:8080 # This should return green page

# Cleanup
kubectl delete -f .
```

#### Expose ingress controller in local k8s

When setting up the local k8s, there's no `LoadBalancer` service type provided. In order to expose ingress controller in local k8s, we need to provide a `LoadBalancer` service type by using [MetalLB](https://metallb.universe.tf/concepts/). Check [this](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/#a-pure-software-solution-metallb) for more details.

```bash
# Install MetalLB
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
```

Create `metallb-config.yaml` with content:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.121.177.40-10.121.177.45
```

_Note_: Assume your k8s cluster is running on a machine with IP `10.121.177.31`. You should change this line `- 10.121.177.40-10.121.177.45` to the range of the available IPs in your LAN where you setup the k8s cluster.

```bash
# Create metallb configmap
kubectl apply -f metallb-config.yaml
# Verify ingress
kubectl get svc -A # The External IP of ingress-nginx-controller should not be <pending> anymore
# Wait for few seconds
kubectl get ing # The address of 'minimal-ingress' should be equal to the ingress-nginx-controller's external IP
```

Add this line to `/etc/hosts`:

```bash
10.121.177.40 localdev.me
```

```bash
# Verify that the ingress controller is directing traffic
curl 10.121.177.40 # This should return nginx page
curl 10.121.177.40/green # This should return 404 Not found
curl localdev.me # This should return nginx page
curl localdev.me/green # This should return green page
# Open browser in another machine in LAN with IP 10.121.177.58, access 10.121.177.40, it should show nginx page

# Cleanup
# Revert change in /etc/hosts

# Delete metallb
kubectl delete -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
kubectl delete -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml # Press ctrl+C to exit
# Check the section 'Useful tools' to delete the namespace 'metallb-system'
```

#### Useful SSH for forwarding

```bash
# Forward requests via SSH
# Assume you are working on machine A with IP 10.121.177.58/28
# and you're running k8s cluster in machine B with IP 10.121.177.31/28
# You want to forward incoming requests from 10.121.177.58:8888 to 10.121.177.31:8080 via SSH
# In machine A, run:
ssh -N -L 8888:127.0.0.1:8080 <username>@10.121.177.31
# By doing this, now you can access http://localdev.me:8080/green from machine A
# Ref: https://www.tecmint.com/create-ssh-tunneling-port-forwarding-in-linux/
# -L flag: defines the port forwarded to the remote host and remote port
# -N flag: do not execute a remote command, you won't get a shell in this case

# Delete 'Terminating' namespace
sudo apt install jq
kubectl proxy
kubectl get ns delete-me-ns -o json | jq '.spec.finalizers=[]' | curl -X PUT http://localhost:8001/api/v1/namespaces/delete-me-ns/finalize -H "Content-Type: application/json" --data @-

# Delete 'Terminating' pod
kubectl delete pod delete-me-pod -n delete-me-ns --grace-period=0 --force
# Double check docker containers to see if it still runs and stop it
docker ps
docker container stop <container_id>
```

### Metrics server using kubeadm

In your local cluster, if you just install [metrics server](https://github.com/kubernetes-sigs/metrics-server) by running this command

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

you will most likely get this error

```bash
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get pods.metrics.k8s.io)
```

The reason is because kubelet certificate needs to be signed by cluster Certificate Authority (or disable certificate validation by passing `--kubelet-insecure-tls` to Metrics Server). This is stated in the [requirements of the metrics server](https://github.com/kubernetes-sigs/metrics-server#requirements). Note that, for cloud based cluster, there's no need to do this.

To fix this, we download the yaml file from:

```bash
https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Then add the flag `--kubelet-insecure-tls` to the argument list of the metrics-server command. It looks like:

```yaml
containers:
  - args:
      - --cert-dir=/tmp
      - --secure-port=4443
      - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      - --kubelet-use-node-status-port
      - --metric-resolution=15s
      - --kubelet-insecure-tls # Add this
    image: k8s.gcr.io/metrics-server/metrics-server:v0.6.1
```

Wait for few seconds, and verify:

```bash
kubectl top pods
kubectl top nodes
```

To troubleshoot/check error of the metrics server, run:

```bash
kubectl describe apiservice v1beta1.metrics.k8s.io
# Check the Status field
# Note: name v1beta1.metrics.k8s.io get from the above yaml file, in APIService component
```
