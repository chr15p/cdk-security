# Auditing
---
#### Warning

> As of Kubernetes 1.10 the auditing functionality is in beta so may change.

---

#### Warning

> In a 15 minute test the default policy produced 10MB of data so make sure log rotation and ageing are configured!

---

Kubernetes can provide an audit log of requests to the apiserver for pretty much any request made, and can include everything from who made the request when, to the full data they were sent.

To turn on auditing the kube-apiserver process needs to switches to be applied

audit-log-path  - the file to write the logs to

audit-policy-file  -  a file containing audit rules telling k8s what events to log

These settings are slightly complicated because in CDK kube-apiserver runs confined in a snap so these two files need to be placed somewhere the snap is allowed to access, and wont be removed by an upgrade. /var/snap/kube-apiserver/common/  is a good location, /root/cdk/ should also work.

This is pretty simple to configure in juju by setting the api-extra-args config option:

>  kubernetes-master:
>    charm: cs:~containers/kubernetes-master-78
>    expose: true
>    num_units: 1
>    options:
>      channel: 1.9/stable
>      api-extra-args: >
>        audit-log-path=/var/snap/kube-apiserver/common/k8s-audit.log
>        audit-policy-file=/var/snap/kube-apiserver/common/audit-policy.log



The policy file needs to be copied on to the master machine manually, for example:

juju scp audit-policy.log kuberentes-master/0:/var/snap/kube-apiserver/common/audit-policy.log

### Other options 

audit-log-maxage defined the maximum number of days to retain old audit log files

audit-log-maxbackup defines the maximum number of audit log files to retain

audit-log-maxsize defines the maximum size in megabytes of the audit log file before it gets rotated


### Policy files

Policy files look like standard k8s resource files but can't be loaded with "kubectl apply"  becuase Policy is not a valid type. Instead kube-apiserver reads the policies from a file at startup (or restart)

A very simple policy:

> # Log all requests at the Metadata level.
> apiVersion: audit.k8s.io/v1beta1
> kind: Policy
> rules:
> - level: Metadata

A slightly more complex policy:

> apiVersion: audit.k8s.io/v1beta1 # This is required.
> kind: Policy
> # Don't generate audit events for all requests in RequestReceived stage.
> omitStages:
>   - "RequestReceived"
> rules:
>  # Log pod changes at RequestResponse level
>  - level: Metadata
>    verb: ["get"]
>    resources:
>    - group: ""
>      # Resource "pods" doesn't match requests to any subresource of pods,
>      # which is consistent with the RBAC policy.
>      resources: ["pods"]


[https://kubernetes.io/docs/tasks/debug-application-cluster/audit/](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)

[https://github.com/kubernetes/kubernetes/blob/master/cluster/gce/gci/configure-helper.sh#L735] (audit profile used by GCE)



### Reading the files

The audit file is stored as json and can be examined with the usual tools:

> cat /var/snap/kube-apiserver/common/k8s-audit.log | python -m json.tool | less

