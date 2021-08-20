# Overview

In some situations, it may be desired to retain the simplicity of an IPI install while allowing custom infrastructure to be used such as:

- API/Ingress load balancing
- DNS


# Getting Started

## 1. Deploy Infrastructure

The following infrastructure must be deploy prior to removing the IPI infrastructure components:

- Layer 4 Load balancer(s) deployed for `api`/`api-int` targeting the control plane nodes for ports 6443 and 22623
NOTE: If `api-int` is deployed as a separate load balancer from `api`, only that load balancer should include port 22623.
- Layer 4 Load balancer deployed for `*.apps` targeting nodes which could host router pods
- Upstream DNS server contains `api-int` A record which targets the `api-int` load balancer
- Upstream DNS server contains `api` A record which targets the `api` load balancer
- Upstream DNS server contains `*.apps` A record which targets the `*.apps` load balancer

## 2. Testing the Infrastructure

To reduce the liklyhood of downtime, it is critical that the deployed infrastructure be tested to ensure:

Are the proper A records defined?  Run the following `dig`s and check the output.

~~~
dig api.<cluster-fqdn>
dig api-int.<cluster-fqdn>
dig console-openshift-console.apps.<cluster-fqdn>
~~~

- Is the load balancer targeting control plane and nodes which could host router pods appropriately?

Perform the following curls and check the certficate subject:
~~~
curl -kv https://api.<cluster-fqdn>:6443
curl -kv https://api-int.<cluster-fqdn>:6443
curl -kv https://api-int.<cluster-fqdn>:22623
curl -kv https://console-openshift-console-apps.<cluster-fqdn>:443
# This will not produce a cert but should return an HTTP 503 response
curl -kv http://console-openshift-console-apps.<cluster-fqdn>
~~~

## 3. Migrating to Custom Infrastructure

~~~
curl https://raw.githubusercontent.com/rvanderp3/onprem-user-infra/main/remove-ipi-infra.yaml | oc create -f -

# Wait for all operators to become available
watch oc get co
~~~

## 4. Restoring IPI Provisioned Infrastructure

~~~
oc delete machineconfig disable-ipi-infra-cp disable-ipi-infra-compute 

# Wait for all operators to become available
watch oc get co
~~~
