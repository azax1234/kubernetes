### Installing NGINX ingress controller

install an Ingress Controller
An Ingress Controller is required to manage Ingress resources. The NGINX Ingress Controller is a popular choice:
Create a namespace for the Ingress Controller:
``` bash
kubectl create namespace ingress-basic
``` 
Add the NGINX Ingress Controller using Helm:
``` bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-basic
``` 

### Create an Ingress resourece. Make sure to add ingress classs
***below is a basic path based ingress resource***
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx # Replace 'nginx' with the correct IngressClass name if different
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: explore-california
            port:
              number: 80
```
Apply the Ingress resource:
```sh
kubectl apply -f my-app-ingress.yaml
```
### Ingress troubleshooting

```sh
kubectl get pods -n ingress-basic
kubectl logs -n ingress-basic <ingress-controller-pod-name>
kubectl get ingress
kubectl get ingressclasses