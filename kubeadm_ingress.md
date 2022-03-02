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
    - name: green
      image: docker.io/mtinside/blue-green:green
---
apiVersion: v1
kind: Service
metadata:
  name: green
spec:
  ports:
    - port: 80
      targetPort: 8080
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
                name: green
                port:
                  number: 80
```

Test:

```bash
# Create resources
kubectl apply -f .
# Forward port 80 from ingress ngxin controller to localhost:8080
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
# Test
curl http://demo.localdev.me:8080/green
# Clean resources
kubectl delete -f .
```
