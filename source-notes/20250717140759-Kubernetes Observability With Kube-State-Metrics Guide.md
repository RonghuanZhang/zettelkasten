---
type: source-note
title: "Kubernetes Observability With Kube-State-Metrics: Guide"
id: 20250717140759
created: 2025-07-17T14:07:59
source:
  - web
url: https://spacelift.io/blog/kube-state-metrics
tags:
  - source-note
  - prometheus
processed: false
archived: false
---
[Kubernetes](https://spacelift.io/blog/kubernetes)

![Kubernetes Observability With Kube-State-Metrics: Guide](https://spacelift.io/_next/image?url=https%3A%2F%2Fsecure.gravatar.com%2Favatar%2F402ba26575ce7909a6b670410ee12eda9ce420710cc68c3f05adc425d2ba9acb%3Fs%3D48%26d%3Dmm%26r%3Dg&w=96&q=75)

Kubernetes Observability With Kube-State-Metrics: Guide

· 15 min read

Reviewed by:

![kube state metrics](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2025%2F04%2F554.kube-state-metrics.png&w=3840&q=100)

kube state metrics

Kube-State-Metrics is a Kubernetes addon that generates and serves metrics about cluster objects. It allows DevOps teams to detect unhealthy workloads, such as by checking the number of Running Pods or Failed Jobs in the cluster.

A robust observability system is essential to ensure [stable Kubernetes operations](https://spacelift.io/blog/kubernetes-challenges#5-no-monitoringlogging) at scale. Kube-State-Metrics is a key component to include in your Kubernetes monitoring strategy because it lets you make decisions based on the states of objects you’ve created.

In this guide, we’ll explore more about what Kube-State-Metrics is, how it works, and what you can use it for. We’ll also share some simple examples of querying different metrics and discuss best practices for using Kube-State-Metrics in your own cluster.

1. [What is Kube-State-Metrics?](https://spacelift.io/blog/kube-state-metrics#what-is-kubestatemetrics)
2. [How does Kube-State-Metrics work?](https://spacelift.io/blog/kube-state-metrics#how-does-kubestatemetrics-work)
3. [How to deploy Kube-State-Metrics](https://spacelift.io/blog/kube-state-metrics#how-to-install-and-deploy-kubestatemetrics)
4. [Example: Using Kube-State-Metrics with Prometheus](https://spacelift.io/blog/kube-state-metrics#example-using-kubestatemetrics-with-prometheus)
5. [Key use cases and Kubernetes metrics to track](https://spacelift.io/blog/kube-state-metrics#kubestatemetrics-key-use-cases-and-kubernetes-metrics-to-track)
6. [Kube-State-Metrics vs Kubernetes Metrics-Server](https://spacelift.io/blog/kube-state-metrics#kubestatemetrics-vs-kubernetes-metricsserver)
7. [Kube-State-Metrics vs Cluster-level metrics](https://spacelift.io/blog/kube-state-metrics#kubestatemetrics-vs-clusterlevel-metrics)
8. [Kube-State-Metrics best practices](https://spacelift.io/blog/kube-state-metrics#kubestatemetrics-best-practices)

## What is Kube-State-Metrics?

[Kube-State-Metrics](https://github.com/kubernetes/kube-state-metrics) is an agent service that listens to the Kubernetes API and generates metrics about the state of cluster objects, such as deployments, nodes, and pods. Unlike resource metrics (like CPU or memory usage), it provides insights into object-level data, such as the number of replicas or pod status, making it useful for monitoring the desired versus actual state of Kubernetes components.

Metrics are exposed in Prometheus format via an HTTP API. You can consume the data by manually running Prometheus queries, graphing results on a Grafana dashboard, or configuring alerts with Alertmanager.

Kube-State-Metrics provides critical insights into what’s happening in Kubernetes at the object level. For instance, you can discover how many Nodes are in the `NotReady` state, how many replicas are available for a Deployment, or the number of restarts experienced by a Pod. Monitoring these object-specific values lets you take action to improve service reliability and performance.

## How does Kube-State-Metrics work?

Kube-State-Metrics is a Go application that uses the Kubernetes API to take periodic snapshots of your cluster’s state. These snapshots provide a complete list of all the objects in your cluster.

Kube-State-Metrics then serves the data at its `/metrics` HTTP endpoint. You can scrape the endpoint with Prometheus to collect the metrics and make them available to other components in your [observability stack](https://spacelift.io/blog/observability-metrics-logs-traces).

Because Kube-State-Metrics uses exact Kubernetes state snapshots, what’s reported will always accurately match the objects in your cluster. The data isn’t modified or processed before it’s exported, enabling you to work with the raw values and apply your own processing.

## How to install and deploy Kube-State-Metrics

Kube-State-Metrics is maintained as part of the official Kubernetes project, but it isn’t included in the standard cluster distribution. This means you normally need to install the service manually.

Here are four key ways to get Kube-State-Metrics working in your cluster.

### 1\. Cloud-managed Kube-State-Metrics installation options

Many cloud Kubernetes services include a managed Kube-State-Metrics option. This typically installs the service and makes the reported metrics automatically accessible within the cloud provider’s built-in monitoring tools.

Google GKE offers a [Kube-State-Metrics package](https://cloud.google.com/kubernetes-engine/docs/how-to/kube-state-metrics), for example, that you can enable using the Google Cloud Console or CLI. You should check your own cluster provider’s documentation to discover if a similar option is available.

### 2\. Instal Kube-State-Metrics using Helm

The official Kube-State-Metrics [Helm chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-state-metrics) is maintained as part of the Prometheus Community project. Installing Kube-State-Metrics in your cluster is generally the easiest way.

First, register the Helm repository with your Helm CLI:

```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo update
```

Next, use the `helm install` command to install Kube-State-Metrics within a new Kubernetes namespace:

```bash
$ helm install kube-state-metrics \
    prometheus-community/kube-state-metrics \
    -n kube-state-metrics \
    --create-namespace
NAME: kube-state-metrics
LAST DEPLOYED: Tue Feb 11 15:48:02 2025
NAMESPACE: kube-state-metrics
STATUS: deployed
REVISION: 1
...
```

Your Kube-State-Metrics installation will now be ready to use. You can learn about advanced configuration options in the Helm chart’s [documentation](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-state-metrics#configuration).

### 3\. Instal Kube-State-Metrics using Kubernetes manifest files

If you prefer not to use Helm, you can install Kube-State-Metrics from plain [Kubernetes manifest files](https://spacelift.io/blog/kubernetes-manifest-file). The [example](https://github.com/kubernetes/kube-state-metrics/tree/main/examples/standard) manifests provided in the Kube-State-Metrics Git repository are suitable for most standard deployments.

First, clone the repository to your machine:

```bash
$ git clone https://github.com/kubernetes/kube-state-metrics
$ cd kube-state-metrics
```

Next, visit the project’s [GitHub Releases page](https://github.com/kubernetes/kube-state-metrics/releases) to find the version number of the latest release. Use Git to check the matching release tag:

```bash
$ git checkout v2.15.0
```

Finally, use Kubectl to apply the deployment manifests to your cluster:

```bash
$ kubectl apply -k examples/standard
serviceaccount/kube-state-metrics created
clusterrole.rbac.authorization.k8s.io/kube-state-metrics created
clusterrolebinding.rbac.authorization.k8s.io/kube-state-metrics created
service/kube-state-metrics created
deployment.apps/kube-state-metrics created
```

The deployment will target the `kube-system` namespace by default. Note that `kubectl apply -k` must be used instead of `kubectl apply -f` because the manifests use [Kustomize features](https://spacelift.io/blog/kustomize-vs-helm).

### 4\. Instal Kube-State-Metrics with Kube-Prometheus-Stack

Kube-State-Metrics is included with the popular [Kube-Prometheus-Stack Helm chart](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/README.md). This bundles Prometheus, Grafana, and Alertmanager into one easily configurable installation.

You don’t need to do anything to enable Kube-State-Metrics if you’re already using Kube-Prometheus-Stack. Kube-State-Metrics will have been automatically deployed to the same namespace as the other Kube-Prometheus-Stack components (typically `kube-prometheus-stack`). You can learn more about [installing and configuring Kube-Prometheus-Stack](https://spacelift.io/blog/prometheus-kubernetes).

### Configuring Kube-State-Metrics

Kube-State-Metrics supports several optional CLI arguments that you can use to configure the service. You can set arguments for your installation by customizing the `spec.containers.arg` s section of your Kube-State-Metrics Deployment object:

```yaml
spec:
  template:
    spec:
      containers:
        - args:
          - '--port=8000'
```

A complete list of all the supported arguments is available in the [documentation](https://github.com/kubernetes/kube-state-metrics/blob/main/docs/developer/cli-arguments.md).

## Example: Using Kube-State-Metrics with Prometheus

You must use [Prometheus](https://prometheus.io/) to collect and query Kube-State-Metrics output. The steps to correctly configure Prometheus to scrape Kube-State-Metrics may vary depending on how you installed Prometheus in your cluster.

Follow the Prometheus documentation, or the docs for the Helm chart you used, to learn about available configuration methods.

The following example shows a basic Prometheus config file that scrapes the `kube-state-metrics` service in the `kube-system` Kubernetes namespace. Change `kube-system` to the name of the namespace you’ve installed Kube-State-Metrics to.

```yaml
global:
  scrape_interval: 10s
  evaluation_interval: 10s

  scrape_configs:
    - job_name: "kube-state-metrics"
      static_configs:
        - targets: ["kube-state-metrics.kube-system.svc.cluster.local:8080"]
```

You don’t need to manually configure Prometheus if you’ve installed Prometheus and Kube-State-Metrics using the Kube-Prometheus-Stack Helm chart. This chart automatically registers a Prometheus scrape config for the Kube-State-Metrics service.

Once Prometheus is correctly scraping Kube-State-Metrics, you can inspect your data using standard Prometheus queries. You can also connect a Grafana instance to display the results of your queries as dashboard graphs. Detailed guidance on using Prometheus and Grafana is outside the scope of this guide, but you can learn more in our [kube-prometheus-stack tutorial](https://spacelift.io/blog/prometheus-kubernetes).

You can test that Prometheus is successfully scraping your data by running a simple query against a Kube-State-Metrics value. For example, the following query reports the number of Pods currently scheduled to each Node in your Kubernetes cluster:

```bash
sum(kube_pod_info) by (node)

Result:
{node="do-copt-4vcpu-8gb-etxh3"}    60
{node="do-copt-4vcpu-8gb-etxhn"}    58
```

Now let’s look at some more advanced examples.

## Kube-State-Metrics: Key use cases and Kubernetes metrics to track

Kube-State-Metrics reports a large number of metrics for all built-in Kubernetes object types. It’s not possible to provide an exhaustive summary in this guide, but you can find a complete list of metrics in the [documentation](https://github.com/kubernetes/kube-state-metrics/tree/main/docs).

Here’s a summary of some key metrics and example Prometheus queries for common use cases.

1. Get the number of ready pods in each namespace:

```bash
sum by (namespace) (kube_pod_status_ready)

Result:
{namespace="kube-prometheus-stack"}        21
{namespace="kube-system"}                51
```

1. Find pods that have restarted, including their namespaces:

```bash
sum(kube_pod_container_status_restarts_total) by (namespace, pod) > 0

Result:
{namespace="cert-manager", pod="cert-manager-cainjector-858f6466db-l4x98"}        2
{namespace="mysql-operator", pod="mysql-operator-7db94549df-q98dp"}                        1
```

1. Get the number of pod replicas available for each deployment in a specific namespace:

```bash
sum(kube_deployment_status_replicas_available{namespace="kube-prometheus-stack"}) by (deployment)

Result:
{deployment="kube-prometheus-stack-grafana"}                                    1
{deployment="kube-prometheus-stack-kube-state-metrics"}                1
{deployment="kube-prometheus-stack-operator"}                                    1
```

1. Check changes in a deployment’s replica count over the past 5 minutes:

```bash
kube_deployment_status_replicas_available{deployment="demo-deployment"}[5m]

Result:
1 @1739382877.156
2 @1739382907.156
3 @1739382937.156
3 @1739382967.156
3 @1739382997.156
```

1. List images and how many containers use them:

```bash
sum by (image) (kube_pod_container_info)

Result:
{image="quay.io/jetstack/cert-manager-controller:v1.5.5"}                        1
{image="docker.io/digitalocean/cilium:v1.15.8-conformance-fix"}            2
```

1. Get the last scheduled time for each CronJob in a namespace:

```bash
max(kube_cronjob_status_last_schedule_time{namespace="demo-namespace"}) by (cronjob)

Result:
{cronjob="demo-cron"}            1739336400
```

1. Get the Persistent Volumes using a specific storage class:

```bash
count(kube_persistentvolume_info{storageclass="demo-storage"}) by (persistentvolume)

Result:
{persistentvolume="pvc-71d69277-d842-4ab2-8f87-cb7674cccbe6"}            1
{persistentvolume="pvc-1caca021-671e-46cb-9a58-2e2b069501d4"}            1
```

1. Find how many pods have been created in a namespace in the past day:

```bash
count(increase(kube_pod_created{namespace="demo-namespace"}[1d]))

Result:
{}
```

1. Get the top three namespaces with the most new pods created in the past day:

```bash
topk(3, count by (namespace) (increase(kube_pod_created[1d])))

Result:
{namespace="velero"}                299
{namespace="demo-namespace"}        4
{namespace="demo-namespace-2"}        2
```

1. Get the number of successful completions for each job in a namespace:

```bash
sum by (job_name) (kube_job_spec_completions{namespace="demo-namespace"})

Result:
{job_name="demo-job"}        3
{job_name="demo-job-2"}     1
```

- [How to Maintain Operations Around Kubernetes Cluster](https://spacelift.io/blog/how-to-maintain-operations-around-kubernetes-cluster)
- [15 Kubernetes Best Practices to Follow](https://spacelift.io/blog/kubernetes-best-practices)
- [26 Top Kubernetes Tools for Your K8s Ecosystem](https://spacelift.io/blog/kubernetes-tools)

## Kube-State-Metrics vs Kubernetes Metrics-Server

Kube-State-Metrics is only one part of the Kubernetes metrics ecosystem. [Metrics-Server](https://github.com/kubernetes-sigs/metrics-server) is another commonly discussed component that you should also include in your observability strategy. Whereas Kube-State-Metrics reports metrics about the states of objects in your cluster, Metrics-Server exposes real-time CPU and memory resource utilization metrics for your Nodes and Pods.

After installing Metrics-Server in your cluster, you can use the [kubectl top command](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_top) to monitor resource consumption. Metrics-Server data also powers [Kubernetes autoscaling](https://spacelift.io/blog/kubernetes-scaling) features, which are used by the Horizontal Pod Autoscaler and Vertical Pod Autoscaler components to detect when cluster capacity is being reached.

In summary, **Kubernetes clusters operating in production environments need both Kube-State-Metrics and Metrics-Server**. Kube-State-Metrics lets you inspect the states of your objects, whereas Metrics-Server provides resource utilization stats that enable autoscaling.

## Kube-State-Metrics vs Cluster-level metrics

Cluster-level metrics provide valuable insights into the performance and health of the infrastructure supporting your Kubernetes workloads. These metrics are typically gathered from sources such as the Kubelet, Metrics Server, and cAdvisor and focus on real-time resource consumption, including CPU, memory, disk usage, and network activity.

While cluster-level metrics inform you about what’s happening in terms of system-level indicators like CPU and memory usage, Kube-State-Metrics helps explain why these events might be occurring by surfacing the current state and configuration of Kubernetes objects. Combining both types of metrics gives you a complete picture of your Kubernetes environment, which is vital for debugging, optimizing, and scaling production systems effectively.

## Kube-State-Metrics best practices

Kube-State-Metrics often performs well using its default configuration, but it can need fine-tuning when operating at scale. Here are some best practices to keep in mind:

### 1\. Adjust Kube-State-Metrics resource assignments to match your cluster’s size

Kube-State-Metrics resource consumption generally increases in proportion with the number of objects in your cluster. It’s recommended you allocate at least 250 MiB of memory and 0.1 CPU core to each replica, but larger environments may demand more resources.

You can adjust the resource constraints by modifying the `spec.containers.resources` field of your Kube-State-Metrics [Deployment object](https://spacelift.io/blog/kubernetes-deployment-yaml):

```yaml
spec:
  template:
    spec:
      containers:
        resources:
          limits:
            memory: 300Mi
            cpu: 1.0
          requests:
            memory: 300Mi
            cpu: 0.2
```

The CPU limit setting is especially important, as low limits will cause throttling that prevents Kube-State-Metrics from processing its work queue fast enough.

Not only does this prevent metrics from being reported on time, but the extra items in the queue also cause a knock-on increase in memory consumption.

### 2\. Enable horizontal sharding for improved performance at scale

Kube-State-Metrics replicas can be sharded to improve horizontal scalability. To use sharding, you should deploy your Kube-State-Metrics Pods using a Kubernetes StatefulSet, instead of a plain Deployment. You must then set the `--shard` and `--total-shards` flags on each replica. This informs the replica which shard it is and how many shards there are in total.

Sharding should generally only be used in large-scale environments. You can find detailed guidance on configuring it in the Kube-State-Metrics [documentation](https://github.com/kubernetes/kube-state-metrics?tab=readme-ov-file#horizontal-sharding).

### 3\. Assign Kube-State-Metrics correct RBAC permissions

Kube-State-Metrics generally requires read-only access to your entire Kubernetes cluster. This ensures it can collect metrics about all of your objects. The official installation manifests and Helm chart automatically create an appropriate [ServiceAccount and RoleBinding](https://spacelift.io/blog/kubernetes-rbac) for Kube-State-Metrics to use, but it’s possible you’ll want to restrict access to specific namespaces if you’re in a locked-down environment.

You can achieve this manually by creating a ServiceAccount and per-namespace RoleBindings, then configuring Kube-State-Metrics only to collect metrics from those namespaces. You can find an example in the [documentation](https://github.com/kubernetes/kube-state-metrics?tab=readme-ov-file#limited-privileges-environment).

### 4\. Use Alertmanager to be notified when metrics change

Metrics are most useful when you’re notified of important changes as they happen. You should use [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager) within Prometheus to configure alerts for key Kube-State-Metrics changes. This lets you proactively respond to emerging incidents instead of reacting after the fault has already occurred.

The following Alertmanager rule demonstrates how to use Kube-State-Metrics data to trigger a warning when a container in a Pod has restarted more than five times in the past five minutes:

```yaml
alert: ContainerRestartAlert
annotations:
  summary: "Container {{ $labels.container }} in {{ $labels.pod }} is restarting often"
  expr: sum(increase(kube_pod_container_status_restarts_total[5m])) by (pod, container) > 5
  for: 0m
  labels:
    severity: warning
```

### 5\. Configure metric allow and deny lists to block unwanted noisy metrics

Kube-State-Metrics defaults to collecting all possible metrics about the objects in your cluster. If you’re not planning to use this data, then you can configure allow and deny lists to filter out noisy alerts. This can improve performance and make it easier to find the meaningful metrics that you’re interested in.

You can configure allow and deny lists using Kube-Metrics-Server CLI arguments. Key options include:

- **`--metric-allowlist`** **and** **`--metric-denylist`****:** A comma-separated list of specific metric names to include or exclude, such as `kube_pod_created,kube_pod_info`. The two options are mutually exclusive.
- **`--namespaces`** **and** **`--namespaces-denylist`****:** A comma-separated list of namespace names to allow or deny metrics collection from.
- **`` `--resources` ``:** A comma-separated list of Kubernetes resource names to include, such as `pods,jobs`. There’s no corresponding exclude option.

Combining these settings lets you precisely control which metrics are reported.

## Managing Kubernetes with Spacelift

If you need help managing your Kubernetes projects, consider [Spacelift](https://docs.spacelift.io/vendors/kubernetes/). It brings with it a GitOps flow, so your Kubernetes Deployments are synced with your Kubernetes Stacks, and pull requests show you a preview of what they’re planning to change.

With Spacelift, you get:

- [Policies](https://docs.spacelift.io/concepts/policy) to control what kind of resources engineers can create, what parameters they can have, how many approvals you need for a run, what kind of task you execute, what happens when a pull request is open, and where to send your notifications
- [Stack dependencies](https://docs.spacelift.io/concepts/stack/stack-dependencies.html) to build multi-infrastructure automation workflows with dependencies, having the ability to build a workflow that can combine Terraform with Kubernetes, Ansible, and other infrastructure-as-code (IaC) tools such as OpenTofu, Pulumi, and CloudFormation,
- Self-service infrastructure via [Blueprints](https://docs.spacelift.io/concepts/blueprint.html), enabling your developers to do what matters – developing application code while not sacrificing control
- Creature comforts such as [contexts](https://docs.spacelift.io/concepts/configuration/context.html) (reusable containers for your environment variables, files, and hooks), and the ability to run arbitrary code
- [Drift detection](https://docs.spacelift.io/concepts/stack/drift-detection.html) and optional remediation

If you want to learn more about Spacelift, [create a free account today](https://spacelift.io/free-trial) or [book a demo with one of our engineers](https://spacelift.io/schedule-demo).

## Key points

Kube-State-Metrics is a Kubernetes observability tool that exposes metrics about the objects in your cluster. It enables you to efficiently monitor the states of your objects using Prometheus queries. The data can help you troubleshoot errors and spot emerging problems before they cause incidents.

It’s good practice to use Kube-State-Metrics in all production clusters, but the tool is only one part of a complete Kubernetes monitoring strategy.

You also need solutions such as Metrics-Server or Prometheus’ [Node-Exporter](https://github.com/prometheus/node_exporter) to track your cluster’s resource utilization and broader activity. You can learn more about setting up Prometheus for Kubernetes by reading our separate [tutorial](https://spacelift.io/blog/prometheus-kubernetes), or check out our round-up of [20+ Popular DevOps Monitoring Tools](https://spacelift.io/blog/devops-monitoring-tools) to find more options for implementing cluster observability.

### Manage Kubernetes easier and faster

Spacelift allows you to automate, audit, secure, and continuously deliver your infrastructure. It helps overcome common state management issues and adds several must-have features for infrastructure management.

[Learn more](https://spacelift.io/how-it-works)

### Written by

![avatar_image_jamesw](https://spacelift.io/yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)

avatar\_image\_jamesw

### James Walker

James Walker is the founder of [Heron Web](https://www.heron-web.com/), a UK-based software development studio providing bespoke solutions for SMEs. He has experience managing complete end-to-end web development workflows with DevOps, CI/CD, Docker, and Kubernetes. James is also a technical writer and has written extensively about the software development lifecycle, current industry trends, and DevOps concepts and technologies.

- [jhwalker.net](https://www.jhwalker.net/)

### Read also

[![Kubernetes Persistent Volumes &#8211; Tutorial and Examples](https://spacelift.io/yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)

Kubernetes Persistent Volumes – Tutorial and Examples

](https://spacelift.io/blog/kubernetes-persistent-volumes)

[Kubernetes](https://spacelift.io/blog/kubernetes) 14 min read

[

##### Kubernetes Persistent Volumes – Tutorial and Examples

](https://spacelift.io/blog/kubernetes-persistent-volumes)

[![What are Kubernetes Custom Resource Definitions (CRDs)?](https://spacelift.io/yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)

What are Kubernetes Custom Resource Definitions (CRDs)?

](https://spacelift.io/blog/kubernetes-crd)

[Kubernetes](https://spacelift.io/blog/kubernetes) 12 min read

[

##### What are Kubernetes Custom Resource Definitions (CRDs)?

](https://spacelift.io/blog/kubernetes-crd)

[![Kubectl Apply vs. Kubectl Create &#8211; What’s the Difference?](https://spacelift.io/yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)

Kubectl Apply vs. Kubectl Create – What’s the Difference?

](https://spacelift.io/blog/kubectl-apply-vs-create)

[Kubernetes](https://spacelift.io/blog/kubernetes) 8 min read