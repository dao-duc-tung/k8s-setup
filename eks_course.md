# Elastic Kubernetes Service (EKS)

This document describes several steps to use EKS for beginner level. This document is based on the course [Running Kubernetes on AWS (EKS) on Linkedin Learning](https://www.linkedin.com/learning/running-kubernetes-on-aws-eks/).

EKS fully manages the k8s cluster for us and provide EC2 or Fargate instances as worker nodes or pods, respectively. We can use both at the same time, in the same cluster.

- EC2
  - Compute as a Service
  - Supports > 4vCPU, 30GB per Pod, SSD/managed IOPS local storage, GPU
- Fargate
  - Container as a Service
  - Adds a profile (namespace + label) to map Pod -> ECS profile
  - No node management, usually no set scale limits, no need to scale out workers, only pay for active pods, not active node
  - Supports 0.5 - 4vCPU, no GPU
  - Container service needs to map to Farget CPU or memory tier. This mapping is based on largest sum of resources (init vs. non-init containers) and consumes 250m cores + 512Mi memory
- Conclusion: EC2 = Flexibility, Fargate = Less management

- Checkout [EKS Quickstart](https://aws-quickstart.github.io/quickstart-amazon-eks/) for users who are looking for a repeatable, customizable reference deployment for Amazon EKS using AWS CloudFormation

## Setup EKS

```bash
# Setup a cluster. Ref: https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html
# --name <cluster-name> --version <version> --without-nodegroup --with-oidc
eksctl create cluster
# Get cluster info
eksctl get clusters
# Check kubectl version in client side and server side (in EKS)
kubectl version

# Create kubectl's config file in '~/.kube/config' if it doesn't exist or we cannot get server side's info
eksctl utils write-kubeconfig --cluster <cluster-name>
# Get nodes
kubectl get node
```

## Create nodegroups

```bash
# Create a new node group, node group is a group of similar nodes
# --name: to track the node easier, if not provided, a random name will be given
# --node-ami-family: the AMI we want to use to create this node
# --nodes: total number of nodes, default = 2
eksctl create nodegroup --node-type m5.large --cluster <cluster-name> --name <node-group-name> --node-ami-family Bottlerocket
# Check nodegroups
eksctl get nodegroups --cluster <cluster-name>

# Create a new node group with auto-scaling group capabilities
# --asg-access:
eksctl create nodegroup --node-type m5.large --cluster <cluster-name> --name <node-group-name> --asg-access --nodes-min 1 --nodes-max 3
# Check nodegroups
eksctl get nodegroups --cluster <cluster-name>
# Config Scaling policies for this node group
# AWS console > EC2 > Auto Scaling Groups > Select the node group > Automatic scaling tab
# Create dynamic scaling policy

# Create a new node group with labels
eksctl create nodegroup --cluster <cluster-name> --name <node-group-name> --node-labels <label-key>=<label-value>
# Check labels
kubectl get node --show-labels

# Clean resources
eksctl delete nodegroup --cluster <cluster-name> <node-group-name>
```

## Deploy resources

This step deploy a Deployment and a Service. The Deployment has a node-affinity which selects only the node that has label nodetype=generalpurpose.

```yaml
# hostname.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hostname-v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hostname-v1
      version: v1
  template:
    metadata:
      labels:
        app: hostname-v1
        version: v1
    spec:
      containers:
        - image: rstarmer/hostname:v1
          imagePullPolicy: Always
          name: hostname
          resources:
            limits:
              cpu: 256m
              memory: 128Mi
      nodeSelector:
        nodetype: generalpurpose
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hostname-v1
  name: hostname-v1
spec:
  ports:
    - name: web
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: hostname-v1
```

```bash
# Label a node
kubectl label node <node-name> nodetype=generalpurpose
# Apply resources
kubectl apply -f hostname.yaml
# Check pod
kubectl get pod -o wide
# Clean resources
kubectl delete -f hostname.yaml
```

## Create a general purpose Storage Class with _Retain_ Reclaim policy

```bash
# Check the default storage class (SC) that eksctl created
kubectl get sc
# This default SC has Reclaim policy = Delete -> The storage will be deleted when Pod dies
# So we want to create another general purpose SC with 'Retain' reclaim policy
```

```yaml
# gp-retain.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: gp2-retain
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
mountOptions:
  - debug
```

```bash
# Create SC with 'retain' reclaim policy
kubectl apply -f gp-retain.yaml
# Check SC
kubectl get sc
```

## Create a _Retain_ Storage Class with IOPS version of EBS

```yaml
# fast-storage.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast-100
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "100" # speed: 100 iops/GB
reclaimPolicy: Retain
mountOptions:
  - debug
```

```bash
# Create SC with IOPS EBS
# EKS has already all of the internal bindings to create the Storage Class connection
# to the backend EBS storage system
kubectl apply -f fast-storage.yaml
# Check SC
kubectl get sc
```

## Create deployment with default _Delete_ Storage Classes

```yaml
# hostname-volume-reclaim.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hostname-volume
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hostname-volume
      version: v1
  template:
    metadata:
      labels:
        app: hostname-volume
        version: v1
    spec:
      volumes:
        - name: hostname-pvc
          persistentVolumeClaim:
            claimName: hostname-pvc
      containers:
        - image: rstarmer/hostname:v1
          imagePullPolicy: Always
          name: hostname
          volumeMounts:
            - mountPath: "/www"
              name: hostname-pvc
          resources:
            limits:
              cpu: 250m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hostname-volume
  name: hostname-volume
spec:
  ports:
    - name: web
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: hostname-volume
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hostname-pvc
spec:
  storageClassName: gp2
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```bash
# Create resources
kubectl apply -f hostname-volume-reclaim.yaml
# Check pvc, pv
kubectl get pvc
kubectl get pv
# Delete resources
kubectl delete -f hostname-volume-reclaim.yaml
# Check pvc, pv, there should be no pvc and pv
kubectl get pvc
kubectl get pv
```

## Create deployment with _Retain_ Storage Classes

```yaml
# hostname-volume-dont-reclaim.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hostname-volume
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hostname-volume
      version: v1
  template:
    metadata:
      labels:
        app: hostname-volume
        version: v1
    spec:
      volumes:
        - name: hostname-pvc
          persistentVolumeClaim:
            claimName: hostname-pvc
      containers:
        - image: rstarmer/hostname:v1
          imagePullPolicy: Always
          name: hostname
          volumeMounts:
            - mountPath: "/www"
              name: hostname-pvc
          resources:
            limits:
              cpu: 250m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hostname-volume
  name: hostname-volume
spec:
  ports:
    - name: web
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: hostname-volume
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hostname-pvc
spec:
  storageClassName: gp2-retain
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```bash
# Create resources
kubectl apply -f hostname-volume-dont-reclaim.yaml
# Check pvc, pv
kubectl get pvc
kubectl get pv
# Delete resources
kubectl delete -f hostname-volume-dont-reclaim.yaml
# Check pvc, pv, there should be 0 pvc and 1 pv
kubectl get pvc
kubectl get pv
# Delete pv
kubectl delete pv <pv-name>
# Double check
kubectl get pv

# Clean resources
kubectl delete -f fast-storage.yaml gp-retain.yaml
```

## Network Policy

In default EKS environment, the network is established based on AWS's VPC environment, and a driver that was written by AWS to support that interaction. If we want to apply the k8s Network Policy resource, we need another network interface. E.g: Calico service. Follow [this](https://docs.aws.amazon.com/eks/latest/userguide/calico.html) to install the Calico add-on in EKS.

```yaml
# hostname.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hostname-v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hostname-v1
      version: v1
  template:
    metadata:
      labels:
        app: hostname-v1
        version: v1
    spec:
      containers:
        - image: rstarmer/hostname:v1
          imagePullPolicy: Always
          name: hostname
          resources:
            limits:
              cpu: 256m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hostname-v1
  name: hostname-v1
spec:
  ports:
    - name: web
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: hostname-v1
---
# default-deny.yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny
  namespace: default
spec:
  podSelector:
    matchLabels: {}
---
# allow-hostname.yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  namespace: default
  name: allow-hostname
spec:
  podSelector:
    matchLabels:
      app: hostname-v1
  ingress:
    - from:
        - namespaceSelector:
            matchLabels: {}
```

```bash
# Create a deployment and its service
kubectl apply -f hostname.yaml
# Test connection sending from another pod, expect Success
kubectl run --image nixery.dev/shell/curl curl
kubectl exec -it curl -- curl --connect-timeout 5 http://hostname-v1/version/
# Apply default-deny
kubectl apply -f default-deny.yaml
# Test connection, expect Failed
kubectl exec -it curl -- curl --connect-timeout 5 http://hostname-v1/version/
# Apply allow-hostname
kubectl apply -f allow-hostname.yaml
# Test connection, expect Success
kubectl exec -it curl -- curl --connect-timeout 5 http://hostname-v1/version/

# Clean resources
kubectl delete pod curl
kubectl delete -f .
# Follow https://docs.aws.amazon.com/eks/latest/userguide/calico.html to remove the Calico add-on in EKS
```

## Ingress controller

Currently, the [kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx), which is an Ingress Controller, only supports legacy load balancer for AWS Network Load Balancer. Check [this](https://kubernetes.github.io/ingress-nginx/deploy/#aws) for more details. AWS provides the documentation on how to use [Network load balancing on Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/network-load-balancing.html) with [AWS Load Balancer Controller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html).

Check [this video](https://youtu.be/ZO5dUN18C70) for the AWS Load Balancer Controller overview. This is [its Github](https://github.com/kubernetes-sigs/aws-load-balancer-controller) and [its document](https://kubernetes-sigs.github.io/aws-load-balancer-controller/).

### Install AWS Load Balancer Controller (LBC)

To install AWS LBC, follow [this guide](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html). After installation, follow [this guide](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html).

```bash
# Create an IAM policy
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.0/docs/install/iam_policy.json

aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

# Create an IAM role
eksctl create iamserviceaccount --cluster=<cluster-name> --namespace=kube-system --name=aws-load-balancer-controller --attach-policy-arn=arn:aws:iam::<user-id>:policy/AWSLoadBalancerControllerIAMPolicy --override-existing-serviceaccounts --approve

# Install the AWS Load Balancer Controller
helm repo add eks https://aws.github.io/eks-charts
helm repo update

# Replace image.repository with desired one. Check here: https://docs.aws.amazon.com/eks/latest/userguide/add-ons-images.html
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<cluster-name> --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set image.repository=602401143452.dkr.ecr.ap-southeast-1.amazonaws.com/amazon/aws-load-balancer-controller --set image.tag="v2.4.0" --set region=<region-code> --set vpcId=<vpc-id>

# Troubleshoot using LBC logs
kubectl logs -n kube-system deployment.apps/aws-load-balancer-controller
# During the installation, I got livenessProbe failed, and had to recreate the cluster
# Otherwise, check this: https://aws.amazon.com/premiumsupport/knowledge-center/eks-resolve-failed-health-check-alb-nlb/

# upgrade to a newer chart when it becomes available
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master"
# Run above 'install' command but replace 'install' with 'upgrade'

# Clean resources
helm delete aws-load-balancer-controller -n kube-system
# Delete IAM policy
```

### Deploy a sample app

```bash
# Create resources
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.0/docs/examples/2048/2048_full.yaml
# Check ingress if it has Address
kubectl get ingress/ingress-2048 -n game-2048
# Go to the address to test
# Clean resources
kubectl delete -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.0/docs/examples/2048/2048_full.yaml
```

## Map IAM user with k8s user using aws-auth configmap

```bash
# Test permission with current eksadmin user, assume it is current user
kubectl get pod # expect success
# Check aws-auth configmap
kubectl get configmap aws-auth -n kube-system -o yaml
# Test permission with other username 'eksuser', assume it exists
export AWS_PROFILE=eksuser
kubectl get pod # expect failed

# Create IAM mapping, replace <k8s-username> by eksuser
eksctl create iamidentitymapping --cluster <cluster-name> --arn <user-arn> --username <k8s-username>
# Edit directly aws-auth configmap
kubectl edit configmap aws-auth -n kube-system -o yaml
```

```yaml
# Below username, userarn, add
groups:
  - system:masters
```

```bash
# Test permission with other username 'eksuser'
kubectl get pod # expect success
```

## Map IAM user with k8s user using RBAC

```yaml
# rbac.yaml
# eksuser is the namespace, k8s username, IAM username, role name, rolebinding name
apiVersion: v1
kind: Namespace
metadata:
  name: eksuser
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: eksuser
  namespace: eksuser
rules:
  - apiGroups: ["apps", ""]
    resources:
      ["deployments", "pods", "services", "pods/log", "pods/exec", "ingresses"]
    verbs: ["*"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: eksuser
  namespace: eksuser
subjects:
  - kind: User
    name: eksuser
roleRef:
  kind: Role
  name: eksuser
  apiGroup: rbac.authorization.k8s.io
```

```bash
# Create resources
kubectl apply -f rbac.yaml

# Test permission with other username 'eksuser'
kubectl get pod # expect failed
kubectl get pod -n eksuser # expect success

# Clean resources
kubectl delete -f rbac.yaml
```

## Prometheus

Follow [this tutorial](https://docs.aws.amazon.com/eks/latest/userguide/prometheus.html) to deploy Prometheus.

### Install

```bash
kubectl create namespace prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade -i prometheus prometheus-community/prometheus --namespace prometheus --set alertmanager.persistentVolume.storageClass="gp2",server.persistentVolume.storageClass="gp2"

# Verify
kubectl get pods -n prometheus

# Access dashboard
kubectl --namespace=prometheus port-forward deploy/prometheus-server 9090
```

### Deploy an app

Deploy hostname app with Envoy container as the extension. In the `annotations`, we specify Prometheus paths and forwarding connection to talk to the Prometheus endpoint. Because the hostname app doesn't generate Prometheus metrics, so we add Envoy container as the gate to collect traffic metrics. The request route looks like:

```bash
REQUEST --> Service:8080 --> 8080:Envoy:80 --> hostname:80
```

Below is the example of the hostname app. The Prometheus configurations in the `annotations` might be out-dated. The path and port might be different in the current version of Prometheus.

```yaml
# hostname-envoy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hostname-v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hostname-v1
      version: v1
  template:
    metadata:
      labels:
        app: hostname-v1
        version: v1
      annotations:
        prometheus.io/path: /stats/prometheus
        prometheus.io/port: "9901"
        prometheus.io/scrape: "true"
    spec:
      containers:
        - image: rstarmer/hostname:v1
          imagePullPolicy: Always
          name: hostname
          resources:
            limits:
              cpu: 256m
              memory: 128Mi
        - name: envoy
          image: opsani/envoy-sidecar:latest
          imagePullPolicy: Always
          env:
            - name: SERVICE_PORT
              value: "80"
            - name: LISTEN_PORT
              value: "8080"
            - name: METRICS_PORT
              value: "9901"
          ports:
            - containerPort: 8080
              name: service # service listener provided by Envoy proxy
            - containerPort: 9901
              name: metrics # metrics provided by Envoy
          resources:
            limits:
              cpu: 250m
              memory: 256Mi
            requests:
              cpu: 125m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hostname-v1
  name: hostname-v1
spec:
  ports:
    - name: web
      port: 80
      protocol: TCP
      targetPort: 8080
  selector:
    app: hostname-v1
```

### Clean resources

```bash
helm uninstall prometheus -n prometheus
```

## Clean resources

```bash
eksctl delete iamserviceaccount --cluster=<cluster-name> --namespace=<namespace> --name=aws-load-balancer-controller
eksctl delete cluster --name <cluster-name>
```
