# Kubernetes setup

<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/dao-duc-tung/k8s-setup">
    <img src="assets/banner.png" alt="Logo" width="300" height="100">
  </a>

  <h3 align="center">Kubernetes Setup</h3>
</p>

<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#introduction">Introduction</a></li>
    <li><a href="#install-k8s">Install k8s</a></li>
    <li><a href="#k8s-tools">k8s tools</a></li>
    <li><a href="#k8s-common-resources">k8s common resources</a></li>
    <li><a href="#kubectl-common">kubectl common</a></li>
    <li><a href="#eks-common">EKS common</a></li>
    <li><a href="#eks-workshop">EKS workshop</a></li>
    <li><a href="#install-kubeflow">Install Kubeflow</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>

## Introduction

This repository describes step by step about how to install Kubernetest (k8s) and its common tools. For some tools, this repository also guides you through their most basic usage.

## Install k8s

See [k8s_install.md](k8s_install.md). This document describes steps to install k8s and setup helper tools.

1. Personal and Production clusters
1. Install `kubectl`, `kubeadm`, `kubelet`
1. Init k8s cluster
1. Install common tools: `kubectl` bash completion, k8s dashboard, Ingress controller, Ingress type `LoadBalancer`, metrics server.

## k8s tools

See [k8s_tools.md](k8s_tools.md). This document describes steps to install and use some tools in k8s.

1. `krew`, `tree`, `ctx`, `ns`
1. Service mesh: Linkerd
1. Helm
1. Kustomize
1. Sniff
1. Skaffold
1. Prometheus

## k8s common resources

See [k8s_common.md](k8s_common.md). This document describes how to work with common k8s resources including, but not limited to, `Pod`, `Service`, `Deployment`, `ConfigMap`, `Secret`, `Network Policy`, Controlling access, Role-based access control (RBAC), and Custom Resource Definition (CRD).

## kubectl common

See [kubectl_common.md](kubectl_common.md). This document describes common `kubectl` commands related to k8s' resources including, but not limited to, `Namespace`, `Pod`, `Service`, `Deployment`, and `ConfigMap`.

This document also describes some common processes as below.

- Debug running pod

## EKS common

See [eks_common.md](eks_common.md). This document describes several steps to use Elastic Kubernetes Service (EKS) for beginner level.

1. Setup EKS
1. Create nodegroups
1. Deploy resources
1. Work with Storage Class and Reclaim policy
1. Network Policy
1. Ingress controller
1. Map IAM user with k8s user
1. Prometheus

## EKS workshop

See [eks_workshop.md](eks_workshop.md). This document desribes the content and keywords of the [eksworkshop](https://www.eksworkshop.com/) for keyword searching convenience purpose.

## Install Kubeflow

See [kubeflow_install.md](kubeflow_install.md). This document describes steps to install Kubeflow. This is currently a work in progress.

## License

Distributed under the MIT License. See [LICENSE](LICENSE) for more information.

## Contact

Tung Dao - [LinkedIn](https://www.linkedin.com/in/tungdao17/)

Project Link: [https://github.com/dao-duc-tung/k8s-setup](https://github.com/dao-duc-tung/k8s-setup)
