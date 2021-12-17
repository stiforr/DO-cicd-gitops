# DO-cicd-gitops

Digital Ocean GitOps CI/CD implementation for Kubernetes
Challenge https://www.digitalocean.com/community/pages/kubernetes-challenge

## Getting Started

Table of Contents

- [Fork and Clone the repository](#Fork and Clone the repository)
- [Setup GPG and SOPS](#Setup GPG and SOPS)
- [Bootstrap the cluster and wait for resources to be ready](#Bootstrap the cluster and wait for resources to be ready)

# Tools used

- helm
- flux
- argocd
- tekton
- sops

# Fork and Clone the repository

- `git clone https://github.com/<GITHUB_USERNAME>/DO-cicd-gitops`

# Setup GPG and SOPS

> Note: Detailed guide here https://fluxcd.io/docs/guides/mozilla-sops/

- Create `flux-system` namespace

`kubectl create ns flux-system`

- Install gnupg and sops

`brew install gnupg sops`

- Generate a key and get the fingerprint

```bash
export KEY_NAME="cluster0.yourdomain.com"
export KEY_COMMENT="flux secrets"

gpg --batch --full-generate-key <<EOF
%no-protection
Key-Type: 1
Key-Length: 4096
Subkey-Type: 1
Subkey-Length: 4096
Expire-Date: 0
Name-Comment: ${KEY_COMMENT}
Name-Real: ${KEY_NAME}
EOF 
```

```bash
gpg --list-secret-keys "${KEY_NAME}"

export KEY_FP=1F3D1CED2F865F5E59CA564553241F147E7C5FA4
```

- Export the keypair into a secret

```bash
gpg --export-secret-keys --armor "${KEY_FP}" |
kubectl create secret generic sops-gpg \
--namespace=flux-system \
--from-file=sops.asc=/dev/stdin
```

- Update .sops.yaml

```yaml
creation_rules:
  - path_regex: .*.yaml
    encrypted_regex: ^(data|stringData)$
    pgp: <KEY_FP>
```

- Update secrets in `./cluster/secrets`
    - `auth.yaml` is a basic auth secret generated with `htpasswd`
    - `cloudflare-secrets.yaml` is a cloudflare email and api-token secret

# Bootstrap the cluster and wait for resources to be ready

```bash
flux bootstrap github --owner <GITHUB_USERNAME> --repository DO-cicd-gitops --branch main --path cluster/base --components source-controller,kustomize-controller
```

```bash
kubectl get kustomizations -n flux-system -w
```