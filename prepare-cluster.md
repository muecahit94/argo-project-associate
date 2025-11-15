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

To access the Argo Workflows UI, forward the server port:
```bash
kubectl -n argo port-forward deployment/argo-workflows-server 2746:2746
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


# Install Argo Rollouts

```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/download/v1.6.4/install.yaml
```

## Install Argo Rollouts CLI

```bash
brew install argoproj/tap/kubectl-argo-rollouts
kubectl argo rollouts version

# Optional, bash completion
source <(kubectl-argo-rollouts completion zsh)
```

## Access Argo Rollouts Dashboard

```bash
kubectl argo rollouts dashboard
```

Then open [http://localhost:3100](http://localhost:3100) in your browser.

## Example commands

```bash
kubectl get rollout
kubectl argo rollouts get rollout <rollout-name>
kubectl argo rollouts promote <rollout-name>
kubectl argo rollouts undo <rollout-name>
```


# Installing Argo Events and the Needed Components

```bash
kubectl create namespace argo-events
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install.yaml
```

The next command applies a validating webhook for Argo Events. Validating webhooks are
used to ensure that incoming requests to the Kubernetes API server are valid:

```bash
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install-validating-webhook.yaml
```

For setting up a native EventBus in the 'argo-events' namespace, which handles event
transportation in Argo Events, apply the configuration with this command:

```bash
kubectl -n argo-events apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/eventbus/native.yaml
```

Next we need to define an EventSource configuration that listens for webhook events in Argo
Events, apply the following configuration using this command:

```bash
kubectl -n argo-events apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/event-sources/webhook.yaml
```

For the Sensor to properly interact with Kubernetes resources, apply the necessary RBAC
policies:

```bash
kubectl apply -n argo-events -f https://raw.githubusercontent.com/argoproj/argo-events/master/examples/rbac/sensor-rbac.yaml
```

Similarly, apply RBAC policies for Workflows to ensure they have the necessary permissions in
Kubernetes:

```bash
kubectl apply -n argo-events -f https://raw.githubusercontent.com/argoproj/argo-events/master/examples/rbac/workflow-rbac.yaml
```

Set up a Sensor to trigger workflows based on webhook events by applying this Sensor
configuration:

```bash
kubectl -n argo-events apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/sensors/webhook.yaml
```