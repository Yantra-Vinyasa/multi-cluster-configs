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


# Install App of Apps

1. Use the overlay folder relevant to your environment in the multi-cluster-configs/cluster-configs/overlays/{env}/app-factory.

2. Check the values of the `repoURL`, `targetRevision`, and `applications.pacman.source.path`, update if necessary.

3. There are other applications commented out in the same file, uncomment if you want to use them.

4. run on the `multi-cluster-configs/bootstrap/overlays/{env}/gitops-operator` folder:

$ kustomize build|oc apply -f -

ArgoCD will be installed and the `appofapp` will be created
