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
  --rule=demo.localdev.me/*=demo:80
# Forward port 80 from ingress ngxin controller to localhost:8080
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
# Test
curl http://demo.localdev.me:8080/
# Clean resources
kubectl delete ing demo-localhost
kubectl delete svc demo
kubectl delete deployment demo
```

## Customize

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
  rules:
    - host: "demo.localdev.me"
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
# Forward incoming requests at localhost:8080 to ingress ngxin controller:80
# nginx ingress controller listens on port 80
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
# Test
curl http://demo.localdev.me:8080/green
# Clean resources
kubectl delete -f .
```

## Useful commands

```bash
# Assume you are working on machine A with IP 10.121.177.58/28
# and you're running k8s cluster in machine B with IP 10.121.177.31/28
# You want to forward incoming requests from 10.121.177.58:8888 to 10.121.177.31:8080 through SSH
# In machine A, run:
ssh -N -L 8888:127.0.0.1:8080 <username>@10.121.177.31
# By doing this, now you can access http://demo.localdev.me:8080/green from machine A
# Ref: https://www.tecmint.com/create-ssh-tunneling-port-forwarding-in-linux/
# -L flag: defines the port forwarded to the remote host and remote port
# -N flag: do not execute a remote command, you won't get a shell in this case
```
