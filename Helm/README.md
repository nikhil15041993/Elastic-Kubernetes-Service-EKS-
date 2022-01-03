# Helm

In simple terms, Helm is a package manager for Kubernetes. Helm is the K8s equivalent of yum or apt. Helm deploys charts, which you can think of as a packaged application
Helm is a tool for managing Charts. Charts are packages of pre-configured Kubernetes resources.

```bash
cd 
mkdir helm && cd helm
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
cd -
```

## Add official stable Helm repository

```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```
creted a repositary with name stable.

to check the newly created repositary
``` yem repo list ```
