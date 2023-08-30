# rhods-gitops
An example of GitOps driven Lab environment deployment with Red Hat Openshift Data Science.

This repo demonstrates how easy it can be to roll out any number of Jupyter notebooks (JupyterLab servers) in minutes, painlessly add and remove students as well as remove the whole Lab.

The setup relies on three components:

* Red Hat Openshift. A managed flavour such as ROSA or ARO could speed things up even more, so the whole lab is ready from scratch in under one hour.
* Red Hat Openshift GitOps. The productized version of ArgoCD. It's ready to use in minutes and requires little to no additional configuration.
* Red Hat Openshift Data Science. Provides curated content for data science tooling, such as Jupyter, collaborative functionality for every persona as well as its management, authentication and advanced features like data pipelines.

In this example each student is expected to have an account in the IDP connected to OpenShift. They will each get a ready to use JupyterLab environment authenticated via OpenShift's oauth.

The lab is created using the App of Apps pattern of ArgoCD: the [parent app](apps) is a Helm Chart that brings in a child app for each student, which in turn provisions all the necessary resources for a complete environment. In case of RHODS, the bare minimum setup requires a Custom Resource called `Notebook`, a `PersistentVolumeClaim`, a `Role` and a `RoleBinding`. These are found in the [child app Helm Chart](rhods-notebook).

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
      valuesObject: 
        base_domain: apps.your.ingress.controller.url
        workbench_name: python-lab33
    path: apps
    repoURL: 'https://github.com/nesanton/rhods-gitops'
    targetRevision: feature/namespaced
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - ApplyOutOfSyncOnly=true
```
* Tweak the user list in [values.yaml](apps/values.yaml). As soon as the changes are committed the Lab environment adjusts automatically by adding or deleting notebooks depending on the change.
* Once done with the lab - simply delete the parent app. The rest will be deleted automatically in cascade.

## Good to Know
### Preparing the end User environment

Each `Notebook` CR has an annotation `opendatahub.io/link` that points directly to the JupyterLab UI. Technically, this is the only piece of information that needs sharing with the Lab user (besides their login credentials for the connected Identity Provider).

In order to populate the empty notebook with some Lab material, one can replace the `/lab` at the end of the link with something like `/git-pull?repo=https%3A%2F%2Fgithub.com%2Fguimou%2Fdemo-notebooks.git&urlpath=lab%2Ftree%2Fdemo-notebooks.git%2Ftensorflow%2Ftensorflow_beginner.ipynb&branch=main` during the first launch. This example query string will populate the notebook from the https://github.com/guimou/demo-notebooks repo. For more details and examples see the [documentation](https://nbgitpuller.readthedocs.io/en/latest/) of [nbgitpuller](https://github.com/jupyterhub/nbgitpuller).

### Compute and Storage Resources

These are parametrized via the child Helm Chart [values.yaml](rhods-notebook/values.yaml). They don't have to match the T-shirt sizes provided in RHODS dashboard.

### Notebook container Image

It is also parametrized via the child Helm Chart [values.yaml](rhods-notebook/values.yaml).
