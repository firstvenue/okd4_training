# Manage Users and Groups <!-- omit in toc -->

Table of Contents

---
- [Understanding authentication](#understanding-authentication)
- [Users](#users)
- [Groups](#groups)
- [Identity provider](#identity-provider)
- [Configuring an HTPasswd identity provider](#configuring-an-htpasswd-identity-provider)
- [Manage Users](#manage-users)
- [Manage Groups](#manage-groups)



# Understanding authentication

For users to interact with OKD, they must first authenticate to the cluster. The authentication layer identifies the user associated with requests to the OKD API. The authorization layer then uses information about the requesting user to determine if the request is allowed.


# Users

A user in OKD is an entity that can make requests to the OKD API. An OKD user object represents an actor which can be granted permissions in the system by adding roles to them or to their groups. Typically, this represents the account of a developer or administrator that is interacting with OKD.

**User Types:**

| User type | Description |
| :--- | :--- |
| Regular users | This is the way most interactive OKD users are represented. Regular users are created automatically in the system upon first login or can be created via the API. Regular users are represented with the User object. Examples: `joe` `alice` |
| System users | Many of these are created automatically when the infrastructure is defined, mainly for the purpose of enabling the infrastructure to interact with the API securely. They include a cluster administrator (with access to everything), a per-node user, users for use by routers and registries, and various others. Finally, there is an `anonymous` system user that is used by default for unauthenticated requests. Examples: `system:admin` `system:openshift-registry` `system:node:node1.example.com` |
| Service accounts | These are special system users associated with projects; some are created automatically when the project is first created, while project administrators can create more for the purpose of defining access to the contents of each project. Service accounts are represented with the ServiceAccount object. Examples: `system:serviceaccount:default:deployer` `system:serviceaccount:foo:builder` |

> Each user must authenticate in some way in order to access OKD. API requests with no authentication or invalid authentication are authenticated as requests by the anonymous system user. Once authenticated, policy determines what the user is authorized to do.


# Groups

A user can be assigned to one or more groups, each of which represent a certain set of users. Groups are useful when managing authorization policies to grant permissions to multiple users at once.

In addition to explicitly defined groups, there are also system groups, or virtual groups, that are automatically provisioned by the cluster.

**Most inportant default virtual groups:**
| Virtual group | Description |
| :--- | :--- |
| system:authenticated | Automatically associated with all authenticated users. |
| system:authenticated:oauth | Automatically associated with all users authenticated with an OAuth access token. |
| system:unauthenticated | Automatically associated with all unauthenticated users. |

> To disable login for a user you can remove it from `system:authenticated` and `system:authenticated:oauth` groups

# Identity provider

The OKD master includes a built-in OAuth server. Developers and administrators obtain OAuth access tokens to authenticate themselves to the API.
As an administrator, you can configure OAuth to specify an identity provider after you install your cluster.

By default, only a `kubeadmin` user exists on your cluster. To specify an identity provider, you must create a Custom Resource (CR) that describes that identity provider and add it to the cluster.

[Supported identity providers](https://docs.okd.io/latest/authentication/understanding-identity-provider.html#supported-identity-providers)


# Configuring an HTPasswd identity provider

https://docs.okd.io/latest/authentication/identity_providers/configuring-htpasswd-identity-provider.html#configuring-htpasswd-identity-provider

To define an HTPasswd identity provider you must perform the following steps:
1. Create an htpasswd file to store the user and password information.
2. Create an OKD secret to represent the htpasswd file.
3. Define the HTPasswd identity provider resource.
4. Apply the resource to the default OAuth configuration.


## Creating an HTPasswd file <!-- omit in toc -->

First you must generate a flat file that contains the user names and passwords for your cluster by using `htpasswd`.

[Man page for htpasswd](https://linux.die.net/man/1/htpasswd)
```bash
htpasswd -c -B -b </path/to/users.htpasswd> <user_name> <password>
```

Example:
```bash
htpasswd -c -B -b users.htpasswd user1 MyPassword!
```
Output should be something like this:
```bash
Adding password for user user1
```



Continue to add or update credentials to the file:
```bash
htpasswd -B -b </path/to/users.htpasswd> <user_name> <password>
```

## Creating the HTPasswd Secret <!-- omit in toc -->

When you have your htpasswd file, you need to create a OKD Secret resource from it for OKD to use the htpasswd identity provider. This step needs to be done every time the htpasswd file is updated in order to update the OKD Secret as well.

Create an OKD Secret that contains the HTPasswd users file:
```bash
oc create secret generic htpass-secret --from-file=htpasswd=</path/to/users.htpasswd> -n openshift-config
```

## Create Sample HTPasswd Custom Resource <!-- omit in toc -->

Next thing we need to create the Custom Resource (CR) for our new Identity Provider.
Something like this:
[Example CR](files/htpasswd_cr.yaml) 

## Adding an identity provider to your clusters <!-- omit in toc -->

1. Apply the defined CR:
```bash
oc apply -f </path/to/CR>
```
2. Log in to the cluster as a user from your identity provider:
```bash
oc login -u <username>
```
3. Confirm that the user logged in successfully:
```bash
oc whoami
```


# Manage Users
We can't manage the htpasswd file in OKD itself.
We need to extract it from Secret, modify and then replace it.

## Add Users <!-- omit in toc -->

1. Confirm secret name:
```bash
oc get secret -n openshift-config
```
	
2. Retrieve htpasswd file from Secret:
```bash
oc extract secret/HTPASSWD_SECRET --to - -n openshift-config > users.htpasswd
```

3. Add new user:
```bash
htpasswd -b -B htpasswd <new_user_name> <password>
```

4. Replace Secret with our new Secret:
```bash
oc create secret generic HTPASSWD_SECRET --from-file=users.htpasswd --dry-run -o yaml -n openshift-config | oc replace -f -
```

5. Check identity:
```bash
oc get identity
```

6. Verify:
```bash
oc login -u <new_user_name>

oc whoami
```


## Remove Users <!-- omit in toc -->
It will be mostly the same process as adding a new user.

1. Confirm secret name:
```bash
oc get secret -n openshift-config
```
	
2. Retrieve htpasswd file from Secret:
```bash
oc extract secret/HTPASSWD_SECRET --to - -n openshift-config > users.htpasswd
```

3. Remove user:

Simply edit the file and delete the line with the username and password hash:
```bash
vim users.htpasswd
```
or remove with htpasswd:
```bash
htpasswd -D users.htpasswd <username>
```

4. Replace Secret with our new Secret:
```bash
oc create secret generic HTPASSWD_SECRET --from-file=users.htpasswd --dry-run -o yaml -n openshift-config | oc replace -f -
```

5. Remove user's Identity:
```bash
oc get identity

oc delete identity/HTPASSWD_PROVIDER:<username>
```

6. Remove user from the cluster:
```bash
oc get users

oc delete user/<username>
```

> User needs to be removed, otherwise the user can continue using their token as long as it has not expired.

7. Wait for OAuth pod to restart then verify:
```bash
oc get users

oc get identity
```


# Manage Groups

## Add new group <!-- omit in toc -->
### Create group <!-- omit in toc -->
```bash
oc adm groups new <group>
```

### Create new group with one user <!-- omit in toc -->
```bash
oc adm groups new <group> <user>
```

## Add users to group <!-- omit in toc -->
### Add users to existing group <!-- omit in toc -->
```bash
oc adm groups add-users <group> <user1> <user2>
```

## Remove group <!-- omit in toc -->
### Remove user from group <!-- omit in toc -->
```bash
oc adm groups remove-users <group> <user1> <user2>
```
