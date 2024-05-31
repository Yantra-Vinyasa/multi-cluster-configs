# Local Volume Set
This component installs Local Storage Operator (LSO), provisions persistent volumes based on local volumes and creates storage class for them.

In terms of availability, the basic requirement for Ceph/ODF is to deploy it on at least 3 nodes. Each of this nodes must have a dedicated storage volume (e.g. disk) and all volumes must be equal in size for optimal usage of storage. 

In VMWare environment the easiest way to provision such volumes is to use thin-csi driver which allows for dynamic provisioning of Kubernetes physical volumes stored on VMWare datastore. The limitation of thin-csi driver is that one thin-csi based storage class is tied to one datastore. If there's a need to use several datastores to put a volumes on them (e.g. each volume on different datastore), there must be as many thin-csi driver based storage classes as datastores to be used.

This approach is not suitable for ODF because the storagecluster object, which is essential for deploying a Ceph cluster in OCP, only allows a single storage class for its storage configuration.

If a dynamic provisioner like thin-csi cannot be used, the alternative is to use local node volumes, these volumes can be provisioned as Kubernetes Persistent Volumes using the Local Storage Operator (LSO).

LSO offers a Local Volume Set object that allows for the automatic provisioning of local PersistentVolumes based on specified criteria (such as type, size, vendor, etc.) and creates a storageclass for the provisioned volumes.

## How it works
Once the LSO is installed, several new custom resources are added, including the LocalVolumeSet. The main parameters of the LocalVolumeSet are as follows:

```
apiVersion: local.storage.openshift.io/v1alpha1
kind: LocalVolumeSet
metadata:
  name: lso-volumeset
  namespace: openshift-local-storage
spec:
  nodeSelector:
    nodeSelectorTerms:
    - matchExpressions:
      [...]
  tolerations:
  - effect: NoSchedule
    key: node.ocs.openshift.io/storage
    operator: Equal
    value: "true"
  deviceInclusionSpec:
    deviceTypes:
    - disk
    deviceMechanicalProperties:
      - NonRotational
    minSize: 512Gi
    maxSize: 512Gi
    models:
      - SAMSUNG
    vendors:
      - ST2000LM
  storageClassName: lso-volumeset
  volumeMode: Block
  maxDeviceCount: 1
```

The key point from the LocalVolumeSet definition is that it acts as a filter specifying the types of devices to be considered and their respective nodes. Once the Custom Resource is applied and matching devices are found, the following occurs:

* A storage class, as specified in the storageClassName parameter, is created.
* The expected number of Persistent Volumes that meet the criteria defined in the Custom Resource are created, each using the specified storage class.




## Requirements
minimum 3 nodes, each defined as
* having not less then 8 cpu cores and 32 Gib of RAM
* having labels and taints for storage node
* having addional disks attached 

## Useful commands

Check if the operator is installed successfully.  
```
oc get csv -n openshift-local-storage
```

Check if the storage class lso-volumeset exists
```
oc get sc 
```


Check provisioned devices and respective volumes :
```
oc get localvolumesets.local.storage.openshift.io  -n openshift-local-storage
```

```
oc get pv -l storage.openshift.com/owner-name=lso-volumeset

```


## Links of interest
[Automating discovery and provisioning for local storage devices](https://docs.openshift.com/container-platform/latest/storage/persistent_storage/persistent_storage_local/persistent-storage-local.html#local-storage-discovery_persistent-storage-local)

