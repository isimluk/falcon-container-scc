# falcon-container-SCC

This repository show-cases how to implement allow-listing of Falcon Container for OpenShift SCC (SecurityContextConstraints) based admission.
If you are looking to implement OPA Gatekeeper admission instead please consult [sister repository](https://github.com/isimluk/falcon-container-opa-templates).

## Error message that you may have seen

If you have seen the following error message you may find this text useful.
```
pods "XYZ" is forbidden: unable to validate against any security context constraint: [provider "anyuid": Forbidden: not usable by user or serviceaccount, spec.containers[0].securityContext.runAsUser: Invalid value: 0: must be in the ranges: [1000970000, 1000979999], spec.containers[0].securityContext.capabilities.add: Invalid value: "SYS_PTRACE": capability may not be added, provider "nonroot": Forbidden: not usable by user or serviceaccount, provider "pcap-dedicated-admins": Forbidden: not usable by user or serviceaccount, provider "hostmount-anyuid": Forbidden: not usable by user or serviceaccount, provider "machine-api-termination-handler": Forbidden: not usable by user or serviceaccount, provider "hostnetwork": Forbidden: not usable by user or serviceaccount, provider "hostaccess": Forbidden: not usable by user or serviceaccount, provider "splunkforwarder": Forbidden: not usable by user or serviceaccount, provider "node-exporter": Forbidden: not usable by user or serviceaccount, provider "privileged": Forbidden: not usable by user or serviceaccount]
```

# Introduction - What is SCC?

SecurityContextConstraints (SCCs) is OpenShift mechanism for allowing/denying pod deployments based on system level permissions that the pod requires.

The following command can be used to inspect currently installed SCC on the cluster:

    oc get scc
    
or

    kubectl get securitycontextconstraints.security.openshift.io

It is recommended to consult [OpenShift Documentation](https://docs.openshift.com/container-platform/4.9/authentication/managing-security-context-constraints.html) to understand SCC in detail. The bellow excerpt however is critical to understand the admission process:

> Admission uses the following approach to create the final security context for the pod:
>
>  - Retrieve all SCCs available for use.
>  - Generate field values for security context settings that were not specified on the request.
>  - Validate the final settings against the available constraints.
>
> If a matching set of constraints is found, then the pod is accepted. If the request cannot be matched to an SCC, the pod is rejected. A pod must validate every field against the SCC. 

# Enabling Falcon Container operations

Prior to the deployment of Falcon Container to OpenShift Container Platform, please make sure that SCC (SecurityContextConstraints)
allow for Falcon Container requirements to be enabled in any set of the SCCs.

    runAsUser:
      type: RunAsAny
    allowedCapabilities:
     - SYS_PTRACE

# Useful commands

After the pod is successfully accepted by SCC, following command can be used to see which particular SCC approved the given pod

    oc get pods my-test-pod-cddc6b8fd-pw4nn -o yaml | grep openshift.io/scc:

Deploy falcon-container SCC. This SCC is copy of `restricted` SCC with Falcon Container operations allowed:

    oc apply -f https://raw.githubusercontent.com/isimluk/falcon-container-scc/main/scc-falcon-container.yaml

Allowing any authenticated user to use the given SCC

    oc adm policy add-scc-to-group <scc-name> system:authenticated

## Useful links
 - https://docs.openshift.com/container-platform/4.9/authentication/managing-security-context-constraints.html
 - https://cloud.redhat.com/blog/managing-sccs-in-openshift

