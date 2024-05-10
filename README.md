# multi-cluster-configs

## Install Bootstrap

1. Download and install  kustomize cli https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/


2. check the overlay directory structure:

```bash

$ cd multi-cluster-configs/bootstrap

$ tree
.
├── base
│   └── gitops-operator
│       ├── kustomization.yaml
│       ├── operator
│       │   ├── og.yaml
│       │   ├── operator-ns.yaml
│       │   └── subscription.yaml
│       └── resources
│           ├── argo-ns.yaml
│           ├── crb.yaml
│           ├── instance.yaml
│           └── trusted-ca-bundle-cm.yaml
└── overlays
    ├── aro
    │   └── gitops-operator
    │       ├── appofapp.yaml
    │       ├── kustomization.yaml
    │       ├── patch-rbac.yaml
    │       └── patch-sub-channel.yaml
    ├── datacenter
    │   └── gitops-operator
    │       ├── appofapp.yaml
    │       ├── kustomization.yaml
    │       ├── patch-rbac.yaml
    │       └── patch-sub-channel.yaml
    ├── gcp
    │   └── gitops-operator
    │       ├── appofapp.yaml
    │       ├── kustomization.yaml
    │       ├── patch-rbac.yaml
    │       └── patch-sub-channel.yaml
    └── rosa
        └── gitops-operator
            ├── appofapp.yaml
            ├── kustomization.yaml
            ├── patch-rbac.yaml
            └── patch-sub-channel.yaml

```

change the values of the `targetRevision`, `repoURL`, and `path` on the `appofapp.yaml` file in the overlays folders to match your repo details


3. Make sure the GitOps operator version is the latest in file `patch-sub-channel.yaml` --> `value`


4. run on the `overlay/{env}/gitops-operator` folder: 

$ kustomize build|oc apply -f -

ArgoCD will be installed and the `appofapp` will be created
