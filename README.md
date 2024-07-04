# multi-cluster-configs

## Top level directories:

* bootstrap: Contains configuration for bootstrapping the GitOps System.

* cluster-configs/base: Components that need to be installed or configured in the cluster.

* cluster-configs/overlay: Selects components to be installed into the cluster.

* tools : Helper scripts (e.g. setting up CI for gitlab & linting ).


### GitOps:
* [What is GitOps](https://opengitops.dev/)
* [App of Apps pattern](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/)
  
### Bootstrap process:

Configuring a new cluster (e.g. here dev cluster ) requires the execution of the following commands :

1. Install the OpenShift GitOps Operator

```
kustomize build bootstrap/overlays/dev/gitops-operator | oc apply -f - 
```

2. Install the either Sealed Secret controller or external-secrets-operator 

 
