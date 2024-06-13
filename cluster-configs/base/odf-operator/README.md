# Openshift Data Foundation
This component installs Openshift Data Foundation operator and configures storage cluster on storage nodes.

## Introduction
Openshift Data Foundation integrates several open source projects:

* Ceph to provide block storage (ceph-rbd), shared and distributed filesystem (cephfs), and object storage (ceph-rgw)
* Ceph CSI to provide dynamic provisioning of persistent volumes
* Rook that integrates Ceph and Ceph CSI into Kubernetes
* NooBaa providing Multi Object Gateway for accessing and managing cloud based object storage

All of the above are integrated into Openshift platform by a collection of operators:

* odf-operator - a meta operator making sure that other dependent operators are installed properly and providing the integrations with Openshift platform (console plugin and API)
* ocs-operator - a meta operator which provides Custom Resources that trigger other operators to reconcile against them, it also abstracts Ceph and Multi Object Gateway configurations in par with Red Hat best practices and creates and reconciles resources needed by Ceph and NoobBaa
* rook-ceph-operator - Ceph storage system enablement on Openshift, it boostraps the storage clusters, all required components (ceph-csi driver, mons, osds, mgr, rgw and mds daemons) and monitors the storage daemons
* mcg-operator provides Multi Object Gateway functionality

## How it works
After the ODF operator installation the following pods should be in ready state:

```
oc get pods -n openshift-storage
NAME                                              READY   STATUS    RESTARTS      AGE
csi-addons-controller-manager-566c4b7c95-6cfc2    2/2     Running   0             47h
noobaa-operator-5f95455bb6-7r69k                  1/1     Running   0             47h
ocs-metrics-exporter-586f55d968-vcdfc             1/1     Running   0             47h
ocs-operator-58659bf8ff-lhp45                     1/1     Running   0             47h
odf-console-855c866c9f-qsx4p                      1/1     Running   0             47h
odf-operator-controller-manager-f9ffd7b59-zs97z   2/2     Running   0             47h
rook-ceph-operator-f7d9d5f58-5gz5s                1/1     Running   0             47h
```

Once all the operators are installed, several new custom resources are added, including the StorageCluster. The main parameters are as follows:

```
apiVersion: ocs.openshift.io/v1
kind: StorageCluster
metadata:
  name: ocs-storagecluster
  namespace: openshift-storage
spec:
  monDataDirHostPath: /var/lib/rook
  resources:
[...]
  storageDeviceSets:
  - count: 3
    dataPVCTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 512Gi
        storageClassName: lso-volumeset
        volumeMode: Block
    name: ocs-deviceset-lso-volumeset
    replica: 3
    resources:
[...]
  nodeTopologies:
    labels:
      cluster.ocs.openshift.io/openshift-storage:
  placement:
    all:
      tolerations:
        - effect: NoSchedule
          key: node.ocs.openshift.io/storage
          value: "true"
        - effect: NoSchedule
          key: node-role.kubernetes.io/infra
          value: reserved
        - effect: NoExecute
          key: node-role.kubernetes.io/infra
          value: reserved
[...]
```

In this example the most important sections are shown: storageDeviceSets, nodeTopologies and placement.

The `storageDeviceSets` section specifies the Ceph cluster topology and the storage devices used by the cluster. 

* The `count` parameter defines how many OSD pods should be deployed per storage node. we have 3 disks per node for ODF, so count is set to 3.
* The `replica` parameter defines the level of redundancy to be achieved. The default replication count for Ceph pools is 3. This is to ensure redundancy of data in case of disk failures. This also means 12TB of raw disk equals 4TB of useable storage.

* The `nodeTopologies`,`placement` saction defines where all ODF components should be placed.


## Useful commands

```
 oc get storagecluster -n openshift-storage
```

```
oc get cephclusters -n openshift-storage
```

It is important to observe how the components of the ODF installation are distributed across the cluster. The core components of the Ceph cluster—OSDs, MONs, MDS, and MGR—are all located exclusively on storage nodes:

```
oc get pods -n openshift-storage -l app=rook-ceph-osd -o wide
```

```
oc get pods -n openshift-storage -l app=rook-ceph-mon -o wide
```
```
oc get pods -n openshift-storage -l app=rook-ceph-mds -o wide
```
```
oc get pods -n openshift-storage -l app=rook-ceph-mgr -o wide
```


## Requirements
3 nodes, each defined as
* have 16 vCPUs and 64GB of RAM or higher
* has label and taints  
* has addtional disks attached
* storageclass lso-volumeset exists
* additional disks mentioned above are provisioned by Local Storage Operator 

### Calculate usable disk capacity 

VMDK disk Size (300 GB) x 3 vmdk/node x 3 ODF Node = 2700 GB (Total) / 3 (Ceph pool Replications) = 900 GB (Usable)


## Scaling up storage

WIP....


## Ceph Toolbox

The Ceph Toolbox pod will be implemented with every ODF deployment. The details of the configuration procedure is described in [this article](https://access.redhat.com/articles/4628891).
This pod can be used to check the status of Ceph cluster using its internal CLI. Here are some examples of how to use it: 

```
 # oc rsh  $(oc get pods -n openshift-storage -l app=rook-ceph-tools -o name)
 sh-4.4$ ceph status
 
 sh-4.4$ ceph osd tree

 sh-4.4$ ceph osd df

 sh-4.4$ ceph device ls

```


## Links of interest
[Deploying OpenShift Data Foundation on VMware vSphere](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.14/html-single/deploying_openshift_data_foundation_on_vmware_vsphere/index#installing-local-storage-operator_local-storage)

[ODF CPU and RAM Resource Requirements](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.14/html-single/planning_your_deployment/index#resource-requirements_rhodf)

[ODF Storage Capacity Planning](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.14/html-single/planning_your_deployment/index#capacity_planning)
[Scaling up a ODF cluster created using local storage devices](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.14/html-single/scaling_storage/index#scaling-up-storage-by-adding-capacity-to-openshift-data-foundation-nodes-using-local-storage-devices_local-vmware)

[How to use dedicated worker nodes for Red Hat OpenShift Data Foundation](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.14/html-single/managing_and_allocating_storage_resources/index#how-to-use-dedicated-worker-nodes-for-openshift-data-foundation_rhodf)

[Managing container storage interface (CSI) component placements (related to rook-ceph-operator-config ConfigMap)](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.14/html-single/managing_and_allocating_storage_resources/index#managing-container-storage-interface-component-placements_rhodf)

[Changing resources for the OpenShift Data Foundation components](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.14/html-single/troubleshooting_openshift_data_foundation/index#changing-resources-for-the-openshift-data-foundation-components_rhodf)

[Ceph documentation](https://docs.ceph.com/en/latest/#)

[Rook Ceph documentation](https://rook.github.io/docs/rook/latest/Getting-Started/intro/)

[OCS/ODF Installation and Configuration Training](https://red-hat-storage.github.io/ocs-training/training/index.html)

[How to configure Ceph toolbox in OpenShift Data Foundation 4.x ](https://access.redhat.com/articles/4628891)

[Supported configurations for Red Hat OpenShift Data Foundation 4.X](https://access.redhat.com/articles/5001441)
