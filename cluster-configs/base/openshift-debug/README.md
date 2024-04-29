# openshift-debug ns 
Add a toleration to a "openshift-debug" namespace, and let the debug pod to be created in this namespace with --to-namespace option:

`oc debug node/infra-0.example.com --to-namespace openshift-debug`
