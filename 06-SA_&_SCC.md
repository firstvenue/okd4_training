# Service accounts and Security Context Constraints <!-- omit in toc -->

Table of Contents

---
- [Service accounts](#service-accounts)
	- [Default service accounts](#default-service-accounts)
	- [Manage Service Accounts](#manage-service-accounts)
- [Security Context Constraints](#security-context-constraints)



# Service accounts

A service account is an OKD account that allows a component to directly access the API. Service accounts are API objects that exist within each project. Service accounts provide a flexible way to control API access without sharing a regular user’s credentials.

When you use the OKD CLI or web console, your API token authenticates you to the API. You can associate a component with a service account so that they can access the API without using a regular user’s credentials. For example, service accounts can allow:
- Replication controllers to make API calls to create or delete pods.
- Applications inside containers to make API calls for discovery purposes.
- External applications to make API calls for monitoring or integration purposes.

Each service account’s user name is derived from its project and name:
> `system:serviceaccount:<project>:<name>`

Every service account is also a member of two groups:

| Group | Description |
| :--- | :--- |
| `system:serviceaccounts` | Includes all service accounts in the system. |
| `system:serviceaccounts:<project>` | Includes all service accounts in the specified project. |


Each service account automatically contains two secrets:
- An API token
- Credentials for the OpenShift Container Registry

## Default service accounts

Your OKD cluster contains default service accounts for cluster management and generates more service accounts for each project.

### Default cluster service accounts <!-- omit in toc -->

Several infrastructure controllers run using service account credentials. The following service accounts are created in the OKD infrastructure project (openshift-infra) at server start, and given the following roles cluster-wide:
| Service Account | Description |
| :--- | :--- |
| replication-controller | Assigned the `system:replication-controller` role |
| deployment-controller | Assigned the `system:deployment-controller` role |
| build-controller | Assigned the `system:build-controller` role. Additionally, the `build-controller` service account is included in the privileged Security Context Constraint in order to create privileged build pods. |


### Default project service accounts and roles <!-- omit in toc -->

Three service accounts are automatically created in each project:
| Service Account | Usage |
| :--- | :--- |
| builder | Used by build pods. It is given the `system:image-builder` role, which allows pushing images to any imagestream in the project using the internal Docker registry. |
| deployer | Used by deployment pods and given the `system:deployer` role, which allows viewing and modifying replication controllers and pods in the project. |
| default | Used to run all other pods unless they specify a different service account. |

All service accounts in a project are given the `system:image-puller` role, which allows pulling images from any imagestream in the project using the internal container image registry.

https://docs.okd.io/latest/authentication/understanding-and-creating-service-accounts.html
https://docs.okd.io/latest/authentication/using-service-accounts-in-applications.html

## Manage Service Accounts

### View the service accounts in the current project <!-- omit in toc -->
```bash
oc get sa
```

### Create a new service account in the current project <!-- omit in toc -->
```bash
oc create sa <service_account_name>
```

### View details about a service account <!-- omit in toc -->
```bash
oc describe sa/<service_account_name>
```

### Grant role to service account for a project <!-- omit in toc -->
```bash
oc adm policy add-role-to-user <role> system.serviceaccount.<project>.<service_account_name>
```



# Security Context Constraints


Similar to the way that RBAC resources control user access, administrators can use Security Context Constraints (SCCs) to control permissions for pods. These permissions include actions that a pod, a collection of containers, can perform and what resources it can access. You can use SCCs to define a set of conditions that a pod must run with in order to be accepted into the system.

SCCs allow an administrator to control:
- Whether a pod can run privileged containers.
- The capabilities that a container can request.
- The use of host directories as volumes.
- The SELinux context of the container.
- The container user ID.
- The use of host namespaces and networking.
- The allocation of an FSGroup that owns the pod’s volumes.
- The configuration of allowable supplemental groups.
- Whether a container requires the use of a read only root file system.
- The usage of volume types.
- The configuration of allowable seccomp profiles.



https://docs.okd.io/latest/authentication/managing-security-context-constraints.html