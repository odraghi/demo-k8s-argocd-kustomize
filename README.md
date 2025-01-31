# Gitops sample

# Information

This repo is a demo for kubernetes:
- argocd
- kustomize, base yaml files for 3 overlay env (dev, pp, prod)
- let's encrypt certificate (issuer and cluster-issuer usage)
- nginx template to get env variable and k8s secret

Directories:
- application : standard yaml file, certif SSL via issuer let's encrypt in the namespace
- kustomapp : kustomize application with 3 env (dev, pp, prod), certif SSL via cluster-issuer let's encrypt
- argocd : ingress to expose argocd AND applications yaml files

# Application
Topology
ingress -> service -> pods (nginx return json sample that read a kubernetes secret)

The app use DNS name in nip.ip domain that resovle my public IPs.
That simplify let's encrypt certificat generations as DNS is already set (ACME mode http01).
- myapp.95-181-220-26.nip.io -> 95.181.220.26
- myapp.dev.95-181-220-26.nip.io -> 95.181.220.26
- myapp.prod.95-181-220-26.nip.io -> 95.181.220.26


## install ArgoCD in k8s
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## access ArgoCD UI
```bash
kubectl get svc -n argocd
kubectl port-forward svc/argocd-server 8080:443 -n argocd
```
To expose argoCD UI on https://argocd.95-181-220-26.nip.io
```bash
kubectl apply -n argocd -f argocd/ingress.yaml
```

login with admin user and below token (as in documentation):
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
```

you can change and delete init password

## Pre-requisites
- gitlab / project and argocd user member as a repoter
- install cert-manager in k8s cluster
 
Setup CusterIssuer for let's encrypt
```bash
kubectl apply -f cluster-issuer.yaml
```

## applications in argoCD
```bash
kubectl apply -f argocd/application.yaml

curl -vI https://myapp.95-181-220-26.nip.io
curl -s https://myapp.95-181-220-26.nip.io | jq

kubectl delete -f argocd/application.yaml
```

## kustomapp in argoCD
```bash
## Env Devel
kubectl apply -f argocd/kustomapp-dev.yaml
curl -vI https://kustomapp.dev.95-181-220-26.nip.io
curl -s https://kustomapp.dev.95-181-220-26.nip.io | jq
kubectl delete -f argocd/kustomapp-dev.yaml

## Env Pre-prod
kubectl apply -f argocd/kustomapp-pp.yaml
curl -vI https://kustomapp.pp.95-181-220-26.nip.io
curl -s https://kustomapp.pp.95-181-220-26.nip.io | jq
kubectl delete -f argocd/kustomapp-pp.yaml

## Env Production
kubectl apply -f argocd/kustomapp-pp.yaml
curl -vI https://kustomapp.pp.95-181-220-26.nip.io
curl -s https://kustomapp.pp.95-181-220-26.nip.io | jq
kubectl delete -f argocd/kustomapp-pp.yaml
```

## kustomapp (kustomize demo) - sample without argoCD
```bash
## Env Prod
kubectl create namespace prod
kubectl kustomize kustomapp/overlays/prod/
kubectl apply -k kustomapp/overlays/prod/
kubectl -n prod get all

curl -vI https://kustomapp.prod.95-181-220-26.nip.io
curl -s https://kustomapp.prod.95-181-220-26.nip.io | jq
kubectl delete -k kustomapp/overlays/prod/
```