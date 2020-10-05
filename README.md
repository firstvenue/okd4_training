# OKD 4.5 training <!-- omit in toc -->

This Training material was designed to mostly cover [EX280 - Red Hat Certified Specialist in OpenShift Administration exam](https://www.redhat.com/en/services/training/ex280-red-hat-certified-specialist-in-openshift-administration-exam?section=Objectives) objectives.


<span style="font-size:2em;">👉 [Training Material](https://github.com/tothti/okd4_training/wiki)  
👉 [Official OKD documentation](https://docs.okd.io/latest/welcome/index.html)</span>


Table of Contents
- [Topics](#topics)
- [What resources you'll need for this training](#what-resources-youll-need-for-this-training)


### Topics
- Basics
  - Components
  - Architecture
- Installation
  - Installation Options
  - Deploy with CodeReady Containers
  - Deploy with openshift-installer to AWS
- Manage Openshift
  - Connect to our cluster
  - CLI-openshift client
  - WebUI
  - Basic Project Management
  - Openshift logs
  - Monitoring
  - Alerting
- Manage Users and Groups
  - Configure Identity Provider
  - Add/Remove Users
  - Add/Remove Groups
  - Manage User/Group roles
- Role Based Access Control
  - Create Custom Roles
  - Role Bidings
  - Service Accounts and Security Context Constraints
- Networking
  - Cluster Network Operator
  - Debug network related issues
  - Manage External routes
  - Ingress operator
  - Using SSL and Edge termination
- Pod scheduling
  - Limit Resource usage
  - Application Scaling
  - Control Pod placement using Scheduler Policy
- Node scheduling
  - Scaling worker count Manually
  - Scaling worker count Automatically


### What resources you'll need for this training
- Red Hat account  
  &nbsp;&nbsp;&nbsp;&nbsp;You can create one at https://access.redhat.com
- AWS Account (to deploy the cluster in AWS)
- A linux machine or a mac (to run cluster installer)
- (Optional) A machine (which meets [hardware requirements](02-Installation.md#dev-environment-with-codeready-containers) for CodeReady Containers)

#### AWS Cost Estimate <!-- omit in toc -->
#### Cluster with minimal resources <!-- omit in toc -->
| Machine Type | AWS EC2 Type| estimated cost |
| --- | --- | --- |
| 1 master node | t4g.xlarge | ~120 USD/month |
| 1 worker node	| t4g.xlarge | ~120 USD/month |
| | |
| **SUM:** | | **~240 USD/month** |
> For a 1 week training period -> ~60-80 USD/week

#### Cluster with complete / recomended resources <!-- omit in toc -->
| Machine Type | AWS EC2 Type| estimated cost |
| --- | --- | --- |
| 3 master node | t4g.xlarge | 3x ~120 USD/month |
| 3 worker node	| t4g.xlarge | 3x ~120 USD/month |
| | |
| **SUM:** | | **~720 USD/month** |
> For a 1 week training period -> ~180-220 USD/week

Those prices are slightly overestimated, since the cluster needs a bootstrap server (it's deleted after installation is complete), loadbalancers,  EBS disk, S3 bucket, etc. and those also will cost some amount.

The default configuration for OKD installation is to deploy 3 master and 3 worker nodes, but cluster functionality, application deployment, monitoring, etc. can be used with only 1 master and 1 worker. Throughout this training the method to deploy only the required minimum resources are explained and can be used.