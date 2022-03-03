# Ingress using kubeadm

[Ingress document](https://kubernetes.io/docs/concepts/services-networking/ingress/)

## Install

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

## Customize ingress

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

## Expose ingress controller in local k8s

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

## Useful tools

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
