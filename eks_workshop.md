# EKS Workshop

This document desribes the content and keywords of the [eksworkshop](https://www.eksworkshop.com/) for keyword searching convenience purpose.

There's another interesting workshop related to DevOps which is [Blue/Green and Canary deployment for EKS and ECS](https://catalog.us-east-1.prod.workshops.aws/workshops/2175d94a-cd79-4ed2-8e7e-1f0dd1956a3a/en-US).

## [Beginner](https://www.eksworkshop.com/beginner/)

- [Deploy the k8s dashboard](https://www.eksworkshop.com/beginner/040_dashboard/)
- [Deploy the example microservices](https://www.eksworkshop.com/beginner/050_deploy/)
  - Deploy NodeJS backend, Crystal backend, Frontend
  - Find the ELB's Service address
  - Scale the backend services, frontend
- [Helm](https://www.eksworkshop.com/beginner/060_helm/)
  - Deploy nginx with helm
  - Deploy example microservices using helm
- [Health checks](https://www.eksworkshop.com/beginner/070_healthchecks/)
  - Configure liveness probe, readiness probe
- [Autoscaling Apps and Clusters](https://www.eksworkshop.com/beginner/080_scaling/)
  - Install Kube-ops-view
  - Configure and scale pods with Horizontal Pod AutoScaler (HPA)
  - Configure and scale cluster with Cluster Autoscaler (CA)
- [High-performance autoscaling with Karpenter](https://www.eksworkshop.com/beginner/085_scaling_karpenter/)
- [Intro to RBAC](https://www.eksworkshop.com/beginner/090_rbac/)
  - Map an IAM User to k8s
  - Create the Role and Binding
- [Using IAM Groups to manage k8s access](https://www.eksworkshop.com/beginner/091_iam-groups/)
  - Create IAM Roles, Groups, Users
  - Configure k8s RBAC, k8s Role access
- [IAM Roles for Service Accounts](https://www.eksworkshop.com/beginner/110_irsa/)
  - Create an OIDC identity provider
  - Create and specify an IAM Role for Service Account
- [Security groups for pods](https://www.eksworkshop.com/beginner/115_sg-per-pod/)
- [Secure EKS cluster with Network Policies](https://www.eksworkshop.com/beginner/120_network-policies/)
  - Create Network Policies using Calico
  - Calico enterprise usecases
- [Exposing a Service](https://www.eksworkshop.com/beginner/130_exposing-service/)
  - Connect apps with Services
  - Access, expose the Service
  - Ingress, Ingress controller
- [Assign pods to nodes](https://www.eksworkshop.com/beginner/140_assigning_pods/)
  - nodeSelector
  - Affinity, anti-affinity
  - Practical usecases
- [Use spot instances](https://www.eksworkshop.com/beginner/150_spotnodegroups/)
  - Add Spot managed node group
  - Spot configuration and lifecycle
- [Advanced VPC networking](https://www.eksworkshop.com/beginner/160_advanced-networking/)
  - Use 2nd CIDRs
- [Stateful containers using StatefulSets](https://www.eksworkshop.com/beginner/170_statefulset/)
  - Amazon EBS CSI Driver
  - Define Storage Class
  - Create ConfigMap, Services, StatefulSet
- [Deploy Bottlerocket nodes](https://www.eksworkshop.com/beginner/185_bottlerocket/)
- [Deploy microservices to EKS Fargate](https://www.eksworkshop.com/beginner/180_fargate/)
  - Create a Fargate profile
  - Setup the LB controller
  - Deploy pods to Fargate
  - Setup Ingress
- [Deploy Stateful microservices with Amazon FSx Lustre](https://www.eksworkshop.com/beginner/190_fsx_lustre/)
- [Deploy Stateful microservices with AWS EFS](https://www.eksworkshop.com/beginner/190_efs/)
  - EFS Provisioner for EKS with CSI Driver
- [Optimized worker node management with Ocean from Spot by NetApp](https://www.eksworkshop.com/beginner/190_ocean/)
  - Create a Free Spot by NetApp Account
  - Connect Ocean to EKS cluster
  - Headroom, Showback, Rightsizing
  - Cluster logs and scaling decisions
- [Encrypt Secrets with AWS Key Management Service (KMS) Keys](https://www.eksworkshop.com/beginner/191_secrets/)
  - AWS KMS and Custom Key store
  - Access the Secret from a pod
- [Mount secrets from AWS Secrets Manager](https://www.eksworkshop.com/beginner/194_secrets_manager/)
  - Secrets Store CSI Driver and ASCP
  - Prepare secret and IAM access control
  - Deploy pods with mounted secrets
  - Sync with native k8s secrets
- [Secure Secrets using Sealed Secrets](https://www.eksworkshop.com/beginner/200_secrets/)
  - Sealed Secrets for k8s
  - Scale Secrets
  - Manage the Sealing Key
- [Windows containers on EKS](https://www.eksworkshop.com/beginner/300_windows/)
  - Windows nodes
  - Create Network Policies using Calico

## [Intermediate](https://www.eksworkshop.com/intermediate/)

- [Migrate to EKS](https://www.eksworkshop.com/intermediate/200_migrate_to_eks/)
  - Create, configure EKS cluster using `kind`
  - Deploy app, expose app, database
- [Resource management](https://www.eksworkshop.com/intermediate/201_resource_management/)
  - Deploy metrics server
  - Basic and advanced pod CPU and memory management
  - Resource quotas
  - Pod priority and preemption
- [Deploy Jenkins](https://www.eksworkshop.com/intermediate/210_jenkins/)
- [CI/CD with CodePipeline](https://www.eksworkshop.com/intermediate/220_codepipeline/)
- [Logging with Amazon OpenSearch, Fluent Bit](https://www.eksworkshop.com/intermediate/230_logging/)
  - Configure IRSA for Fluent Bit, deploy Fluent Bit
  - Provision, configure an Amazon OpenSearch cluster
- [Monitoring using Prometheus and Grafana](https://www.eksworkshop.com/intermediate/240_monitoring/)
- [Monitoring using Pixie](https://www.eksworkshop.com/intermediate/241_pixie/)
- [Tracing with X-Ray](https://www.eksworkshop.com/intermediate/245_x-ray/)
- [Monitoring using Amazon Managed Service for Prometheus/Grafana (AMP)](https://www.eksworkshop.com/intermediate/246_monitoring_amp_amg/)
- [EKS CloudWatch Container Insights](https://www.eksworkshop.com/intermediate/250_cloudwatch_container_insights/)
  - Install Container Insights
  - Run Load test
  - Use CloudWatch Alarms
- [GitOps with Weave Flux](https://www.eksworkshop.com/intermediate/260_weave_flux/)
- [Continuous deployment with Argo CD](https://www.eksworkshop.com/intermediate/290_argocd/)
- [Continuous delivery with Spinnaker](https://www.eksworkshop.com/intermediate/265_spinnaker_eks/)
- [Custom Resource Definition](https://www.eksworkshop.com/intermediate/270_custom_resource_definition/)
- [CIS EKS Benchmark assessment using kube-bench](https://www.eksworkshop.com/intermediate/300_cis_eks_benchmark/)
- [Use Open Policy Agent (OPA) for policy-based control in EKS](https://www.eksworkshop.com/intermediate/310_opa_gatekeeper/)
- [Patch/Upgrade EKS cluster](https://www.eksworkshop.com/intermediate/320_eks_upgrades/)
  - The Upgrade process
  - Upgrade EKS control plane, core add-ons, managed node group
- [AWS App Mesh](https://www.eksworkshop.com/intermediate/330_app_mesh/)

## [Advanced](https://www.eksworkshop.com/advanced/)

- [Service Mesh with Istio](https://www.eksworkshop.com/advanced/310_servicemesh_with_istio/)
- [Service Mesh using AWS App Mesh](https://www.eksworkshop.com/advanced/330_servicemesh_using_appmesh/)
- [Canary deployment using Flagger in AWS App Mesh](https://www.eksworkshop.com/advanced/340_appmesh_flagger/)
- [Batch processing with Argo workflow](https://www.eksworkshop.com/advanced/410_batch/)
  - k8s jobs
  - Simple, advanced batch workflow
- [Machine Learning using Kubeflow](https://www.eksworkshop.com/advanced/420_kubeflow/)
  - Jupyter notebook, training, inference
  - Publish inference endpoint using Kubeflow Fairing
  - Kubeflow pipeline, distributed training
- [EMR on EKS](https://www.eksworkshop.com/advanced/430_emr_on_eks/)
  - Monitoring and logging setup, CloudWatch, S3, Spark history server, Prometheus, Grafana
  - Configure autoscaling
  - Use Spot instance
  - Serveless EMR job setup, monitor and troubleshoot
