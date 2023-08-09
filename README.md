# rhods-gitops

## How to use

* Install the Red Hat GitOps Operator (ArgoCD)
* Configure the parent app

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: rhods-lab
  namespace: openshift-gitops
spec:
  destination:
    namespace: openshift-gitops
    server: 'https://kubernetes.default.svc'
  project: default
  source:
    helm:
      values: 'base_domain: apps.your.ingress.controller.url'
    path: apps
    repoURL: 'https://github.com/nesanton/rhods-gitops'
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - ApplyOutOfSyncOnly=true
```
* 