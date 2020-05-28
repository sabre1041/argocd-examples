operatorhub-manual-approval
===========================

Demonstrates an automated, GitOps driven method for manual approvals of Operator Subscriptions.

## Overview

This example will deploy the [Keycloak](https://www.keycloak.org/) operator into a new namespace called `argocd-olm-manual-approval` and create a subscription with a manual approval strategy for deploying the Keycloak operator. The subscription will then be approved by a Job to trigger the operator deployment. Finally, an instance of Keycloak will be provisioned.

The following actions occur:

1. `argocd-olm-manual-approval` Namespace created
2. A dedicated ServiceAccount and RBAC policies are created to aid in automatically approving the InstallPlan
3. A Subscription is created for the Keycloak operator
4. A job is launched to automatically approve all InstallPlans
5. `Keycloak` Custom Resource is created to deploy Keycloak. 

To facilitate the processes described above, [Sync Waves](https://argoproj.github.io/argo-cd/user-guide/sync-waves/) are used to govern the order of execution. Theh majority of the resources are created in the default Sync Wave. Afterward, the InstallPlan approver Job is implemented as an ArgoCD [Resource Hook](https://argoproj.github.io/argo-cd/user-guide/resource_hooks/) within its own Sync Wave. Finally, the `KeyCloak` custom resource is created to finish the automated provisioning. 
