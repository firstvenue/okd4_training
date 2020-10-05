# Role Based Access Control <!-- omit in toc -->

Table of Contents

---
- [Projects and namespaces](#projects-and-namespaces)
	- [Default projects](#default-projects)
- [Role Based Access Control](#role-based-access-control)
	- [Default cluster roles](#default-cluster-roles)
	- [Evaluating authorization](#evaluating-authorization)
- [Manage Roles](#manage-roles)
	- [Local Role Binding Commands](#local-role-binding-commands)
	- [Cluster Role Binding Commands](#cluster-role-binding-commands)
	- [Create Custom Roles](#create-custom-roles)
- [Removing the kubeadmin user](#removing-the-kubeadmin-user)


# Projects and namespaces

A Kubernetes *namespace* provides a mechanism to scope resources in a cluster.

Namespaces provide a unique scope for:
- Named resources to avoid basic naming collisions.
- Delegated management authority to trusted users.
- The ability to limit community resource consumption.

Most objects in the system are scoped by namespace, but some are excepted and have no namespace, including nodes and users.

A *project* is a Kubernetes namespace with additional annotations and is the central vehicle by which access to resources for regular users is managed. A project allows a community of users to organize and manage their content in isolation from other communities. Users must be given access to projects by administrators, or if allowed to create projects, automatically have access to their own projects.


Projects can have a separate `name`, `displayName`, and `description`.
- The mandatory name is a unique identifier for the project and is most visible when using the CLI tools or API. The maximum name length is 63 characters.
- The optional displayName is how the project is displayed in the web console (defaults to name).
- The optional description can be a more detailed description of the project and is also visible in the web console.

Each project scopes its own set of:

| Object | Description |
| :--- | :--- |
| Objects | Pods, services, replication controllers, etc. |
| Policies | Rules for which users can or cannot perform actions on objects. |
| Constraints | Quotas for each kind of object that can be limited. |
| Service accounts | Service accounts act automatically with designated access to objects in the project. |


Cluster administrators can create projects and delegate administrative rights for the project to any member of the user community. Cluster administrators can also allow developers to create their own projects.



## Default projects
OKD comes with a number of default projects, and projects starting with `openshift-` are the most essential to users. These projects host master components that run as pods and other infrastructure components. The pods created in these namespaces that have a critical pod annotation are considered critical, and the have guaranteed admission by kubelet. Pods created for master components in these namespaces are already marked as critical.

# Role Based Access Control

Role-based access control (RBAC) objects determine whether a user is allowed to perform a given action within a project.

Cluster administrators can use the cluster roles and bindings to control who has various access levels to the OKD platform itself and all projects.

Developers can use local roles and bindings to control who has access to their projects. Note that authorization is a separate step from authentication, which is more about determining the identity of who is taking the action.


There are two levels of RBAC roles and bindings that control authorization:

| RBAC level | Description |
| :--- | :--- |
| Cluster RBAC | Roles and bindings that are applicable across all projects. Cluster roles exist cluster-wide, and cluster role bindings can reference only cluster roles. |
| Local RBAC | Roles and bindings that are scoped to a given project. While local roles exist only in a single project, local role bindings can reference both cluster and local roles. |

A cluster role binding is a binding that exists at the cluster level. A role binding exists at the project level.

## Default cluster roles
OKD includes a set of default cluster roles that you can bind to users and groups cluster-wide or locally. You can manually modify the default cluster roles, if required.

| Default Cluster Role | Description |
| :--- | :--- |
| admin | A project manager. If used in a local binding, an admin has rights to view any resource in the project and modify any resource in the project except for quota. |
| basic-user | A user that can get basic information about projects and users. |
| cluster-admin | A super-user that can perform any action in any project. When bound to a user with a local binding, they have full control over quota and every action on every resource in the project. |
| cluster-status | A user that can get basic cluster status information. |
| edit | A user that can modify most objects in a project but does not have the power to view or modify roles or bindings. |
| self-provisioner | A user that can create their own projects. |
| view | A user who cannot make any modifications, but can see most objects in a project. They cannot view or modify roles or bindings. |

> Be mindful of the difference between local and cluster bindings. For example, if you bind the `cluster-admin` role to a user by using a local role binding, it might appear that this user has the privileges of a cluster administrator. This is not the case. Binding the `cluster-admin` to a user in a project grants super administrator privileges for only that project to the user. That user has the permissions of the cluster role `admin`, plus a few additional permissions like the ability to edit rate limits, for that project. This binding can be confusing via the web console UI, which does not list cluster role bindings that are bound to true cluster administrators. However, it does list local role bindings that you can use to locally bind `cluster-admin`.

The relationships between cluster roles, local roles, cluster role bindings, local role bindings, users, groups and service accounts:
![image](images/roles_and_bindings_structure.png)


## Evaluating authorization
OKD evaluates authorization by using:

#### Identity <!-- omit in toc -->
The user name and list of groups that the user belongs to.

#### Action <!-- omit in toc -->
The action you perform. In most cases, this consists of:
 - **Project**: The project you access. A project is a Kubernetes namespace with additional annotations that allows a community of users to organize and manage their content in isolation from other communities.
 - **Verb**: The action itself: get, list, create, update, delete, deletecollection, or watch.
 - **Resource Name**: The API endpoint that you access.

#### Bindings <!-- omit in toc -->
The full list of bindings, the associations between users or groups with a role.

OKD evaluates authorization by using the following steps:
1. The identity and the project-scoped action is used to find all bindings that apply to the user or their groups.
2. Bindings are used to locate all the roles that apply.
3. Roles are used to find all the rules that apply.
4. The action is checked against each rule to find a match.
5. If no matching rule is found, the action is then denied by default.

> Remember that users and groups can be associated with, or bound to, multiple roles at the same time.

Project administrators can use the CLI to view local roles and bindings, including a matrix of the verbs and resources each are associated with.



# Manage Roles
You can use the `oc adm` administrator CLI to manage the roles and bindings.

Binding, or adding, a role to users or groups gives the user or group the access that is granted by the role. You can add and remove roles to and from users and groups using `oc adm policy` commands.

You can bind any of the default cluster roles to local users or groups in your project.



## Local Role Binding Commands

### Check which user can preform an action on a resource <!-- omit in toc -->
```bash
oc adm policy who-can <verb> <resource>
```

### Bind role to user in current project <!-- omit in toc -->
```bash
oc adm policy add-role-to-user <role> <user>
```

### Bind rule to a user in a specific project <!-- omit in toc -->
```bash
oc adm policy add-role-to-user <role> <user> -n <project>
```

For example add `admin` role to `alice` in project `joe`
```bash
oc adm policy add-role-to-user admin alice -n joe
```

### Remove role from user in current project <!-- omit in toc -->
```bash
oc adm policy remove-role-from-user <role> <user>
```

### Remove users and all their roles in current project <!-- omit in toc -->
```bash
oc adm policy remove-user <user>
```


### Bind role to group in current project <!-- omit in toc -->
```bash
oc adm policy add-role-to-group <role> <group>
```

### Bind rule to a group in a specific project <!-- omit in toc -->
```bash
oc adm policy add-role-to-group <role> <group> -n <project>
```

### Remove role from group in current project <!-- omit in toc -->
```bash
oc adm policy remove-role-from-group <role> <group>
```

### Remove groups and all their roles in current project <!-- omit in toc -->
```bash
oc adm policy remove-group <group>
```

### View local role bindings <!-- omit in toc -->
```bash
oc describe rolebinding.rbac -n <project>
```


## Cluster Role Binding Commands
### Bind role to user for all projects in the cluster <!-- omit in toc -->
```bash
oc adm policy add-cluster-role-to-user <role> <user>
```

### Remove role from user for all projects in the cluster <!-- omit in toc -->
```bash
oc adm policy remove-cluster-role-from-user <role> <user>
```

### Bind role to group for all projects in the cluster <!-- omit in toc -->
```bash
oc adm policy add-cluster-role-to-group <role> <group>
```
		
### Remove role from group for all projects in the cluster <!-- omit in toc -->
```bash
oc adm policy remove-cluster-role-from-group <role> <group>
```

### Create cluster admin <!-- omit in toc -->
Users with the **cluster-admin** default cluster role bound cluster-wide can perform any action on any resource, including viewing cluster roles and bindings.

```bash
oc adm policy add-cluster-role-to-user cluster-admin <user>
```

### View cluster role bindings <!-- omit in toc -->
```bash
oc describe clusterrolebinding.rbac
```


## Create Custom Roles
### Create local role <!-- omit in toc -->
```bash
oc create role <name> --verb=<verb> --resource=<resource> -n <project>
```

In this command, specify:
- `<name>`, the local role’s name
- `<verb>`, a comma-separated list of the verbs to apply to the role
- `<resource>`, the resources that the role applies to
- `<project>`, the project name

For example:
```bash
oc create role podview --verb=get --resource=pod -n blue
```

To bind the new role to a user:
```bash
oc adm policy add-role-to-user podview user2 --role-namespace=blue -n blue
```



### Create cluster role <!-- omit in toc -->
```bash
oc create clusterrole <name> --verb=<verb> --resource=<resource>
```

In this command, specify:
- `<name>`, the local role’s name
- `<verb>`, a comma-separated list of the verbs to apply to the role
- `<resource>`, the resources that the role applies to

For example:
```bash
oc create clusterrole podviewonly --verb=get --resource=pod
```

To bind the new role to a user:
```bash
oc adm policy add-cluster-role-to-user podviewonly user2
```

https://docs.okd.io/latest/authentication/using-rbac.html



# Removing the kubeadmin user

## The kubeadmin user  <!-- omit in toc -->
OKD creates a cluster administrator, `kubeadmin`, after the installation process completes.

This user has the `cluster-admin` role automatically applied and is treated as the root user for the cluster. The password is dynamically generated and unique to your OKD environment. After installation completes the password is provided in the installation program’s output. For example:

```bash
INFO Install complete!
INFO Run 'export KUBECONFIG=<your working directory>/auth/kubeconfig' to manage the cluster with 'oc', the OpenShift CLI.
INFO The cluster is ready when 'oc login -u kubeadmin -p <provided>' succeeds (wait a few minutes).
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.demo1.openshift4-beta-abcorp.com
INFO Login to the console with user: kubeadmin, password: <provided>
```


## Removing the kubeadmin user  <!-- omit in toc -->
After you define an identity provider and create a new `cluster-admin` user, you can remove the `kubeadmin` to improve cluster security.

> If you follow this procedure before another user is a `cluster-admin`, then OKD must be reinstalled. It is not possible to undo this command.

Prerequisites:
- You must have configured at least one identity provider.
- You must have added the `cluster-admin` role to a user.
- You must be logged in as an administrator.


Remove the `kubeadmin` secrets:
```bash
oc delete secrets kubeadmin -n kube-system
```

https://docs.okd.io/latest/authentication/remove-kubeadmin.html