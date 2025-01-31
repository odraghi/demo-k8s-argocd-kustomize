# Gitops sample

# Information

Directories:
- application : standard yaml file
- kustomapp : kustomize application with 2 env (dev & prod)
- argocd : ingress to expose argocd AND applications yaml files

Argocd file to create apps:
- application.yaml
- kustomapp-dev.yaml
- kustomapp-prod.yaml

# Application
Topology
ingress -> service -> pods (nginx return json sample that read a kubernetes secret)

The app use DNS name in nip.ip domain that resovle my public IPs.
That simplify let's encrypt certificat generations as DNS is already set (ACME mode http01).
- myapp.95-181-220-26.nip.io -> 95.181.220.26
- myapp.dev.95-181-220-26.nip.io -> 95.181.220.26
- myapp.prod.95-181-220-26.nip.io -> 95.181.220.26


## Pre-requisites
- gitlab / project and argocd user member as a repoter
- install cert-manager in k8s cluster
 
Setup CusterIssuer for let's encrypt
```bash
kubectl apply -f cluster-issuer.yaml
```

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


## login with admin user and below token (as in documentation):
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
```

you can change and delete init password

