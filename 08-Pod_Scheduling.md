# Pod Scheduling <!-- omit in toc -->

Table of Contents

---
- [Scaling Pods Manually](#scaling-pods-manually)
- [Horizontal pod autoscalers](#horizontal-pod-autoscalers)
- [Resource quotas](#resource-quotas)
  - [Compute resources managed by quota](#compute-resources-managed-by-quota)
  - [Storage resources managed by quota](#storage-resources-managed-by-quota)
  - [Object counts managed by quota](#object-counts-managed-by-quota)
  - [Quota scopes](#quota-scopes)
  - [Quota enforcement](#quota-enforcement)
  - [Requests versus limits](#requests-versus-limits)
  - [Creating a quota](#creating-a-quota)
  - [Create object count quotas](#create-object-count-quotas)
  - [Viewing a quota](#viewing-a-quota)
- [Clusterquotas](#clusterquotas)
- [Control Pod Placement Across Cluster Nodes (scheduling)](#control-pod-placement-across-cluster-nodes-scheduling)


# Scaling Pods Manually
With the help of deployments and replication controllers we can define the number of concurently running pods for our application.

### Set replica count <!-- omit in toc -->
```bash
oc scale --replicas=<number> dc/<deployment_name>
```
			
example:

```bash
oc scale --replicas=2 dc/ruby-hello-world
```

or you can edit the deployment and set the replica count there

```bash
oc edit dc/<deployment_name>
```


# Horizontal pod autoscalers
You can use a horizontal pod autoscaler (HPA) to specify how OKD should automatically increase or decrease the scale of a replication controller or deployment configuration.

You can create a horizontal pod autoscaler to specify the minimum and maximum number of pods you want to run, as well as the CPU utilization or memory utilization your pods should target.

After you create a horizontal pod autoscaler, OKD begins to query the CPU and/or memory resource metrics on the pods. When these metrics are available, the horizontal pod autoscaler computes the ratio of the current metric utilization with the desired metric utilization, and scales up or down accordingly.

In order to use horizontal pod autoscalers, your cluster administrator must have properly configured cluster metrics.
You can use the `oc describe PodMetrics <pod-name>` command to determine if metrics are configured. If metrics are configured, the output appears similar to the following, with `Cpu` and `Memory` displayed under `Usage`.

Example:

```bash
oc describe PodMetrics openshift-kube-scheduler-ip-10-0-135-131.ec2.internal
```

Example output:
```shell
Name:         openshift-kube-scheduler-ip-10-0-135-131.ec2.internal
Namespace:    openshift-kube-scheduler
Labels:       <none>
Annotations:  <none>
API Version:  metrics.k8s.io/v1beta1
Containers:
  Name:  wait-for-host-port
  Usage:
	Memory:  0
  Name:      scheduler
  Usage:
	Cpu:     8m
	Memory:  45440Ki
Kind:        PodMetrics
Metadata:
  Creation Timestamp:  2019-05-23T18:47:56Z
  Self Link:           /apis/metrics.k8s.io/v1beta1/namespaces/openshift-kube-scheduler/pods/openshift-kube-scheduler-ip-10-0-135-131.ec2.internal
Timestamp:             2019-05-23T18:47:56Z
Window:                1m0s
Events:                <none>
```

### Create horizontal pod autoscaler for CPU utilization <!-- omit in toc -->
```bash
oc autoscale dc/<dc-name> --min <number> --max <number> --cpu-percent=<percent>
```

Example:

```bash
oc autoscale dc/ruby-hello-world --min 1 --max 5 --cpu-percent=75
```


### Create deployment file for HPA <!-- omit in toc -->
Create the resource yaml file for HPA like this: [HorizontalPodAutoscaler.yaml](files/HorizontalPodAutoscaler.yaml)

then apply it to your project:

```bash
oc create -f HorizontalPodAutoscaler.yaml
```


### Check horizontal pod autoscalers <!-- omit in toc -->
```bash
oc get hpa -n <project>
```
			
### Edit horizontal pod autoscaler <!-- omit in toc -->
```bash
oc edit hps <hpa_name>
```

### Check HPA status <!-- omit in toc -->
```bash
oc describe hpa <hpa_name>
```

Example output:
```shell
Name:                        hpa-resource-metrics-memory
Namespace:                   default
Labels:                      <none>
Annotations:                 <none>
CreationTimestamp:           Wed, 04 Mar 2020 16:31:37 +0530
Reference:                   ReplicationController/example
Metrics:                     ( current / target )
  resource memory on pods:   2441216 / 500Mi
Min replicas:                1
Max replicas:                10
ReplicationController pods:  1 current / 1 desired
Conditions:
  Type            Status  Reason              Message
  ----            ------  ------              -------
  AbleToScale     True    ReadyForNewScale    recommended size matches current size
  ScalingActive   True    ValidMetricFound    the HPA was able to successfully calculate a replica count from memory resource
  ScalingLimited  False   DesiredWithinRange  the desired count is within the acceptable range
Events:
  Type     Reason                   Age                 From                       Message
  ----     ------                   ----                ----                       -------
  Normal   SuccessfulRescale        6m34s               horizontal-pod-autoscaler  New size: 1; reason: All metrics below target
```

AbleToScale:
> indicates whether HPA is able to fetch and update metrics, as well as whether any backoff-related conditions could prevent scaling.
- A `True` condition indicates scaling is allowed.
- A `False` condition indicates scaling is not allowed for the reason specified.

ScalingActive:
> indicates whether the HPA is enabled (for example, the replica count of the target is not zero) and is able to calculate desired metrics.
- A `True` condition indicates metrics is working properly.
- A `condition` generally indicates a problem with fetching metrics.

ScalingLimited:
> indicates that the desired scale was capped by the maximum or minimum of the horizontal pod autoscaler.
- A `True` condition indicates that you need to raise or lower the minimum or maximum replica count in order to scale.
- A `False` condition indicates that the requested scaling is allowed.

https://docs.okd.io/latest/nodes/pods/nodes-pods-autoscaling.html



# Resource quotas
A resource quota, defined by a `ResourceQuota` object, provides constraints that limit aggregate resource consumption per project. It can limit the quantity of objects that can be created in a project by type, as well as the total amount of compute resources and storage that may be consumed by resources in that project.


The following describes the set of compute resources and object types that can be managed by a quota.

## Compute resources managed by quota
| Resource Name | Description |
| :--- | :--- |
| cpu | The sum of CPU requests across all pods in a non-terminal state cannot exceed this value. `cpu` and `requests.cpu` are the same value and can be used interchangeably. |
| memory | The sum of memory requests across all pods in a non-terminal state cannot exceed this value. `memory` and `requests.memory` are the same value and can be used interchangeably. |
| ephemeral-storage | The sum of local ephemeral storage requests across all pods in a non-terminal state cannot exceed this value. `ephemeral-storage` and `requests.ephemeral-storage` are the same value and can be used interchangeably. This resource is available only if you enabled the ephemeral storage technology preview. This feature is disabled by default. |
| requests.cpu | The sum of CPU requests across all pods in a non-terminal state cannot exceed this value. `cpu` and `requests.cpu` are the same value and can be used interchangeably. |
| requests.memory | The sum of memory requests across all pods in a non-terminal state cannot exceed this value. `memory` and `requests.memory` are the same value and can be used interchangeably. |
| requests.ephemeral-storage | The sum of ephemeral storage requests across all pods in a non-terminal state cannot exceed this value. `ephemeral-storage` and `requests.ephemeral-storage` are the same value and can be used interchangeably. This resource is available only if you enabled the ephemeral storage technology preview. This feature is disabled by default. |
| limits.cpu | The sum of CPU limits across all pods in a non-terminal state cannot exceed this value. |
| limits.memory | The sum of memory limits across all pods in a non-terminal state cannot exceed this value. |
| limits.ephemeral-storage | The sum of ephemeral storage limits across all pods in a non-terminal state cannot exceed this value. This resource is available only if you enabled the ephemeral storage technology preview. This feature is disabled by default. |

## Storage resources managed by quota
|Resource Name | Description |
| :--- | :--- |
| requests.storage | The sum of storage requests across all persistent volume claims in any state cannot exceed this value. |
| persistentvolumeclaims | The total number of persistent volume claims that can exist in the project. |
| <storage-class-name>.storageclass.storage.k8s.io/requests.storage | The sum of storage requests across all persistent volume claims in any state that have a matching storage class, cannot exceed this value. |
| <storage-class-name>.storageclass.storage.k8s.io/persistentvolumeclaims | The total number of persistent volume claims with a matching storage class that can exist in the project. |

## Object counts managed by quota
| Resource Name | Description |
| :--- | :--- |
| pods | The total number of pods in a non-terminal state that can exist in the project. |
| replicationcontrollers | The total number of ReplicationControllers that can exist in the project. |
| resourcequotas | The total number of resource quotas that can exist in the project. |
| services | The total number of services that can exist in the project. |
| services.loadbalancers | The total number of services of type LoadBalancer that can exist in the project. |
| services.nodeports | The total number of services of type NodePort that can exist in the project. |
| secrets | The total number of secrets that can exist in the project. |
| configmaps | The total number of ConfigMap objects that can exist in the project. |
| persistentvolumeclaims | The total number of persistent volume claims that can exist in the project. |
| openshift<span>.io</span>/imagestreams | The total number of imagestreams that can exist in the project. |

## Quota scopes
Each quota can have an associated set of scopes. A quota only measures usage for a resource if it matches the intersection of enumerated scopes.

Adding a scope to a quota restricts the set of resources to which that quota can apply. Specifying a resource outside of the allowed set results in a validation error.

| Scope | Description |
| :--- | :--- |
| Terminating | Match pods where spec.activeDeadlineSeconds >= 0. |
| NotTerminating | Match pods where spec.activeDeadlineSeconds is nil. |
| BestEffort | Match pods that have best effort quality of service for either cpu or memory. |
| NotBestEffort | Match pods that do not have best effort quality of service for cpu and memory. |



A `BestEffort` scope restricts a quota to limiting the following resources:
- pods

A `Terminating`, `NotTerminating`, and `NotBestEffort` scope restricts a quota to tracking the following resources:
- pods
- memory
- requests.memory
- limits.memory
- cpu
- requests.cpu
- limits.cpu
- ephemeral-storage
- requests.ephemeral-storage
- limits.ephemeral-storage

## Quota enforcement
After a resource quota for a project is first created, the project restricts the ability to create any new resources that may violate a quota constraint until it has calculated updated usage statistics.

After a quota is created and usage statistics are updated, the project accepts the creation of new content. When you create or modify resources, your quota usage is **incremented immediately** upon the request to create or modify the resource.

When you delete a resource, your quota use is **decremented during the next full recalculation of quota statistics** for the project. A configurable amount of time determines how long it takes to reduce quota usage statistics to their current observed system value.

If project modifications exceed a quota usage limit, the server denies the action, and an appropriate error message is returned to the user explaining the quota constraint violated, and what their currently observed usage statistics are in the system.

## Requests versus limits

When allocating compute resources, each container might specify a request and a limit value each for CPU, memory, and ephemeral storage. Quotas can restrict any of these values.

If the quota has a value specified for requests.cpu or requests.memory, then it requires that every incoming container make an explicit request for those resources. If the quota has a value specified for limits.cpu or limits.memory, then it requires that every incoming container specify an explicit limit for those resources.


## Creating a quota
You can create a quota to constrain resource usage in a given project.

1. Define the quota in a file.
2. Use the file to create the quota and apply it to a project  
`oc create -f <file> [-n <project_name>]`

## Create object count quotas
You can create an object count quota for all OKD standard namespaced resource types, such as `BuildConfig`, and `DeploymentConfig`. An object quota count places a defined quota on all standard namespaced resource types.

When using a resource quota, an object is charged against the quota if it exists in server storage. These types of quotas are useful to protect against exhaustion of storage resources.

```bash
oc create quota <name> --hard=count/<resource>.<group>=<quota>,count/<resource>.<group>=<quota>
```

Example:

```bash
oc create quota test --hard=count/deployments.extensions=2,count/replicasets.extensions=4,count/pods=3,count/secrets=4
```

This example limits the listed resources to the hard limit in each project in the cluster.

### Check quota details <!-- omit in toc -->
```bash
oc describe quota test
```

## Viewing a quota
### View list of quotas <!-- omit in toc -->
```bash
oc get quota -n <project>
```
		
### Describe quota of interest in a project <!-- omit in toc -->
```bash
oc describe quota <quota_name> -n <project>
```

https://docs.okd.io/latest/applications/quotas/quotas-setting-per-project.html



# Clusterquotas
https://docs.okd.io/latest/applications/quotas/quotas-setting-across-multiple-projects.html


# Control Pod Placement Across Cluster Nodes (scheduling)
The default OKD pod scheduler is responsible for determining placement of new pods onto nodes within the cluster. It reads data from the pod and tries to find a node that is a good fit based on configured policies. It is completely independent and exists as a standalone/pluggable solution. It does not modify the pod and just creates a binding for the pod that ties the pod to the particular node.

## Creating a scheduler policy file <!-- omit in toc -->
You can control change the default scheduling behavior by creating a JSON file with using the with the desired predicates and priorities. You then generate a ConfigMap from the JSON file and point the cluster Scheduler object to use the ConfigMap.

1. Create a JSON file named [policy.cfg](files/sample-scheduler-policy.cfg)
2. Create a ConfigMap based on the scheduler JSON file:
```bash
oc create configmap -n openshift-config --from-file=policy.cfg <configmap-name> 
```
3. Add ConfigMap to the Scheduler Operator Custom Resource:
```bash
oc patch Scheduler cluster --type='merge' -p '{"spec":{"policy":{"name":<configmap-name>}}}'
```
> After making the change to the Scheduler config resource, wait for the opensift-kube-apiserver pods to redeploy. This can take several minutes. Until the pods redeploy, new scheduler does not take effect.

4. Verify the scheduler policy is configured by viewing the log of a scheduler pod in the `openshift-kube-scheduler` namespace. The following command checks for the predicates and priorities that are being registered by the scheduler:
```bash
oc logs <scheduler-pod> | grep predicates
```
Example output:
```shell

Creating scheduler with fit predicates 'map[MaxGCEPDVolumeCount:{} MaxAzureDiskVolumeCount:{} CheckNodeUnschedulable:{} NoDiskConflict:{} NoVolumeZoneConflict:{} GeneralPredicates:{} MaxCSIVolumeCountPred:{} CheckVolumeBinding:{} MaxEBSVolumeCount:{} MatchInterPodAffinity:{} PodToleratesNodeTaints:{}]' and priority functions 'map[InterPodAffinityPriority:{} LeastRequestedPriority:{} ServiceSpreadingPriority:{} ImageLocalityPriority:{} SelectorSpreadPriority:{} EqualPriority:{} BalancedResourceAllocation:{} NodePreferAvoidPodsPriority:{} NodeAffinityPriority:{} TaintTolerationPriority:{}]'
```

https://docs.okd.io/latest/nodes/scheduling/nodes-scheduler-default.html