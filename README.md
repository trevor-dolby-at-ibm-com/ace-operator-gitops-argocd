# ace-operator-gitops-argocd

Install the ACE operator using ArgoCD and create a Dashboard:

![ace-operator-gitops-light](/pictures/ace-operator-gitops-light.png#gh-light-mode-only)![ace-operator-gitops-dark](/pictures/ace-operator-gitops-dark.png#gh-dark-mode-only)

This repo currently ignores QA (placeholder directories only) but the same 
approach would work for QA and other environments. A similar approach would
also work for other CP4i operators.

## Getting started

Several pre-reqs are necessary before the application set can be created:
- ArgoCD must be running (tested using OpenShift GitOps 1.20.1)
- The `cp4i` namespace must exist and have the `ibm-entitlement-key` secret already in place. 
  This could also be automated using ArgoCD but is manual for the moment.
- ArgoCD needs permission to push to cp4i namespace. This can be achieved by various means.

Once these are in place, then the [application set](/ace-operator/ace-operator-application-set.yaml)
can be applied to the cluster to cause ArgoCD to begin the installation process. Errors are
expected to being with, as the dashboard YAML apply will fail until the operator is running
and the operator itself will have to wait for the catalog source.

## Notes

### Using cluster-wide install for the ACE operator

The operator could be installed in a specific namespace, ideally using another variant 
under the [ace-operator/variants](/ace-operator/variants/) directory. See https://www.ibm.com/docs/en/app-connect/13.0.x?topic=iaco-from-openshift-cli
for more details on per-namespace setup.

### ArgoCD could run in another cluster

The current setup is deploying to the local cluster using `server: https://kubernetes.default.svc`
but there is no reason ArgoCD needs to run locally and this could be changed.

### Problems with putting everything in one application

While it might seem that the second application (`ace-dashboard`) could be included in the 
`ace-operator` application, this can lead to issues during the initial apply: ArgoCD reports
```
The Kubernetes API could not find appconnect.ibm.com/Dashboard for requested 
resource cp4i/db-01-quickstart. Make sure the "Dashboard" CRD is installed on
the destination cluster.
```
when starting from a completely clean cluster (no ACE CRDs in place) because the Dashboard
doesn't exist at the time the apply is attempted.

Splitting the process into two applications works because only the `ace-dashboard`
apply fails initially, and it will succeed once the `ace-operator` process has completed.
