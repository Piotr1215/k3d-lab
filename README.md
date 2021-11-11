# k3d-lab

Quick k3s lab cluster for MacOS

## Install k3d

```bash
k3d cluster create k3d-rancher \
    --api-port 6550 \
    --servers 1 \
    --image rancher/k3s:v1.20.10-k3s1 \
    --port 443:443@loadbalancer \
    --wait --verbose
```

### Swap to right context

```bash
k3d kubeconfig get k3d-rancher > $HOME/.kube/k3d-config

echo export KUBECONFIG=~/.kube/config:~/.kube/k3d-config >> ~/.zshrc

source ~/.zshrc
```

## Install Rancher and Cert Manager

### Install cert-manager with helm
helm repo add jetstack https://charts.jetstack.io
helm repo update
kubectl create namespace cert-manager
helm install cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --version v1.5.3 \
    --set installCRDs=true --wait --debug
kubectl -n cert-manager rollout status deploy/cert-manager

### Install the helm repos for rancher
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
kubectl create namespace cattle-system
helm install rancher rancher-latest/rancher \
    --namespace cattle-system \
    --version=2.6.0 \
    --set hostname=rancher.localhost \
    --set bootstrapPassword=passwd123 \
    --wait --debug
kubectl -n cattle-system rollout status deploy/rancher
kubectl -n cattle-system get all,ing
