# API Server
The _APIserver_ object in OpenShift is a custom resource that manages the different API servers in the platform. Most importantly the _kube-apiserver_, _openshift-apiserver_ and the _OAuth APIserver_. When discussing the API server in OpenShift we are most of the time referring to the kube-apiserver.

This API server acts as a hub for communication within a OpenShift cluster. It receives requests from various sources, including users, command-line tools, web interfaces, and other components in the cluster.

To enable encryption on etcd, we specify this by enabling it on the _APIserver_ object directly. When enabled, the following resources are encrypted in etcd.

* Secrets
* Config maps
* Routes
* OAuth access tokens
* OAuth authorize tokens

Available encryption types are:
* AES-CBC
* AES-GCM

## Certificate
The API server is using a certificate stored in HashiCorp Vault. It is referenced by the _externalSecret_ resource and created in the openshift-config _namespace_. The API server reads this secret.

## Link of interest 
* [Api server](https://docs.openshift.com/container-platform/latest/rest_api/config_apis/apiserver-config-openshift-io-v1.html)
* [Api server certificates](https://docs.openshift.com/container-platform/latest/security/certificates/api-server.html)
* [etcd encryption](https://docs.openshift.com/container-platform/latest/security/encrypting-etcd.html)

## Useful commands
Verify encryption is completed on the OpenShift API-server
```bash
oc get openshiftapiserver -o=jsonpath='{range .items[0].status.conditions[?(@.type=="Encrypted")]}{.reason}{"\n"}{.message}{"\n"}'

EncryptionCompleted
All resources encrypted: routes.route.openshift.io
```
Verify the same for the kube API-server
```bash
oc get kubeapiserver -o=jsonpath='{range .items[0].status.conditions[?(@.type=="Encrypted")]}{.reason}{"\n"}{.message}{"\n"}'

EncryptionCompleted
All resources encrypted: secrets, configmaps
```

Ensure that the OAuth APIserver also completed the encryption:
```bash
oc get authentication.operator.openshift.io -o=jsonpath='{range .items[0].status.conditions[?(@.type=="Encrypted")]}{.reason}{"\n"}{.message}{"\n"}'

EncryptionCompleted
All resources encrypted: oauthaccesstokens.oauth.openshift.io, oauthauthorizetokens.oauth.openshift.io
```