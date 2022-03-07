# Kubernetes tools

## Krew, tree, ctx, ns

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

## Service mesh: Linkerd

- [Kubernetes Service Mesh: A Comparison of Istio, Linkerd, and Consul](https://platform9.com/blog/kubernetes-service-mesh-a-comparison-of-istio-linkerd-and-consul/)
- [Linkerd Service mesh](https://linkerd.io/what-is-a-service-mesh/)

```bash
# Install service mesh linkerd. Ref: https://linkerd.io/2.11/getting-started/#
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | sh
# Add linkerd to PATH
linkerd version
linkerd check --pre
linkerd install | kubectl apply -f -
linkerd check
# Install linkerd dashboard
linkerd viz install | kubectl apply -f -
linkerd viz check
linkerd viz dashboard

# Run this to check how linkerd inject YAML
cat <yaml-file> | linkerd inject --manual -

# Instead, run this to enable linkerd injection every time we deploy a pod in namespace 'default'
# linkerd intercepts the processing of creating the pod and inject some YAML into the pod's yaml
# before passing to k8s
kubectl annotate namespace default linkerd.io/inject=enabled
kubectl annotate namespace default linkerd.io/inject=disabled --overwrite

# Uninstall
linkerd viz uninstall | kubectl delete -f -
linkerd uninstall | kubectl delete -f -
```

## Helm

- [Helm document](https://helm.sh/docs/)
- Helm is for the services that you own where you can control everything (YAML, resources, environments between services, etc). People usually just make one yaml template for a service, and have different values.yaml for different services.

```bash
# Install Helm. Ref: https://helm.sh/docs/intro/install/
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Setup helm folder
helm
│   Chart.yaml
│   values-dev.yaml
│   values-prod.yaml
│
└───templates
        deployment.yaml
        ingress.yaml
        service.yaml
```

```yaml
# Chart.yaml
apiVersion: v2
name: blue-green
description: An a/b test
version: 1.0.0 # chart version
---
# values-dev.yaml
host: dev.example.com
tls:
  enabled: false
---
# values-prod.yaml
host: prod.example.com
tls:
  enabled: true
---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: blue-green
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    {{- if .Values.tls.enabled }}
    cert-manager.io/cluster-issuer: letsencrypt-prod
    {{- end }}
spec:
  ingressClassName: nginx
  rules:
    - host: {{ .Values.host }}
      http:
        paths:
          - path: /blue-green
            pathType: Prefix
            backend:
              service:
                name: blue-green
                port:
                  number: 80
  {{- if .Values.tls.enabled }}
  tls:
    - hosts:
        - {{ .Values.host }}
      secretName: blue-green-prod-tls
  {{- end }}
```

```bash
# Merge values with templates
helm template -f values-dev.yaml .
# Merge and install (apply)
helm install -f values-dev.yaml blue-green . # blue-green is name
# Check svc
kubectl get svc
# Remove service.yaml, and run
helm upgrade -f values-dev.yaml blue-green .
# Check svc
kubectl get svc
# Helm will remove the service, because it tracks objects in history

# Check history
helm history blue-green
# Helm supports rollback to the previous revision

# Delete resources
helm delete blue-green
```

## Kustomize

- [Kustomize document](https://kubectl.docs.kubernetes.io/)
- Kustomize is for services you don't own (3rd-party apps). They usually provide large reference YAMLs and you want to make a few little tweaks. You want to use Kustomize to write your patches, while letting the 3rd-party apps maintain the base YAMLs.

```bash
# Kustomize is already installed with kubectl

# Setup kustomize folder
kustomize
├───base
│       deployment.yaml
│       ingress.yaml
│       kustomization.yaml
│       service.yaml
│
└───overlays
    └───prod
            ingress-patch-hostname.yaml
            ingress.yaml
            issuer-le-prod.yaml
            issuer-le-staging.yaml
            kustomization.yaml
```

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

commonLabels:
  app: blue-green

namespace: blue-green

resources:
  - deployment.yaml
  - service.yaml
  - ingress.yaml
---
# base/ingress.yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: blue-green
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: dev.example.com
      http:
        paths:
          - path: /blue-green
            backend:
              serviceName: blue-green
              servicePort: 80
---
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# These apply to the new resources in this overlay; the base's settings don't apply here.
# If these are different to the base, they're override it
commonLabels:
  app: blue-green

# ditto commonLabels
namespace: blue-green

bases:
  - ../../base

resources:
  - issuer-le-staging.yaml
  - issuer-le-prod.yaml

patchesStrategicMerge:
  - ingress.yaml
patchesJson6902:
  - target:
      group: networking.k8s.io
      version: v1beta1
      kind: Ingress
      name: blue-green
    path: ingress-patch-hostname.yaml
---
# overlays/prod/ingress.yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: blue-green
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts:
        - prod.example.com
      secretName: blue-green-prod-tls
---
# overlays/prod/ingress-patch-hostname.yaml
- op: replace
  path: /spec/rules/0/host
  value: prod.example.com
```

```bash
# generate yaml in base
kubectl kustomize base
# apply
kubectl apply -K base
# generate yaml in overlays/prod
kubectl kustomize overlays/prod
```

## Sniff

This `sniff` tool will open Wireshark software which is meant to use with GUI. Wireshark will help to track all the traffic in/out of a container in a pod.

```bash
# Install Wireshark first
# Install sniff
kubectl krew install sniff
# sniff a container
kubectl sniff <pod-name> -n <namespace> -c <container-name>
```

## Skaffold

To develop quickly and efficiently, we don't want to go through the whole loop of developing a program including editing it, building it, making a Docker image, pushing that, rolling out, restart on the deployment, change the label in deployment, wait for the deployment to roll something out.

Instead, [Skaffold](https://skaffold.dev/docs/) handles the workflow for building, pushing, and deploying your application, and provides building blocks for creating CI/CD pipelines. This enables you to focus on iterating on your application locally while Skaffold continuously deploys to your local or remote Kubernetes cluster.

Ref: [Skaffold yaml syntax](https://skaffold-latest.firebaseapp.com/docs/references/yaml/)

```bash
# Install Skaffold
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 && sudo install skaffold /usr/local/bin/
```

### Demo

This is a simple example based on:

- **building** a single Go file app and with a multistage `Dockerfile` using local docker to build
- **tagging** using the default tagPolicy (`gitCommit`)
- **deploying** a single container pod using `kubectl`

```yaml
# skaffold.yaml
apiVersion: skaffold/v2beta26
kind: Config
build:
  artifacts:
    - image: skaffold-example
  local:
    push: false # Don't push image to registry
deploy:
  kubectl:
    manifests:
      - k8s-*
---
# k8s-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: getting-started
spec:
  containers:
    - name: getting-started
      image: skaffold-example
```

```dockerfile
# Dockerfile
FROM golang:1.12.9-alpine3.10 as builder
COPY main.go .
RUN go build -o /app main.go

FROM alpine:3.10
# Define GOTRACEBACK to mark this container as using the Go language runtime
# for `skaffold debug` (https://skaffold.dev/docs/workflows/debug/).
ENV GOTRACEBACK=single
CMD ["./app"]
COPY --from=builder /app .
```

```go
// main.go
package main

import (
	"fmt"
	"time"
)

func main() {
	for {
		fmt.Println("Hello world!")

		time.Sleep(time.Second * 1)
	}
}
```

Run:

```bash
# continuous build & deploy on code changes
skaffold dev
# check pod in a new terminal
kubectl get pod
# change main.go code, skaffold detects the change and redeploy the pod
# check pod
kubectl get pod
```

## Prometheus

### Install

There're few ways to install Prometheus:

- As a single binary running on your hosts
- As a Docker container
- Using Prometheus Operator: not entire monitoring stack
- Using kube-prometheus: provision an entire monitoring stack
- Using Helm chart: provides similar feature set to kube-prometheus

Some tutorials:

- [2022-01-28 - How to Setup Prometheus Monitoring On Kubernetes Cluster](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/)
- [2021-08-31 - Prometheus Definitive Guide Part III - Prometheus Operator](https://www.infracloud.io/blogs/prometheus-operator-helm-guide/)
- [2021-02-16 - Kubernetes monitoring with Prometheus, the ultimate guide](https://sysdig.com/blog/kubernetes-monitoring-prometheus/)
- [Prometheus document](https://prometheus.io/docs/prometheus/latest/getting_started/)

```bash
# Install using Prometheus Operator
# Ref: https://github.com/prometheus-operator/prometheus-operator
# Use 'create' instead of 'apply'
# Large annotation size is the issue for 'kubectl apply' since it stores last applied config
kubectl create -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml

# Install using kube-prometheus: Follow the instruction in its Github
# Ref: https://github.com/prometheus-operator/kube-prometheus
# Tried but got error Insufficient CPU (not common) for pod prometheus-k8s-0 and prometheus-k8s-0
# maybe this can work on another machine

# Install using Helm chart. Ref: https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
# Replace '[RELEASE_NAME]' by 'prometheus'
helm install [RELEASE_NAME] prometheus-community/kube-prometheus-stack
# Clean resources
helm delete [RELEASE_NAME]
```

### Access GUI

```bash
# For installation using Helm chart
# Access Prometheus dashboard
kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090
# Access Alert Manager dashboard
kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9093
# Access Grafana dashboard, username: admin, password: prom-operator
kubectl port-forward svc/prometheus-kube-prometheus-prometheus 3000:80
```

## Other tools

- [Telepresence](https://www.telepresence.io/docs/latest/quick-start/) helps to swap environments for fast development. Eg: local and remote environments, local and prod environments.
- [Okteto](https://www.okteto.com/docs/welcome/overview/) offers you a way to deploy and develop applications directly on the cloud.
