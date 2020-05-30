operatorhub-manual-approval
===========================

Demonstrates an automated, GitOps driven method for manual approvals of Operator Subscriptions.

## Overview

This example will demonstrate how to deploy the [Keycloak](https://www.keycloak.org/) operator into a new namespace called `argocd-olm-manual-approval` and create a subscription with a manual approval strategy for deploying the Keycloak operator. The subscription will then be approved by a Job to trigger the operator deployment. Finally, an instance of Keycloak will be provisioned.

In total, following actions occur:

1. `argocd-olm-manual-approval` Namespace created
2. A dedicated ServiceAccount and RBAC policies are created to aid in automatically approving the InstallPlan
3. A Subscription is created for the Keycloak operator
4. A job is launched to automatically approve InstallPlans that have yet to be approved that are specified by the Subscription.
5. `Keycloak` Custom Resource is created to deploy Keycloak. 

To facilitate the processes described above, [Sync Waves](https://argoproj.github.io/argo-cd/user-guide/sync-waves/) are used to govern the order of execution. Theh majority of the resources are created in the default Sync Wave. Afterward, the InstallPlan approver Job is implemented as an ArgoCD [Resource Hook](https://argoproj.github.io/argo-cd/user-guide/resource_hooks/) within its own Sync Wave. Finally, the `KeyCloak` custom resource is created to finish the automated provisioning. 

Based on the version on ArgoCD that you are using determines the steps necessary to demonstrate the solution:

## Deploying the Example

Before ArgoCD executes a Sync, it will attempt to perform a Dry Run which communicates against the Kubernetes cluster to verify the resources. However, since the OLM registers the Keycloak CRD's the Keycloak instance that is used in this example is not registered as a known API resource. To work around this issue, two ArgoCD applications must be created (One that manages the majority of the resources including the OLM components, and another application that contains just the Keycloak resource). A new option introduced in ArgoCD will provide an option to omit the validation that will not require the creation of two separate ArgoCD applications. 

Follow the steps listed below based on your ArgoCD version

### ArgoCD <1.6

1. Create a new ArgoCD application that will be used to deploy the baseline resources
   1. Specify a name of `keycloak-base`
   2. Specify a path `operatorhub-manual-approval/1.5-operator`
2. Create a second application that will be used to deploy the Keycloak instance
   1. Specify a name of `keycloak`
   2. Specify a path `operatorhub-manual-approval/1.5-instance`

Each application should sync and successfully deploy Keycloak

### ArgoCD 1.6+

With the release of version 1.6, ArgoCD supports providing the annotation `argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true` that will avoid having to create two separate applications. 

1. Create a new ArgoCD application to deploy Keycloak
   1. Specify a name of `keycloak`
   2. Specify a path `operatorhub-manual-approval/1.6`