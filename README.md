
## Create the cluster

```bash
kind create cluster --config cluster/kind-config.yaml 
```
## Install ArgoCD with Helm

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argo-cd argo/argo-cd --namespace argocd --create-namespace
```

 ## Forward ArgoCD UI
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
## Login to ArgoCD

login to ArgoCD UI at `https://localhost:8080` with the default username `admin` and password from the secret:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
``` 
 ### Deploy cluster services
```bash
kubectl apply -f cluster/metrics-server-argocd.yaml
kubectl apply -f cluster/nginx-ingress.yaml
kubectl apply -f cluster/kube-prometheus-stack.yaml
kubectl apply -f cluster/k6.yaml
```

## Forward Grafana UI
```bash
kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 3000:80
```

## Login to grafana
login to Grafana UI at `http://localhost:3000` with the default username `admin` and password from the secret:

```bash
kubectl get secret kube-prometheus-stack-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 -d
```

## Deploy the demo application

```bash
kubectl apply -f apps/webapp/application.yaml
```

## Forward podinfo UI
```bash
kubectl port-forward -n ingress-nginx svc/nginx-ingress-ingress-nginx-controller 8000:80
```

## Add podinfo host to /etc/hosts
```bash
echo "127.0.0.1 webapp.local" | sudo tee -a /etc/hosts
```

## Access podinfo UI
Open your browser and go to `http://podinfo.local:8000/` to access the podinfo application.


## Apply the load test

```bash
kubectl apply -f k6/test.yaml
```

## Visualize the results

```bash
kubectl get hpa -A -w
```