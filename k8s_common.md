# Kubernetes common yaml

## General

- [Configuration Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
- [Ports and Protocols](https://kubernetes.io/docs/reference/ports-and-protocols/)
- [Recommended Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)
- [guestbook-all-in-one.yaml](https://github.com/kubernetes/examples/blob/master/guestbook/all-in-one/guestbook-all-in-one.yaml)

## Pod

- **Sidecar** concept: A sidecar is just a container that runs on the same Pod as the application container. It shares the same volume and network as the main container, helps, and enhances how the application operates by directly intercepting all traffic. E.g: log shippers, log watchers, monitoring agents among others.

- Each Pod has a single IP address assigned from the Pod CIDR range of its node. This IP address is shared by all containers running within the Pod. To communicate between containers in the same pod, check [this](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/).

## Service

## Deployment

[Deployment document](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

```yaml
# Ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
# livenessProbe: to check health status
# This is used as the signal to restart the container
spec:
  containers:
    - name: envbin
      image: mtinside/envbin:latest
      livenessProbe:
        httpGet:
          path: /health
          port: 8080
        initialDelaySeconds: 1
        periodSeconds: 1
---
# readinessProbe: to check ready status
# This is not used to restart the container, but to not assign IP to the pod
# Use case 1: when the container is not dead, but it's not ready due to some reasons
# e.g: lose connection with database/servers, something inside is broken temporarily
# This error will persist forever without restarting the container until we do something
# Use case 2: when the container takes time to initialize some tasks
# Note: when rolling out a new version of an image, k8s waits for readinessProbe
# to declare one Pod upgraded and move onto the next Pod.
spec:
  containers:
    - name: envbin
      image: mtinside/envbin:latest
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 0
        periodSeconds: 1
        timeoutSeconds: 1
        successThreshold: 1
        failureThreshold: 1
```

```yaml
# Ref: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
# resources requests and limits
# cpu 100m means 100 millicores = 1/10 cores

# For requests
# It's like treating requests as sizes of pods
# k8s finds the node that fits different pods to ensure maximum usage of all nodes
# This operation is called bin packing.
# By default, requests are set to 0. This means that the pod can be assigned to any node.
# That node might not find enough resources on the assigned node. By this reason, you should
# always set the requests. You can run load test, or guess and adjust repeatedly,
# or use k8s's feature called Vertical Pod Auto-scaler which watchs and adjusts for you.

# For limits
# For the CPU to use 1/10 cores, the thread will be forced to run at maximum 10% of time
# For the memory, if the container cannot allocate memory, then it's just crashed
# Setting CPU limit doesn't make sense because the CPU can just use cycles that nobody use.
# If 2 pods want to use more CPU, then one of them will be killed and throttled back
# to the request amount. So we don't need to set the CPU limit.
# Setting memory limit is very important. If 2 pods use more than total memory,
# the OOM (out of memory) killer will kill something randomly,
# and might cause the node to be crashed. Hence, we should set the limit
# to be the same as the request to make sure this never happens.

# If no node can fulfill the pod's requests/limits, the pod will never be assigned to a node
spec:
  containers:
    - name: web
      image: nginx:1.18.0
      resources:
        requests:
          memory: 64Mi
          cpu: 100m
        limits:
          memory: 1Gi
          cpu: 1
```

```yaml
# Ref: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity
# Ref: https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/
# Pod affinity: to put a pod close to others (same node, zone, etc)
# Below example shows that the pod wants to be close to pods that have label app=cache
# topologyKey defines the location where the pods are defined as "close"
# topologyKey=kubernetes.io/hostname means the pod wants to be in the same node with selected pods
# Use case: to put an app pod in the same node with a cache pod
# to increase the bandwidth and reduce the latency
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchLabels:
              app: cache
          topologyKey: kubernetes.io/hostname
---
# Pod anti-affinity: to spread the pod far from others (based on node, zone, etc)
# Below example shows that the pod wants to be far to pods that have label app=web
# topologyKey defines the location where the pods are defined as "far"
# topologyKey: topology.kubernetes.io/zone means the pod wants to be in different zone
# with selected pods.
# maxSkew: 1 means the difference between zone (in this case) should be <= 1
# Use case: spreading pods in different zones to increase the high availability
# This is we almost always want to use
spec:
  topologySpreadConstraints:
    - labelSelector:
        matchLabels:
          app: web
      topologyKey: topology.kubernetes.io/zone
      maxSkew: 1
      whenUnsatisfiable: ScheduleAnyway
---
# Node affinity: to put a pod to specific nodes
# Below example shows that the pod wants to be in the node with name in ["big-gpu", "expensive-gpu"]
# Use case: to put a pod with special jobs in special nodes that have special hardware or sth
# Example: to put an ML pod in nodes that have GPU
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values: ["big-gpu", "expensive-gpu"]
---
# Ref: https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
# Node anti-affinity: to tell pods to stay away from specific nodes for some reasons
# Doing this by "taint"-ing a node: kubectl taint nodes node1 key1=value1:NoSchedule
# This will tell pods to stay away from 'node1'
# Use case: we want to reserve special nodes for special pods by using 'tolerations'
# Below example shows that the pod wants to be in a GPU node (affinity field),
# and it insists to be in a GPU node even when the GPU node is tainted (tolerations field)
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values: ["big-gpu", "expensive-gpu"]
  tolerations:
    - key: "hardware"
      value: "gpu"
```

```yaml
# Ref: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
# Horizontal Pod Auto-scaler: to auto-scale the pod when exceeding average utilization
# The averageUtilization is the average utilization percentage of ALL deployments
# that have the same name (specified by scaleTargetRef) based on what it requests for.
# Eg: It can be 1 pod runs at 100%, 1 pod runs at 0% -> no more scale because average = 50%
# In reality, the user requests will be sent equally to different pods
# to avoid the above imbalanced workload case.
# The default time to check for scaling down to minReplicas is 5 minutes
# to stablize the fluctuating metric values.
# We can use Prometheus to add custom metrics. Eg: response time, etc
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: envbin
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: envbin
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 80
```

## ConfigMap

```yaml
# Ref: https://kubernetes.io/docs/concepts/configuration/configmap/
# Example 1: Load key's value of ConfigMap
# colour-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: colour-config
data:
  colour: pink
---
# pink.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pink
spec:
  containers:
    - name: pink
      image: mtinside/blue-green:blue
      env:
        - name: COLOUR
          valueFrom:
            configMapKeyRef:
              name: colour-config
              key: colour
---
# Example 2: Load files from ConfigMap
# website-configmap.yaml, created by running
# kubectl create configmap website --from-file=index.html --dry-run -o yaml > website-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: website
data:
  index.html:
    "<html>\n    <head></head>\n    <body>\n        <p>Hello from Matt!
    \U0001F44B</p>\n    </body>\n</html>\n"
---
# web.yaml
# Use spec.volumes.configMap to mount files
# Each key is one file
# Key is filename, value is its content
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
    - name: web
      image: nginx:1.18.0
      volumeMounts:
        - name: website-volume
          mountPath: /usr/share/nginx/html
  volumes:
    - name: website-volume
      configMap:
        name: website
```

## Secret

- [Secret document](https://kubernetes.io/docs/concepts/configuration/secret/)
- Common secret type 1: Generic - use it same as ConfigMap
- Common secret type 2: TLS - provded for user's convenience, to ensure the consistency of Secret format by enforcing extra semantic validation over the values.

## Network Policy

```yaml
# Ref: https://kubernetes.io/docs/concepts/services-networking/network-policies/
# NetworkPolicy is not enabled in minikube, even in some cloud env
# When you define at least 1 NetworkPolicy, you have triggered the whole cluster
# to deny the communication between pods by default.
# Example
# deny-all.yaml: This denies all the traffic between pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
---
# allow-blue.yaml: This allows traffic from pod with label app=shell
# to pod with label colour=blue on port 8080 with TCP protocol
# policyTypes=ingress/egress: defines allowed direction of traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-blue
spec:
  podSelector:
    matchLabels:
      colour: blue
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: shell
      ports:
        - protocol: TCP
          port: 8080
```

## Controlling access

- [Controlling access document](https://kubernetes.io/docs/concepts/security/controlling-access/)
- Authentication:
  - Input: entire HTTP request
  - Job: examines the headers and/or client certificate, password, plain tokens, boostrap tokens, and JSON Web Tokens (used for service accounts). Multiple authentication modules can be specified and are tried in sequence until one of them succeeds.
- Authorization:
  - Input: request with username, requested action, object affected by the action
  - Job: examines if an existing policy declares that the user has permissions to complete the requested action. If more than one authorization modules are configured, k8s checks each module. If any module authorizes the request, then the request can proceed.
  - Example module: ABAC, RBAC, Webhook.
- Admission control:
  - Input: requests
  - Job: modify and reject requests, access contents of the created/modified objects. Unlike 2 above modules, if any admission controller module rejects, then the request is immediately rejected.

## Role-based access control (RBAC)

- [RBAC Authorization document](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization.

```yaml
# service-acc.yaml
# Use kind 'ServiceAccount' to create a service account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: envbin
---
# envbin.yaml
apiVersion: v1
kind: Pod
metadata:
  name: envbin-sa
spec:
  # If not specified, serviceAccount will be 'default' with no permission
  serviceAccount: envbin
  containers:
    - name: envbin-sa
      image: mtinside/envbin:latest
---
# rbac.yaml
# Use kind 'Role' to create a set of permissions for Pods
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
---
# Use kind 'ClusterRole' to create a set of permissions for Nodes
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["nodes"]
    verbs: ["get", "watch", "list"]
---
# Use kind 'RoleBinding' to bind a Role with a service account
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: envbin-read-pods
  namespace: default
subjects:
  - kind: ServiceAccount
    name: envbin
    namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
---
# Use kind 'ClusterRoleBinding' to bind a ClusterRole with a service account
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: envbin-read-nodes
  namespace: default
subjects:
  - kind: ServiceAccount
    name: envbin
    namespace: default
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

## Service mesh

- [Kubernetes Service Mesh: A Comparison of Istio, Linkerd, and Consul](https://platform9.com/blog/kubernetes-service-mesh-a-comparison-of-istio-linkerd-and-consul/)
- [Linkerd Service mesh](https://linkerd.io/what-is-a-service-mesh/)

```bash
# Run this to check how linkerd inject YAML
cat <yaml-file> | linkerd inject --manual -

# Instead, run this to enable linkerd injection every time we deploy a pod in namespace 'default'
# linkerd intercepts the processing of creating the pod and inject some YAML into the pod's yaml
# before passing to k8s
kubectl annotate namespace default linkerd.io/inject=enabled
```

## Custom resource definition (CRD)

- [CRD document](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
- [Extend k8s using CRD document](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)
- [Operator pattern document](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)
  - Operators are software extensions to k8s that make use of CRDs to manage apps and their components. With Operators you can stop treating an application as a collection of primitives like Pods, Deployments, Services or ConfigMaps, but instead as a **single** object that only exposes the knobs that make sense for the application.
  - [Operator Hub](https://operatorhub.io/)
