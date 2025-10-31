# Start Kind cluster
```bash
kind create cluster --name argo-proj
```

# Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

To access the Argo CD UI, forward the service port:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open [http://localhost:8080](http://localhost:8080) in your browser.

Get the initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## Install ArgoCD cli
```bash
brew install argocd
argocd login localhost:8080
``` 


---

# Install Argo Workflows

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm upgrade -i argo-workflows argo/argo-workflows -n argo --create-namespace -f argo-workflows/values.yaml
```


## Install argo cli
```bash
ARGO_OS="darwin"; [ "$(uname -s)" != "Darwin" ] && ARGO_OS="linux"
curl -sLO "https://github.com/argoproj/argo-workflows/releases/download/v3.7.0/argo-$ARGO_OS-amd64.gz"
gunzip "argo-$ARGO_OS-amd64.gz"
chmod +x "argo-$ARGO_OS-amd64"
sudo mv "./argo-$ARGO_OS-amd64" /usr/local/bin/argo

# test
argo version
```