# Node Scheduling <!-- omit in toc -->

Table of Contents
- [Manually scaling a MachineSet](#manually-scaling-a-machineset)
	- [Deleting a specific machine](#deleting-a-specific-machine)
- [Applying autoscaling to an OKD cluster](#applying-autoscaling-to-an-okd-cluster)
	- [ClusterAutoscaler](#clusterautoscaler)
	- [MachineAutoscaler](#machineautoscaler)

# Manually scaling a MachineSet
#### Machine <!-- omit in toc -->
it desribes the host of a node

#### Machineset <!-- omit in toc -->
group of machines

> You need to `cluster-admin` for modifying machinesets.

### View the MachineSets that are in the cluster: <!-- omit in toc -->
```bash
oc get machinesets -n openshift-machine-api
```

### Scale the MachineSet <!-- omit in toc -->
```bash
oc scale --replicas=2 machineset <machineset> -n openshift-machine-api
```

or

```bash
oc edit machineset <machineset> -n openshift-machine-api
```
			
You can scale the MachineSet up or down. It takes several minutes for the new machines to be available.

https://docs.okd.io/latest/machine_management/manually-scaling-machineset.html

## Deleting a specific machine
### View the Machines that are in the cluster and identify the one to delete: <!-- omit in toc -->
```bash
oc get machine -n openshift-machine-api
```
> The command output contains a list of Machines in the \<clusterid>-worker-<cloud_region> format.

### Delete the Machine: <!-- omit in toc -->
```bash
oc delete machine <machine> -n openshift-machine-api
```
> By default, the machine controller tries to drain the node that is backed by the machine until it succeeds. In some situations, such as with a misconfigured Pod disruption budget, the drain operation might not be able to succeed in preventing the machine from being deleted. You can skip draining the node by annotating "machine.openshift<snap>.io</snap>/exclude-node-draining" in a specific machine. If the machine being deleted belongs to a MachineSet, a new machine is immediately created to satisfy the specified number of replicas.


https://docs.okd.io/latest/machine_management/deleting-machine.html

# Applying autoscaling to an OKD cluster

Applying autoscaling to an OKD cluster involves deploying a ClusterAutoscaler and then deploying MachineAutoscalers for each Machine type in your cluster.

#### ClusterAutoscaler  <!-- omit in toc -->
adjusts the size of cluster to meet its current deployment needs
- has a cluster scope, and is not associated with a particular namespace.
- makes more machines when the cluster running out of resources

	
#### MachineAutoscaler  <!-- omit in toc -->
adjusts the number of Machines in the MachineSets
- makes more Machines when the cluster runs out of resources
- any changes to the autoscaler resoures (eg.: min, max) applies immediatelly to the machineset 


> You must deploy a MachineAutoscaler for the ClusterAutoscaler to scale your machines. The ClusterAutoscaler uses the annotations on MachineSets that the MachineAutoscaler sets to determine the resources that it can scale. If you define a ClusterAutoscaler without also defining MachineAutoscalers, the ClusterAutoscaler will never scale your cluster.



## ClusterAutoscaler
Because the ClusterAutoscaler is scoped to the entire cluster, you can make only one ClusterAutoscaler for the cluster.

### Create ClusterAutoscaler resource file <!-- omit in toc -->
You need to create a [ClusterAutoscaler.yaml](files/ClusterAutoscaler.yaml) file, which shows the parameters and sample values for the ClusterAutoscaler. 

### Deploy ClusterAutoscaler <!-- omit in toc -->
```bash
oc create -f ClusterAutoscaler.yaml
```

> After you configure the ClusterAutoscaler, you must configure at least one MachineAutoscaler.


## MachineAutoscaler

### Create MachineAutoscaler resource file <!-- omit in toc -->
The [MachineAutoscaler.yaml](files/MachineAutoscaler.yaml) file shows the parameters and sample values for the MachineAutoscaler.


### Deploy MachineAutoscaler <!-- omit in toc -->
```bash
oc create -f MachineAutoscaler.yaml
```

### View MachineAutoscaler <!-- omit in toc -->
```bash
oc get machineautoscalers -n openshift-machine-api
```

### Edit MachineAutoscaler <!-- omit in toc -->
```bash
oc edit machineautoscalers <autosclaer_name> -n openshift-machine-api
```

### Check details of MachineAutoscaler <!-- omit in toc -->
```bash
oc describe machineautoscalers <autosclaer_name> -n openshift-machine-api
```

https://docs.okd.io/latest/machine_management/applying-autoscaling.html