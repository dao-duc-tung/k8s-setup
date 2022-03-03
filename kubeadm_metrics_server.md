# Metrics server using kubeadm

If you just install [metrics server](https://github.com/kubernetes-sigs/metrics-server) by running this command

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

you will most likely get this error

```bash
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get pods.metrics.k8s.io)
```

The reason is because kubelet certificate needs to be signed by cluster Certificate Authority (or disable certificate validation by passing `--kubelet-insecure-tls` to Metrics Server). This is stated in the [requirements of the metrics server](https://github.com/kubernetes-sigs/metrics-server#requirements).

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
