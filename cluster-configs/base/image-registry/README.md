# Image registry
This component manages the image registry (`configs.imageregistry.operator.openshift.io`) in the openshift-image-registry _project_.
The `Image Registry Operator` runs in the openshift-image-registry _namespace_, and manages the registry instance in that location as well. All configuration and workload resources for the registry reside in that namespace.

On platforms that do not provide shareable object storage, the OpenShift Image Registry Operator bootstraps itself as Removed. This allows openshift-installer to complete installations on these platform types.
After installation, you must edit the Image Registry Operator configuration to switch the managementState from Removed to Managed. This is done here through the YAML together with configuring it to run on infra nodes and use persistent storage.

It will set it up to use ODF 

Use the cluster specific values.yaml files to enable/disable the managed state of the registry and also handle the credentials.
## Configuration
By the use of an overlay it is possible to configure things such as, amount of replicas, pvc to be used for storage and more.

## About the image registry
Further reading can be done using the official documentation.

[OpenShift documentation](https://docs.openshift.com/container-platform/latest/registry/configuring-registry-operator.html)
