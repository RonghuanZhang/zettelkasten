---
type: source-note
title: GKE Ingress for Application Load Balancers  |  GKE networking
id: 20250611100615
created: 2025-06-11T10:12:15
source:
  - web
url: https://cloud.google.com/kubernetes-engine/docs/concepts/ingress
tags:
  - source-note
  - gcp/gke
  - kubernetes/ingress
processed: false
archived: false
---
---

This page provides a general overview of Ingress for external Application Load Balancers and how it works. Google Kubernetes Engine (GKE) provides a built-in and managed Ingress controller called GKE Ingress. When you create an Ingress resource in GKE, the controller automatically configures an HTTPS load balancer that allows HTTP or HTTPS traffic to reach your Services.

This page is for Networking specialists who design and architect the network for their organization and install, configure, and support network equipment. To learn more about common roles and example tasks that we reference in Google Cloud content, see [Common GKE Enterprise user roles and tasks](https://cloud.google.com/kubernetes-engine/enterprise/docs/concepts/roles-tasks).

Before reading this page, ensure that you're familiar with [Kubernetes](https://kubernetes.io/).

## Overview

In GKE, an [Ingress object](https://kubernetes.io/docs/concepts/services-networking/ingress/) defines rules for routing HTTP(S) traffic to applications running in a cluster. An Ingress object is associated with one or more [Service objects](https://cloud.google.com/kubernetes-engine/docs/concepts/service), each of which is associated with a set of Pods. To learn more about how Ingress exposes applications using Services, see [Service networking overview](https://cloud.google.com/kubernetes-engine/docs/concepts/service-networking).

When you create an Ingress object, the [GKE Ingress controller](https://github.com/kubernetes/ingress-gce) creates a [Google Cloud HTTP(S) Load Balancer](https://cloud.google.com/load-balancing/docs/https) and configures it according to the information in the Ingress and its associated Services.

To use Ingress, you must have the HTTP load balancing add-on enabled. GKE clusters have HTTP load balancing enabled by default; you must not disable it.

## Ingress for external and internal traffic

GKE Ingress resources come in two types:

- [Ingress for external Application Load Balancers](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress-xlb) deploys the [classic Application Load Balancer](https://cloud.google.com/load-balancing/docs/https). This internet-facing load balancer is deployed globally across Google's edge network as a managed and scalable pool of load balancing resources. Learn how to [set up and use Ingress for external Application Load Balancers](https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress).
- [Ingress for internal Application Load Balancers](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress-ilb) deploys the [internal Application Load Balancer](https://cloud.google.com/load-balancing/docs/l7-internal). These internal Application Load Balancers are powered by Envoy proxy systems outside of your GKE cluster, but within your VPC network. Learn how to [set up and use Ingress for internal Application Load Balancers](https://cloud.google.com/kubernetes-engine/docs/how-to/internal-load-balance-ingress).

## GKE Ingress controller behavior

For clusters running GKE versions 1.18 and later, whether or not the GKE Ingress controller processes an Ingress depends on the value of the `kubernetes.io/ingress.class` annotation:

| `kubernetes.io/ingress.class` value | `ingressClassName` value | GKE Ingress controller behavior |
| --- | --- | --- |
| Not set | Not set | Process the Ingress manifest and create an external Application Load Balancer. |
| Not set | Any value | Takes no action. The Ingress manifest could be processed by a third-party Ingress controller if one has been deployed. |
| `gce` | Any value. This field is ignored. | Process the Ingress manifest and create an external Application Load Balancer. |
| `gce-internal` | Any value. This field is ignored. | Process the Ingress manifest and create an internal Application Load Balancer. |
| Set to a value other than `gce` or `gce-internal` | Any value | Takes no action. The Ingress manifest could be processed by a third-party Ingress controller if one has been deployed. |

For clusters running older GKE versions, the GKE controller processes any Ingress that does not have the annotation `kubernetes.io/ingress.class`, or has the annotation with the value `gce` or `gce-internal`.

### kubernetes.io/ingress.class annotation deprecation

Although the `kubernetes.io/ingress.class` annotation is [deprecated in Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress/#deprecated-annotation), GKE continues to use this annotation.

You cannot use the `ingressClassName` field to specify a GKE Ingress. You must use the `kubernetes.io/ingress.class` annotation.

## Features of external Application Load Balancers

An external Application Load Balancer, configured by Ingress, includes the following features:

- Flexible configuration for Services. An Ingress defines how traffic reaches your Services and how the traffic is routed to your application. In addition, an Ingress can provide a single IP address for multiple Services in your cluster.
- Integration with Google Cloud network services
- Support for multiple TLS certificates. An Ingress can specify the use of multiple TLS certificates for request termination.

For a comprehensive list, see [Ingress configuration](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-configuration).

## Load balancing methods

GKE supports Container-native load balancing and instance groups.

### Container-native load balancing

[Container-native load balancing](https://cloud.google.com/kubernetes-engine/docs/how-to/container-native-load-balancing) is the practice of load balancing directly to Pod endpoints in GKE. Container-native load balancing is a type of [Network Endpoint Groups (NEGs)](https://cloud.google.com/load-balancing/docs/negs).

With NEGs, traffic is load balanced from the load balancer directly to the Pod IP as opposed to traversing the VM IP and kube-proxy networking. In addition, [Pod readiness gates](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate) are implemented to determine the health of Pods from the perspective of the load balancer and not just the Kubernetes in-cluster health probes. This improves overall traffic stability by making the load balancer infrastructure aware of lifecycle events such as Pod startup, Pod loss, or VM loss. These capabilities resolve the preceding limitations and result in more performant and stable networking.

Container-native load balancing is enabled by default for Services when all of the following conditions are true:

- For Services created in GKE clusters 1.17.6-gke.7 and up
- Using VPC-native clusters
- Not using a Shared VPC network
- Not using GKE Network Policy

In these conditions, Services will be annotated automatically with `cloud.google.com/neg: '{"ingress": true}'` indicating that a NEG should be created to mirror the Pod IPs within the Service. The NEG is what allows Compute Engine load balancers to communicate directly with Pods. Note that existing Services created prior to GKE 1.17.6-gke.7 won't be automatically annotated by the Service controller.

For GKE 1.17.6-gke.7 and earlier, where NEG annotation is automatic, it is possible to disable NEGs and force the Compute Engine external load balancer to use an instance group as its backends if necessary. This can be done by explicitly annotating Services with `cloud.google.com/neg: '{"ingress": false}'`. It is not possible to disable NEGs with Ingress for internal Application Load Balancers.

For clusters where NEGs are not the default, it is still **strongly recommended** to use container-native load balancing, but it must be enabled explicitly on a per-Service basis. The annotation should be applied to Services in the following manner:

```
kind: Service
...
  annotations:
    cloud.google.com/neg: '{"ingress": true}'
...
```

### Instance groups

When using [instance groups](https://cloud.google.com/compute/docs/instance-groups), Compute Engine load balancers send traffic to VM IP addresses as backends. When running containers on VMs, in which containers share the same host interface, this introduces the following limitations:

- It incurs two hops of load balancing - one hop from the load balancer to the VM `NodePort` and another hop through kube-proxy routing to the Pod IP addresses (which may reside on a different VM).
- Additional hops add latency and make the traffic path more complex.
- The Compute Engine load balancer has no direct visibility to Pods resulting in suboptimal traffic balancing.
- Environmental events like VM or Pod loss are more likely to cause intermittent traffic loss due to the double traffic hop.

#### External Ingress and routes-based clusters

If you use routes-based clusters with external Ingress, the GKE Ingress controller cannot use container-native load balancing using `GCE_VM_IP_PORT` network endpoint groups (NEGs). Instead, the Ingress controller uses unmanaged instance group backends that include all nodes in all node pools. If these unmanaged instance groups are also used by `LoadBalancer` Services, it can cause issues related to the [Single load-balanced instance group limitation](https://cloud.google.com/kubernetes-engine/docs/concepts/service-load-balancer#limit-lb-ig).

Some older external Ingress objects created in VPC-native clusters might use instance group backends on the backend services of each external Application Load Balancer they create. This is not relevant to internal Ingress because internal Ingress resources always use `GCE_VM_IP_PORT` NEGs and require VPC-native clusters.

To learn how to troubleshoot 502 errors with external Ingress, see [External Ingress produces HTTP 502 errors](https://cloud.google.com/kubernetes-engine/docs/troubleshooting/load-balancing#ingress-502s).

## Shared VPC

Ingress and MultiClusterIngress resources are supported in [Shared VPC](https://cloud.google.com/vpc/docs/shared-vpc), but they require additional preparation.

The Ingress controller runs on the GKE control plane and makes API calls to Google Cloud using the GKE service account of the cluster's project. By default, when a cluster that is located in a Shared VPC service project uses a Shared VPC network, the Ingress controller cannot use the service project's GKE service account to create and update ingress allow firewall rules in the host project.

You can grant the service project's GKE service account permissions to create and [manage VPC firewall rules](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-shared-vpc#managing_firewall_resources) in the host project. By granting these permissions, GKE creates ingress allow firewall rules for the following:

- Google Front End (GFE) proxies and health check systems used by external Application Load Balancers for external Ingress. For more information, see the [External Application Load Balancer overview](https://cloud.google.com/load-balancing/docs/https#firewall-rules).
- Health check systems for internal Application Load Balancers used by internal Ingress.

### Manually provision firewall rules from the host project

If your security policies only allow firewall management from the host project, then you can provision these firewall rules manually. When deploying Ingress in a Shared VPC, the Ingress resource event provides the specific firewall rule you need to add necessary to provide access.

To manually provision a rule:

1. View the Ingress resource event:
	```
	kubectl describe ingress INGRESS_NAME
	```
	Replace INGRESS\_NAME with the name of your Ingress.
	You should see output similar to the following example:
	```
	Events:
	Type    Reason  Age                    From                     Message
	----    ------  ----                   ----                     -------
	Normal  Sync    9m34s (x237 over 38h)  loadbalancer-controller  Firewall change required by security admin: \`gcloud compute firewall-rules update k8s-fw-l7--6048d433d4280f11 --description "GCE L7 firewall rule" --allow tcp:30000-32767,tcp:8080 --source-ranges 130.211.0.0/22,35.191.0.0/16 --target-tags gke-l7-ilb-test-b3a7e0e5-node --project <project>\`
	```
	The suggested required firewall rule appears in the `Message` column.
2. Copy and apply the suggested firewall rules from the host project. Applying the rule provides access to your Pods from the load balancer and Google Cloud health checkers.

### Providing the GKE Ingress controller permission to manage host project firewall rules

If you want a GKE cluster in a service project to create and manage the firewall resources in your host project, the service project's GKE service account must be granted the appropriate IAM permissions using one of the following strategies:

- Grant the service project's GKE service account the [Compute Security Admin role](https://cloud.google.com/compute/docs/access/iam#compute.securityAdmin) to the host project. The following example demonstrates this strategy.
- For a finer grained approach, [create a custom IAM role](https://cloud.google.com/iam/docs/creating-custom-roles#creating_a_custom_role) that includes only the following permissions: `compute.networks.updatePolicy`,`compute.firewalls.list`, `compute.firewalls.get`, `compute.firewalls.create`,`compute.firewalls.update`, `compute.firewalls.delete`, and `compute.subnetworks.list`. Grant the service project's GKE service account that custom role to the host project.

If you have clusters in more than one service project, you must choose one of the strategies and repeat it for each service project's GKE service account.

```
gcloud projects add-iam-policy-binding HOST_PROJECT_ID \
  --member=serviceAccount:service-SERVICE_PROJECT_NUMBER@container-engine-robot.iam.gserviceaccount.com \
  --role=roles/compute.securityAdmin
```

Replace the following:

- `HOST_PROJECT_ID`: the [project ID](https://cloud.google.com/resource-manager/docs/creating-managing-projects#identifying_projects) of the [Shared VPC host project](https://cloud.google.com/vpc/docs/shared-vpc#shared_vpc_host_project_and_service_project_associations).
- `SERVICE_PROJECT_NUMBER`: the [project number](https://cloud.google.com/resource-manager/docs/creating-managing-projects#identifying_projects) of the [service project](https://cloud.google.com/vpc/docs/shared-vpc#shared_vpc_host_project_and_service_project_associations) that contains your cluster.

## Multiple backend services

Each external Application Load Balancer or internal Application Load Balancer uses a single URL map, which references one or more backend services. One backend service corresponds to each Service referenced by the Ingress.

For example, you can configure the load balancer to route requests to different backend services depending on the URL path. Requests sent to your-store.example could be routed to a backend service that displays full-price items, and requests sent to your-store.example/discounted could be routed to a backend service that displays discounted items.

You can also configure the load balancer to route requests according to the hostname. Requests sent to your-store.example could go to one backend service, and requests sent to your-experimental-store.example could go to another backend service.

In a GKE cluster, you create and configure an HTTP(S) load balancer by creating a Kubernetes Ingress object. An Ingress object must be associated with one or more Service objects, each of which is associated with a set of Pods.

If you want to configure GKE Ingress with multiple backends for the same host, you must have a single rule with a single host and multiple paths. Otherwise, the GKE ingress controller provisions only one backend service

Here is a manifest for an Ingress called `my-ingress`:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
 rules:
  - host: your-store.example
    http:
      paths:
      - path: /products
        backend:
          service:
            name: my-products
            port:
              number: 60000
      - path: /discounted-products
        backend:
          service:
            name: my-discounted-products
            port:
              number: 80
```

When you create the Ingress, the GKE ingress controller creates and configures an external Application Load Balancer or an internal Application Load Balancer according to the information in the Ingress and the associated Services. Also, the load balancer is given a stable IP address that you can associate with a domain name.

In the preceding example, assume you have associated the load balancer's IP address with the domain name your-store.example. When a client sends a request to your-store.example, the request is routed to a Kubernetes Service named `my-products` on port 60000. And when a client sends a request to your-store.example/discounted, the request is routed to a Kubernetes Service named `my-discounted-products` on port 80.

The only supported wildcard character for the `path` field of an Ingress is the `*` character. The `*` character must follow a forward slash (`/`) and must be the last character in the pattern. For example, `/*`, `/foo/*`, and `/foo/bar/*` are valid patterns, but `*`, `/foo/bar*`, and `/foo/*/bar` are not.

A more specific pattern takes precedence over a less specific pattern. If you have both `/foo/*` and `/foo/bar/*`, then `/foo/bar/bat` is taken to match `/foo/bar/*`.

For more information about path limitations and pattern matching, see the [URL Maps documentation](https://cloud.google.com/load-balancing/docs/url-map).

The manifest for the `my-products` Service might look like this:

```
apiVersion: v1
kind: Service
metadata:
  name: my-products
spec:
  type: NodePort
  selector:
    app: products
    department: sales
  ports:
  - protocol: TCP
    port: 60000
    targetPort: 50000
```

In the Service manifest, you must use `type: NodePort` unless you're using [container native load balancing](https://cloud.google.com/kubernetes-engine/docs/concepts/container-native-load-balancing). If using container native load balancing, use the `type: ClusterIP`.

In the Service manifest, the `selector` field says any Pod that has both the `app: products` label and the `department: sales` label is a member of this Service.

When a request comes to the Service on port 60000, it is routed to one of the member Pods on TCP port 50000.

Each member Pod must have a container listening on TCP port 50000.

The manifest for the `my-discounted-products` Service might look like this:

```
apiVersion: v1
kind: Service
metadata:
  name: my-discounted-products
spec:
  type: NodePort
  selector:
    app: discounted-products
    department: sales
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

In the Service manifest, the `selector` field says any Pod that has both the `app: discounted-products` label and the `department: sales` label is a member of this Service.

When a request comes to the Service on port 80, it is routed to one of the member Pods on TCP port 8080.

Each member Pod must have a container listening on TCP port 8080.

## Default backend

You can specify a default backend for your Ingress by providing a `spec.defaultBackend` field in your Ingress manifest. This will handle any requests that don't match the paths defined in the `rules` field. For example, in the following Ingress, any requests that don't match `/discounted` are sent to a service named `my-products` on port 60001.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  defaultBackend:
    service:
      name: my-products
      port:
        number: 60001
  rules:
  - http:
      paths:
      - path: /discounted
        pathType: ImplementationSpecific
        backend:
          service:
            name: my-discounted-products
            port:
              number: 80
```

If you don't specify a default backend, GKE provides a default backend that returns 404. This is created as a `default-http-backend` NodePort service on the cluster in the `kube-system` namespace.

The 404 HTTP response is similar to the following:

```
response 404 (backend NotFound), service rules for the path non-existent
```

To set up GKE Ingress with a customer default backend, see the [GKE Ingress with custom default backend](https://github.com/GoogleCloudPlatform/gke-networking-recipes/tree/main/ingress/single-cluster/ingress-custom-default-backend).

## Ingress to Compute Engine resource mappings

The GKE Ingress controller deploys and manages Compute Engine load balancer resources based on the Ingress resources that are deployed in the cluster. The mapping of Compute Engine resources depends on the structure of the Ingress resource.

The following manifest describes an Ingress:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - http:
      paths:
      - path: /*
        pathType: ImplementationSpecific
        backend:
          service:
            name: my-products
            port:
              number: 60000
      - path: /discounted
        pathType: ImplementationSpecific
        backend:
          service:
            name: my-discounted-products
            port:
              number: 80
```

This Ingress manifest instructs GKE to create the following Compute Engine resources:

- A forwarding rule and IP address.
- [Compute Engine firewall rules](https://cloud.google.com/kubernetes-engine/docs/concepts/firewall-rules) that permit traffic for load balancer health checks and application traffic from Google Front Ends or Envoy proxies.
- A target HTTP proxy and a target HTTPS proxy, if you configured TLS.
- A URL map which with a single host rule referencing a single path matcher. The path matcher has two path rules, one for `/*` and another for `/discounted`. Each path rule maps to a unique backend service.
- NEGs which hold a list of Pod IP addresses from each Service as endpoints. These are created as a result of the `my-discounted-products` and `my-products` Services.

## Options for providing SSL certificates

You can provide SSL certificates to an HTTP(S) load balancer using the following methods:

Google-managed certificates

[Google-managed SSL certificates](https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs) are provisioned, deployed, renewed, and managed for your domains. Managed certificates do not support wildcard domains.

Self-managed certificates shared with Google Cloud

You can provision your own SSL certificate and create a certificate resource in your Google Cloud project. You can then list the certificate resource in an annotation on an Ingress to create an HTTP(S) load balancer that uses the certificate. Refer to [instructions for pre-shared certificates](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-multi-ssl#using_pre-shared_certificates) for more information.

Self-managed certificates as Secret resources

You can provision your own SSL certificate and create a [Secret](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/#create-a-secret) to hold it. You can then refer to the Secret in an Ingress specification to create an HTTP(S) load balancer that uses the certificate. Refer to the [instructions for using certificates in Secrets](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-multi-ssl) for more information.

## Health checks

When you expose one or more Services through an Ingress using the default Ingress controller, GKE creates a [classic Application Load Balancer](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress-xlb) or an [internal Application Load Balancer](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress-ilb). Both of these load balancers support multiple [backend services](https://cloud.google.com/load-balancing/docs/backend-service) on a single [URL map](https://cloud.google.com/load-balancing/docs/url-map-concepts). Each of the backend services corresponds to a Kubernetes Service, and each backend service must reference a [Google Cloud health check](https://cloud.google.com/load-balancing/docs/health-check-concepts). This health check is *different* from a Kubernetes liveness or readiness probe because the health check is implemented outside of the cluster.

Load balancer health checks are specified *per backend service*. While it's possible to use the same health check for all backend services of the load balancer, the health check reference isn't specified for the whole load balancer (at the Ingress object itself).

GKE creates health checks based on one of the following methods:

- **`BackendConfig` CRD**: A custom resource definition (CRD) that gives you precise control over how your Services interact with the load balancer.`BackendConfig` CRDs allow you to specify custom settings for the health check associated with the corresponding backend service. These custom settings provide greater flexibility and control over health checks for both the classic Application Load Balancer and internal Application Load Balancer created by an Ingress.
- **Readiness probe**: A diagnostic check that determines if a container within a Pod is ready to serve traffic. The GKE Ingress controller creates the health check for the Service's backend service based on the readiness probe used by that Service's serving Pods. You can derive the health check parameters such as path, port, and protocol from the readiness probe definition.
- **Default values**: The parameters used when you don't configure a `BackendConfig` CRD or define attributes for the readiness probe.

**Best practice**:

Use a `BackendConfig` CRD to have the most control over the load balancer health check settings.

GKE uses the following procedure to create a health check for each backend service corresponding to a Kubernetes Service:

- If the Service references [a `BackendConfig` CRD](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress#direct_hc) with `healthCheck` information, GKE uses that to create the health check. Both the GKE Enterprise Ingress controller and the GKE Ingress controller support creating health checks this way.
- If the Service does *not* reference a `BackendConfig` CRD:
	- GKE can infer some or all of the parameters for a health check if the Serving Pods use a Pod template with a container whose readiness probe has attributes that can be interpreted as health check parameters. See [Parameters from a readiness probe](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress#interpreted_hc) for implementation details and [Default and inferred parameters](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress#def_inf_hc) for a list of attributes that can be used to create health check parameters. Only the GKE Ingress controller supports inferring parameters from a readiness probe.
	- If the Pod template for the Service's serving Pods does **not** have a container with a readiness probe whose attributes can be interpreted as health check parameters, [the default values](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress#def_inf_hc) are used to create the health check. Both the GKE Enterprise Ingress controller and the GKE Ingress controller can create a health check using only the default values.

### Default and inferred parameters

The following parameters are used when you do **not** specify health check parameters for the corresponding Service using [a `BackendConfig` CRD](https://cloud.google.com/kubernetes-engine/docs/concepts/#direct_hc).

| Health check parameter | Default value | Inferable value |
| --- | --- | --- |
| [Protocol](https://cloud.google.com/load-balancing/docs/health-check-concepts#category_and_protocol) | HTTP | if present in the Service annotation `cloud.google.com/app-protocols` |
| [Request path](https://cloud.google.com/load-balancing/docs/health-check-concepts#criteria-protocol-http) | `/` | if present in the serving Pod's `spec`:   `containers[].readinessProbe.httpGet.path` |
| [Request Host header](https://cloud.google.com/load-balancing/docs/health-check-concepts#criteria-protocol-http) | `Host: backend-ip-address` | if present in the serving Pod's `spec`:   `containers[].readinessProbe.httpGet.httpHeaders` |
| [Expected response](https://cloud.google.com/load-balancing/docs/health-check-concepts#criteria-protocol-http) | HTTP 200 (OK) | HTTP 200 (OK)   cannot be changed |
| [Check interval](https://cloud.google.com/load-balancing/docs/health-check-concepts#probes) | - for Container-native load balancing: 15 seconds - for instance groups: 60 seconds | if present in the serving Pod's `spec`: - for Container-native load balancing:   	`containers[].readinessProbe.periodSeconds` - for instance groups:   	`containers[].readinessProbe.periodSeconds + 60 seconds` |
| [Check timeout](https://cloud.google.com/load-balancing/docs/health-check-concepts#probes) | 5 seconds | if present in the serving Pod's `spec`:   `containers[].readinessProbe.timeoutSeconds` |
| [Healthy threshold](https://cloud.google.com/load-balancing/docs/health-check-concepts#health_state) | 1 | 1   cannot be changed |
| [Unhealthy threshold](https://cloud.google.com/load-balancing/docs/health-check-concepts#health_state) | - for Container-native load balancing: 2 - for instance groups: 10 | same as default: - for Container-native load balancing: 2 - for instance groups: 10 |
| [Port specification](https://cloud.google.com/load-balancing/docs/health-check-concepts#category_and_port_specification) | - for Container-native load balancing: the Service's `port` - for instance groups: the Service's `nodePort` | The health check probes are sent to the port number specified by the `spec.containers[].readinessProbe.httpGet.port`, as long as all of the following are also true: - The readiness probe's port number must match the serving Pod's `containers[].spec.ports.containerPort` - The serving Pod's `containerPort` matches the Service's `targetPort` - The Ingress *service backend port specification* references a valid port from `spec.ports[]` of the Service. This can be done in one of two ways: 	- `spec.rules[].http.paths[].backend.service.port.name` in the Ingress matches `spec.ports[].name` defined in the corresponding Service 	- `spec.rules[].http.paths[].backend.service.port.number` in the Ingress matches `spec.ports[].port` defined in the corresponding Service |
| [Destination IP address](https://cloud.google.com/load-balancing/docs/health-check-concepts#hc-packet-dest) | - for Container-native load balancing: the Pod's IP address - for instance groups: the node's IP address | same as default: - for Container-native load balancing: the Pod's IP address - for instance groups: the node's IP address |

### Parameters from a readiness probe

When GKE creates the health check for the Service's backend service, GKE can copy certain parameters from *one container's* readiness probe used by that Service's serving Pods. This option is *only* supported by the GKE Ingress controller.

Supported readiness probe attributes that can be interpreted as health check parameters are listed along with the default values in [Default and inferred parameters](https://cloud.google.com/kubernetes-engine/docs/concepts/#def_inf_hc). Default values are used for any attributes not specified in the readiness probe or if you don't specify a readiness probe at all.

If serving Pods for your Service contain *multiple* containers, or if you're using the GKE Enterprise Ingress controller, you should use a `BackendConfig` CRD to define health check parameters. For more information, see [When to use a `BackendConfig` CRD instead](https://cloud.google.com/kubernetes-engine/docs/concepts/#direct_hc_instead).

#### When to use BackendConfig CRDs instead

Instead of relying on parameters from Pod readiness probes, you should explicitly define health check parameters for a backend service by creating a [`BackendConfig` CRD](https://cloud.google.com/kubernetes-engine/docs/concepts/#direct_hc) for the Service in these situations:

- **If you're using GKE Enterprise:** The GKE Enterprise Ingress controller does **not** support obtaining health check parameters from the readiness probes of serving Pods. It can only create health checks using [implicit parameters](https://cloud.google.com/kubernetes-engine/docs/concepts/#def_inf_hc) or as defined in a [`BackendConfig` CRD](https://cloud.google.com/kubernetes-engine/docs/concepts/#direct_hc).
- **If you have more than one container in the serving Pods:**GKE does not have a way to select the readiness probe of a *specific container* from which to infer health check parameters. Because each container can have its own readiness probe, and because a readiness probe isn't a required parameter for a container, you should define the health check for the corresponding backend service by referencing a [`BackendConfig` CRD](https://cloud.google.com/kubernetes-engine/docs/concepts/#direct_hc) on the corresponding Service.
- **If you need control over the port used for the load balancer's health checks:** GKE only uses the readiness probe's `containers[].readinessProbe.httpGet.port` for the backend service's health check when that port matches the service port for the Service referenced in the Ingress `spec.rules[].http.paths[].backend.servicePort`.

### Parameters from a BackendConfig CRD

You can specify the backend service's health check parameters [using the `healthCheck` parameter of a `BackendConfig` CRD](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-configuration#direct_health) referenced by the corresponding Service. This gives you more flexibility and control over health checks for a classic Application Load Balancer or an internal Application Load Balancer created by an Ingress. See [Ingress configuration](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-configuration) for GKE version compatibility.

This example `BackendConfig` CRD defines the health check protocol (type), a request path, a port, and a check interval in its `spec.healthCheck` attribute:

```
apiVersion: cloud.google.com/v1
kind: BackendConfig
metadata:
  name: http-hc-config
spec:
  healthCheck:
    checkIntervalSec: 15
    port: 15020
    type: HTTPS
    requestPath: /healthz
```

To configure all the fields available when configuring a `BackendConfig` health check, use the [custom health check configuration](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-configuration#direct_health) example.

To set up GKE Ingress with a custom HTTP health check, see [GKE Ingress with custom HTTP health check](https://github.com/GoogleCloudPlatform/gke-networking-recipes/tree/main/ingress/single-cluster/ingress-custom-http-health-check).

## Using multiple TLS certificates

Suppose you want an HTTP(S) load balancer to serve content from two hostnames: your-store.example and your-experimental-store.example. Also, you want the load balancer to use one certificate for your-store.example and a different certificate for your-experimental-store.example.

You can do this by specifying multiple certificates in an Ingress manifest. The load balancer chooses a certificate if the Common Name (CN) in the certificate matches the hostname used in the request. For detailed information on how to configure multiple certificates, see [Using multiple SSL certificates in HTTPS Load Balancing with Ingress](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-multi-ssl).

## Kubernetes Service compared to Google Cloud backend service

A Kubernetes [Service](https://kubernetes.io/docs/concepts/services-networking/service/) and a [Google Cloud backend service](https://cloud.google.com/load-balancing/docs/backend-service) are different things. There is a strong relationship between the two, but the relationship is not necessarily one to one. The GKE ingress controller creates a Google Cloud backend service for each (`service.name`, `service.port`) pair in an Ingress manifest. So it is possible for one Kubernetes Service object to be related to several Google Cloud backend services.

## Limitations

- In clusters using versions earlier than 1.16, the total length of the namespace and name of an Ingress must not exceed 40 characters. Failure to follow this guideline may cause the GKE Ingress controller to act abnormally. For more information, see this [GitHub issue about long names](https://github.com/kubernetes/ingress-gce/issues/537).
- In clusters using NEGs, ingress reconciliation time may be affected by the number of ingresses. For example, a cluster with 20 ingresses, each containing 20 distinct NEG backends, may result in a latency of more than 30 minutes for an ingress change to be reconciled. This especially impacts regional clusters due to the increased number of NEGs needed.
- [Quotas for URL maps](https://cloud.google.com/load-balancing/docs/quotas#url_maps) apply.
- [Quotas for Compute Engine resources](https://cloud.google.com/compute/docs/resource-quotas) apply.
- If you're not using NEGs with the [GKE ingress controller](https://github.com/kubernetes/ingress-gce) then GKE clusters have a limit of 1000 nodes. When services are deployed with NEGs, there is no GKE node limit. Any non-NEG Services exposed through Ingress do not function correctly on clusters above 1000 nodes.
- For the GKE Ingress controller to use your `readinessProbes` as health checks, the Pods for an Ingress must exist at the time of Ingress creation. If your replicas are scaled to 0, the default health check applies. For more information, see this [GitHub issue comment about health checks](https://github.com/kubernetes/ingress-gce/issues/241#issuecomment-384749607).
- Changes to a Pod's `readinessProbe` do not affect the Ingress after it is created.
- An external Application Load Balancer terminates TLS in locations that are distributed globally, to minimize latency between clients and the load balancer. If you require geographic control over where TLS is terminated, you should use a [custom ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) exposed through a GKE Service of type `LoadBalancer` instead, and terminate TLS on backends that are located in regions appropriate to your needs.
- Combining multiple Ingress resources into a single Google Cloud load balancer is not supported.
- You must turn off mutual TLS on your application because it is [not supported for external Application Load Balancers](https://cloud.google.com/load-balancing/docs/https#limitations).
- Ingress can only expose HTTP ports `80` and `443` for its frontend.

## Implementation details

- The Ingress controller performs periodic checks of service account permissions by fetching a test resource from your Google Cloud project. You will see this as a `GET` of the (non-existent) global `BackendService` with the name `k8s-ingress-svc-acct-permission-check-probe`. As this resource should not normally exist, the `GET` request will return "not found". This is expected; the controller is checking that the API call is not rejected due to authorization issues. If you create a BackendService with the same name, then the `GET` will succeed instead of returning "not found".

## Templates for the Ingress configuration

- In [GKE Networking Recipes](https://github.com/GoogleCloudPlatform/gke-networking-recipes/tree/main), you can find templates provided by GKE on many common Ingress usage under the [Ingress](https://github.com/GoogleCloudPlatform/gke-networking-recipes/tree/main/ingress/single-cluster) section.

## What's next

- [Learn about GKE Networking Recipes](https://github.com/GoogleCloudPlatform/gke-networking-recipes/tree/main)
- [Learn more about load balancing in Google Cloud](https://cloud.google.com/load-balancing/docs/https).
- [Read an overview of networking in GKE](https://cloud.google.com/kubernetes-engine/docs/concepts/network-overview).
- [Learn how to configure Ingress for internal Application Load Balancers](https://cloud.google.com/kubernetes-engine/docs/how-to/internal-load-balance-ingress).
- [Learn how to configure Ingress for external Application Load Balancers](https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress).

Except as otherwise noted, the content of this page is licensed under the [Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/), and code samples are licensed under the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0). For details, see the [Google Developers Site Policies](https://developers.google.com/site-policies). Java is a registered trademark of Oracle and/or its affiliates.

Last updated 2025-06-05 UTC.

### Welcome to Cloud Shell

Cloud Shell is a development environment that you can use in the browser:

- Activate Cloud Shell to explore Google Cloud with a terminal and an editor
- Start a free trial to get $300 in free credits

![](https://www.gstatic.com/devrel-devsite/prod/vd980a342b8e3e77c07209be506f8385246f583d6eec83ceb07569bbf26f054dc/images/cloud-shell-cta-art.png)