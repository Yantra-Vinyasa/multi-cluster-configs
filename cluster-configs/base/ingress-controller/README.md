# Ingress-Controller 
The ingress certificate (also known as wildcard) is managed by the ingress Operator. The operator uses the certificate through the IngressController CR. The IngressController requires a TLS _secret_ with the certificate and its corresponding key in the `openshift-ingress` _namespace_.

## Key Capabilitis 

* Manages the namespace: openshift-ingress and deploys cluster resource: router-default pods
* Defines pod placement on specific nodes
* Defines ROUTE and NAMESPACE affinity 
* Defines replicas of router deployment
* Defines the DOMAIN for haproxy handling if not default
* Defines thread count for deployed router pods

## Verification
Run this command to verify that the certificate was successfully added.

```bash
echo Q |  openssl s_client -connect $(oc whoami --show-console | sed s'-https://--'):443 -showcerts 2>/dev/null |  openssl x509 -noout -subject -issuer -enddate

subject=O = EX.COM CN = *.apps.r6-ocp-mgmthub.ex.com
issuer=O = EX.COM, CN = Certificate Authority
notAfter=Apr 11 12:28:47 2026 GMT
```

## Documentation
* [Replacing the default ingress certificate](https://docs.openshift.com/container-platform/latest/security/certificates/replacing-default-ingress-certificate.html)
* [Setting a custom default ingress certificate](https://docs.openshift.com/container-platform/latest/networking/ingress-operator.html#nw-ingress-setting-a-custom-default-certificate_configuring-ingress)
