# cluster-argo-setup

```bash

kind create cluster --config=config.yaml

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# download argo CLI from https://github.com/argoproj/argo-cd/releases/latest
# or brew install argocd

ARGOPASS=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)

# with UI

kubectl port-forward svc/argocd-server -n argocd 8080:443

# with CLI

argocd login <ARGOCD_SERVER>
argocd login localhost:8080 --username admin --password $ARGOPASS --insecure
argocd account update-password
```

## Ingress

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Create ingress: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/

## terminate TLS ingress traffic at the edge

The Argo CD API server should be run with TLS disabled. Edit the argocd-server Deployment to add the --insecure flag to the argocd-server container command.

```yaml
spec:
  template:
    spec:
      containers:
      - name: argocd-server
        command:
        - argocd-server
        - --repo-server
        - argocd-repo-server:8081
        - --insecure
```

## Repo

```bash
argocd repo add git@github.com:argoproj/argocd-example-apps.git --ssh-private-key-path ~/.ssh/id_rsa

argocd repo add https://github.com/MRostanski/cluster-argo-setup.git

# argocd app create app-manual --repo https://github.com/MRostanski/cluster-argo-setup.git --path dev --dest-namespace default

kubectl apply -f application.yaml

```