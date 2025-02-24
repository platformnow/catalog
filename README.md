# PlatformNOW Catalog

## Install Crossplane

```bash
helm repo add \
crossplane-stable https://charts.crossplane.io/stable
helm repo update
```

```bash
helm install \
crossplane crossplane-stable/crossplane \
--namespace landscape-system \
--create-namespace
```

## Install Providers

```bash
kubectl apply -f ./providers/kubernetes
kubectl apply -f ./providers/helm
kubectl apply -f ./providers/argocd
kubectl apply -f ./providers/github
```

## Install Provider Config for Github

Required so that we can talk to Github from Crossplane. Example uses GitHub Actions to set the GH_TOKEN and GH_OWNER environment variables.
we can also use Github App intead see [https://github.com/crossplane-contrib/provider-upjet-github/blob/main/README.md](here) for more details.

```bash
export GH_TOKEN=...
export GH_OWNER=...
```

```bash
cat <<EOF | kubectl apply -f -
---
apiVersion: v1
kind: Secret
metadata:
  name: provider-secret
  namespace: landscape-system
type: Opaque
stringData:
  credentials: "{\"token\":\"${GH_TOKEN}\",\"owner\":\"${GH_OWNER}\"}"

---
apiVersion: github.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: provider-github
spec:
  credentials:
    source: Secret
    secretRef:
      name: provider-secret
      namespace: landscape-system
      key: credentials
EOF
```

## Create the Catalog Github Repository

Create the catalog repository in Github.

```bash
cat <<EOF | kubectl apply -f -
---
apiVersion: repo.github.upbound.io/v1alpha1
kind: Repository
metadata:
  name: platformnow-catalog
spec:
  forProvider:
    visibility: public
  providerConfigRef:
    name: provider-github
EOF
```
