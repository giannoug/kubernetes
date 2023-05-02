[//]: # "renovate: datasource=github-releases depName=k3s-io/k3s"
[![k3s](https://img.shields.io/badge/k8s-v1.27.1+k3s1-orange?style=for-the-badge&logo=kubernetes)](https://k3s.io/)
[![flux](https://img.shields.io/badge/GitOps-Flux-blue?style=for-the-badge&logo=git)](https://fluxcd.io/)
[![renovate](https://img.shields.io/badge/renovate-enabled-brightgreen?style=for-the-badge&logo=renovatebot)](https://github.com/renovatebot/renovate)

# ☸ k8s clusters backed by Flux v2

Kubernetes clusters using the [GitOps](https://www.weave.works/blog/what-is-gitops-really) tool [Flux](https://fluxcd.io/).  
The Git repository is driving the state of the Kubernetes clusters.
The awesome [Flux SOPS integration](https://toolkit.fluxcd.io/guides/mozilla-sops/) is used to encrypt secrets with age.

## 📂 Repository structure

The Git repository contains the following directories:

```sh
📁
├─📁 apps
│  ├─📁 all          # apps available for intallation
│  └─📁 ...          # kustomization and overlays for app installations per cluster
├─📁 base
│  ├─📁 flux-system  # flux & gitops operator
│  └─📁 ...          # flux configuration per cluster
├─📁 charts          # helm chart repos
├─📁 config          # configs per cluster
└─📁 crds            # custom resources required by apps
```

## 🤖 Automation

[Renovate](https://www.whitesourcesoftware.com/free-developer-tools/renovate) Bot makes sure the components are never outdated.

It creates PullRequests when Helm charts or Docker images have newer versions available and even keeps Flux and k3s up-to-date.

## 🤝 Thanks

Big shout out to [Pumba98](https://github.com/Pumba98), [k8s@home](https://github.com/k8s-at-home) and everyone from [awesome-home-kubernetes](https://github.com/k8s-at-home/awesome-home-kubernetes) for the inspiration :heart:

## 📖 Notes

<details>
    <summary>📍 Bootstrap Notes</summary>
<br>

Install your favorite OS, and install K3s without traefik (we do that ourselves).

```
# curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=traefik" sh -
```

Create your personal age key and overwrite the Kubernetes secret with it.

```
# age-keygen -o age.agekey

# cat ~/age.agekey |
kubectl create secret generic sops-age \
--namespace=flux-system \
--from-file=age.agekey=/dev/stdin \
--dry-run=client \
-o yaml > base/flux-system/init/flux-sops-age-secret.sops.yaml

# export SOPS_AGE_RECIPIENTS=age1hlfnnwk9z9jynzngesd0j35n6rmpry70z9zak6ullmvesvvjge2sjc9nsf

# sops --encrypt --encrypted-regex '^(data|stringData)$' --in-place base/flux-system/init/flux-sops-age-secret.sops.yaml

# flux install --export > base/flux-system/gotk-components.yaml
```

</details>

<details>
    <summary>📍 Installation Notes</summary>
<br>

**tl;dr**
```
# kubectl create namespace flux-system --dry-run=client -o yaml | kubectl apply -f -
# sops -d ./base/flux-system/init/flux-sops-age-secret.sops.yaml | kubectl apply -f -
# kubectl apply --kustomize=./base/flux-system
# kubectl apply --kustomize=./base/cultured-crocodile
```

1. Pre-create the `flux-system` namespace

```
# kubectl create namespace flux-system --dry-run=client -o yaml | kubectl apply -f -
```

4. Add the Flux age key in-order for Flux to decrypt SOPS secrets

```
# sops -d ./base/flux-system/init/flux-sops-age-secret.sops.yaml | kubectl apply -f -
```

5. Install Flux

```
# kubectl apply --kustomize=./base/flux-system
```

6. Configure Flux

```
# kubectl apply --kustomize=./base/cultured-crocodile
```

</details>
