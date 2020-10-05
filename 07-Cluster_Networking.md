# Cluster Networking <!-- omit in toc -->

Table of Contents

---
- [Cluster Network Operator](#cluster-network-operator)
	- [Troubleshoot software defined networking](#troubleshoot-software-defined-networking)
- [DNS Operator](#dns-operator)
- [External routes (unsecured)](#external-routes-unsecured)
	- [Edit routes](#edit-routes)
- [Network Ingress](#network-ingress)
- [Secure Routes Using TLS Certificates](#secure-routes-using-tls-certificates)


# Cluster Network Operator

The Cluster Network Operator implements the `network` API from the `operator.openshift.io` API group. The Operator deploys the OpenShift SDN default Container Network Interface (CNI) network provider plug-in, or the default network provider plug-in that you selected during cluster installation, by using a DaemonSet.

- operates the cluster network
- deploys and manages the network components of your cluster (including the CNI [container network interface] and also the Pod Network Provider Plugin [selected for the cluster during installation])
- implements a network API from `operator.openshift.io` API group
- deployed during installation
- deployed as a kubernetes deployment

## Troubleshoot software defined networking
Since CNO is responsible for our networking, we should be able to check it's deployment state, it's status, it's logs in order to troubleshoot issues when the occur.

### View cluster network deployment status <!-- omit in toc -->
```bash
oc get -n openshift-network-operator deployment/network-operator
```

### Viewing Cluster Network Operator status <!-- omit in toc -->
```bash
oc get clusteroperator/network

oc describe clusteroperators/network
```

### Viewing the cluster network configuration <!-- omit in toc -->
Every new OKD installation has a `network.config` object named `cluster`.

```bash
oc describe network.config/cluster
```

### Viewing Cluster Network Operator logs <!-- omit in toc -->
```bash
oc logs --namespace=openshift-network-operator deployment/network-operator
```

https://docs.okd.io/latest/networking/cluster-network-operator.html



### Check endpoints <!-- omit in toc -->
```bash
oc get endpoints -n <project>
```

### Track down pod IPs <!-- omit in toc -->
```bash
oc get pods -n <project> --template='{{range.item}}HostIP:{{status.hostIP}} PodIP: {{.status.podIP}}{{"\n"}}{{end}}'
```

### Check routes <!-- omit in toc -->
```bash
oc get route -n <project>
```

### Check services <!-- omit in toc -->
```bash
oc get services -n <project_name>
```



# DNS Operator
The DNS Operator deploys and manages CoreDNS to provide a name resolution service to pods, enabling DNS-based Kubernetes Service discovery in OpenShift.

The DNS Operator implements the `dns` API from the `operator.openshift.io` API group. The operator deploys CoreDNS using a DaemonSet, creates a Service for the DaemonSet, and configures the kubelet to instruct pods to use the CoreDNS Service IP for name resolution.

It is also deployed during installation as a Kubernetes Deployment.

### Check DNS Operator deployment status <!-- omit in toc -->
```bash
oc get -n openshift-dns-operator deployment/dns-operator
```

### DNS operator status <!-- omit in toc -->
```bash
oc get clusteroperator/dns
```

```bash
oc describe clusteroperator/dns
```

### DNS operator logs <!-- omit in toc -->
```bash
oc logs -n openshift-dns-operator deployment/dns-operator
```

https://docs.okd.io/latest/networking/dns-operator.html


# External routes (unsecured)
To create routes we will expose our services available in our cluster. When exposed a service will be reachable outside from the cluster.


### Create a route <!-- omit in toc -->
```bash
oc expose service <service_name>
```

### Create a route with custom label and route name <!-- omit in toc -->
```bash
oc expose service <service_name> -l name=<label> --name=<route_name>
```

### Create a route with specified port and protocol <!-- omit in toc -->
```bash
oc expose service --port=<port> --protocol="<protocol>"
```

### Create a route with a specified path <!-- omit in toc -->
```bash
oc expose service <service_name> --path=<path>
```


## Edit routes

### Add a timeout to the route <!-- omit in toc -->
```bash
oc annotate route <route> --overwrite haproxy.router.openshift.io/timeout=<timeout>.<time_unit>
```

### Add a desired cookie name <!-- omit in toc -->
```bash
oc annotate route <route> router.openshift.io/<cookie_name>="-<cookie_annotation>"
```

### Restrict access by whitelisting an IP <!-- omit in toc -->
```bash
oc annotate route <route> haproxy.router.openshift.io/ip_whitelist="<ip1 ip2 ip3>"
```

### Enable rate limiting <!-- omit in toc -->
```bash
oc annotate route <route> haproxy.router.openshift.io/rate-limit-connections=true
```




# Network Ingress
Ingress expose HTTP and HTTPS routes from outside cluster to services within the cluster.
Rules are defined in the Ingress resource to controll traffic routing.
The Ingress operator implements the ingress controller API to enable access to the openshift cluster services.
The operator deploys and manages one or more HAProxy based ingress controllers that makes this all possible.

https://docs.okd.io/latest/networking/configuring_ingress_cluster_traffic/overview-traffic.html


### Ingress Resource <!-- omit in toc -->
[example-ingress.yaml](files/example-ingress.yaml)

### Apply Ingress Controller file <!-- omit in toc -->

```bash
oc create -f router-internal.yaml
```


# Secure Routes Using TLS Certificates

### Create a secure route using edge TLS termination <!-- omit in toc -->
```bash
oc create route edge --service=frontend --cert=tls.crt --key=tls.key --hostname=www.example.com
```



Examine route (command above will create something like this)
```bash
apiVersion: v1
kind: Route
metadata:
  name: frontend
spec:
  host: www.example.com
  to:
    kind: Service
	name: frontend
  tls:
    termination: edge
	key: |-
	  -----BEGIN PRIVATE KEY-----
	  [...]
	  -----END PRIVATE KEY-----
	certificate: |-
	  -----BEGIN CERTIFICATE-----
	  [...]
	  -----END CERTIFICATE-----
```
https://docs.okd.io/latest/networking/routes/secured-routes.html

### Create self-signed cert <!-- omit in toc -->
```bash
openssl req -x509 -newkey rsa:4096 -nodes -keyout cert.key -out cert.crt
```

### Remove a passphrase from a key file <!-- omit in toc -->
```bash
openssl rsa -in <password_protected_key_file>.key -out <key_file>.key
```

