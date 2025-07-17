---
type: source-note
title: Prometheus Monitoring for Kubernetes Cluster [Tutorial]
id: 20250717140709
created: 2025-07-17T14:08:09
source:
  - web
url: https://spacelift.io/blog/prometheus-kubernetes
tags:
  - source-note
  - prometheus
processed: false
archived: false
---
[Kubernetes](https://spacelift.io/blog/kubernetes)

![Prometheus Monitoring for Kubernetes Cluster [Tutorial]](https://spacelift.io/_next/image?url=https%3A%2F%2Fsecure.gravatar.com%2Favatar%2F402ba26575ce7909a6b670410ee12eda9ce420710cc68c3f05adc425d2ba9acb%3Fs%3D48%26d%3Dmm%26r%3Dg&w=96&q=75)

Prometheus Monitoring for Kubernetes Cluster \[Tutorial\]

· 13 min read

Reviewed by:

![How to Setup Prometheus Monitoring On a Kubernetes Cluster](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2023%2F01%2F145.prometheus-monitoring-on-kubernetes-cluster.png&w=3840&q=100)

How to Setup Prometheus Monitoring On a Kubernetes Cluster

[Kubernetes](https://spacelift.io/blog/kubernetes-tutorial) is the most popular orchestrator for running containerized workloads in production. It gives you a complete set of tools for deploying, scaling, and administering your containers.

Kubernetes alone isn’t enough to successfully operate apps, though. You also need visibility into cluster utilization, performance, and any errors that occur.[Prometheus](https://prometheus.io/) is an open-source monitoring system that collects metrics in a time series database, allowing you to answer these questions.

In this article, you’ll learn how to set up and use Prometheus with your Kubernetes cluster. We’ll cover the basics of installing Prometheus, querying data, setting up visual dashboards, and managing alerting rules. You’ll need [Kubectl](https://kubernetes.io/docs/reference/kubectl/kubectl), [Helm](https://helm.sh/docs/intro/install), and a Kubernetes cluster before you begin.

We will cover:

1. [What is Prometheus?](https://spacelift.io/blog/prometheus-kubernetes#what-is-prometheus)
2. [Why use Prometheus for Kubernetes monitoring?](https://spacelift.io/blog/prometheus-kubernetes#why-use-prometheus-for-kubernetes-monitoring)
3. [What is kube-prometheus-stack?](https://spacelift.io/blog/prometheus-kubernetes#what-is-kubeprometheusstack)
4. [How to set up Prometheus monitoring on a Kubernetes cluster](https://spacelift.io/blog/prometheus-kubernetes#how-to-set-up-prometheus-monitoring-on-a-kubernetes-cluster)
5. [Kubernetes Prometheus best practices](https://spacelift.io/blog/prometheus-kubernetes#kubernetes-prometheus-best-practices)

## What is Prometheus?

Prometheus is an open-source monitoring and alerting toolkit under the CNCF umbrella. It has a robust time series database designed for optimal performance in storing and querying metric data.

It has a pull-based approach to metrics collection, actively getting data from application endpoints and servers at regular intervals. In this way, it provides real-time insights into the health and performance of the monitored system, allowing for dynamic discovery of targets through various mechanisms such as K8s service discovery.

## Why use Prometheus for Kubernetes monitoring?

There are many reasons why you would use Prometheus for Kubernetes monitoring:

- **Built-in support for K8s service discovery** – automatically discover and monitor new services and pods as they are deployed and scaled up within the K8s cluster
- **Rich data model** – granular categorization and querying of metrics based on various attributes such as pod labels, namespace, service name, etc
- **Integration with visualization tools** – integrates seamlessly with Grafana, enabling users to create custom dashboards and visualizations to gain deeper insights into K8s metrics
- **Scalability and performance** – Prometheus handles large volumes of data with minimal resource overhead, making it ideal for Kubernetes
- **Proven reliability** – It has been adopted by many organizations of all sizes for monitoring Kubernetes environments, being effective with the everchanging status of K8s environments
- **Community support** – Prometheus, as well as Kubernetes, are part of CNCF, which has a large community and powerful documentation and tutorials
- **Open-source** – Prometheus is open-source, making it a flexible choice for monitoring Kubernetes

See also: [Prometheus with Helm Chart on Kubernetes Setup](https://spacelift.io/blog/prometheus-helm-chart)

## What is kube-prometheus-stack?

The [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/README.md) Helm chart is the simplest way to bring up a complete Prometheus stack inside your Kubernetes cluster. It bundles several different components in one automated deployment:

- **Prometheus** – Prometheus is the time series database that scrapes, stores, and exposes the metrics from your Kubernetes environment and its applications.
- **Node-Exporter** – Prometheus works by scraping data from a variety of configurable sources called *exporters*. [Node-Exporter](https://github.com/prometheus/node_exporter) is an exporter which collects resource utilization data from the Nodes in your Kubernetes cluster. The kube-prometheus-stack chart automatically deploys this exporter and configures your Prometheus instance to scrape it.
- **Kube-State-Metrics** – [Kube-State-Metrics](https://spacelift.io/blog/kube-state-metrics) is another exporter that supplies data to Prometheus. It exposes information about the API objects in your Kubernetes cluster, such as Pods and containers.
- **Grafana** – Although you can directly query Prometheus, this is often tedious and repetitive. [Grafana](https://grafana.com/) is an observability platform that works with several data sources, including Prometheus databases. You can use it to create dashboards that surface your Prometheus data.
- **Alertmanager** – [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager) is a standalone Prometheus component that provides notifications when metrics change. You can use it to get an email when CPU utilization spikes or a Slack notification if a Pod is evicted, for example.

Deploying, configuring, and maintaining all these components individually can be burdensome for administrators. Kube-Prometheus-Stack provides an automated solution that performs all the hard work for you.

## How to set up Prometheus monitoring on a Kubernetes cluster

Let’s see how to set up and use Prometheus with your Kubernetes cluster in practice.

### 1\. Instal kube-prometheus-stack

First, register the chart’s repository in your Helm client:

```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" has been added to your repositories
```

Next, update your repository lists to discover the chart:

```bash
$ helm repo update
```

Now you can run the following command to deploy the chart into a new namespace in your cluster:

```bash
$ helm install kube-prometheus-stack \
  --create-namespace \
  --namespace kube-prometheus-stack \
  prometheus-community/kube-prometheus-stack
NAME: kube-prometheus-stack
LAST DEPLOYED: Tue Jan  3 14:26:18 2023
NAMESPACE: kube-prometheus-stack
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace kube-prometheus-stack get pods -l "release=kube-prometheus-stack"
```

It can take a couple of minutes for the chart’s components to start. Run the following command to check how they’re progressing:

```bash
$ kubectl -n kube-prometheus-stack get pods
NAME                                                       READY   STATUS    RESTARTS      AGE
alertmanager-kube-prometheus-stack-alertmanager-0          2/2     Running   1 (66s ago)   83s
kube-prometheus-stack-grafana-5cd658f9b4-cln2c             3/3     Running   0             99s
kube-prometheus-stack-kube-state-metrics-b64cf5876-52j8l   1/1     Running   0             99s
kube-prometheus-stack-operator-754ff78899-669k6            1/1     Running   0             99s
kube-prometheus-stack-prometheus-node-exporter-vdgrg       1/1     Running   0             99s
prometheus-kube-prometheus-stack-prometheus-0              2/2     Running   0             83s
```

Once all the Pods show as Running, your monitoring stack is ready to use. The data exposed by the exporters will be automatically scraped by Prometheus.

Now you can start querying your metrics.

### 2\. Run a Prometheus query

Prometheus includes a web UI that you can use to query your data. This is not exposed automatically. You can access it by using Kubectl port forwarding to redirect local traffic to the service in your cluster:

```bash
$ kubectl port-forward -n kube-prometheus-stack svc/kube-prometheus-stack-prometheus 9090:9090
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
```

This command redirects traffic to `localhost:9090` to the Prometheus service. Visiting this URL in your web browser will reveal the Prometheus UI:

![prometheus monitring prometheus UI](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2023%2F01%2Fprometheus-monitring-prometheus-UI.png&w=3840&q=75)

prometheus monitring prometheus UI

The “Expression” input at the top of the screen is where you enter your queries as [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics) expressions. Start typing into the input to reveal autocomplete suggestions for the available metrics.

Try selecting the `node_memory_Active_bytes` metric, which surfaces the memory consumption of each of the Nodes in your cluster. Press the “Execute” button to run your query. The results will be displayed in a table that provides the query’s raw output:

![prometheus monitoring query's raw output](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2023%2F01%2Fprometheus-monitoring-querys-raw-output.png&w=3840&q=75)

prometheus monitoring query's raw output

Most metrics are easier to interpret as **graphs**.

Switch to the “Graph” tab at the top of the screen to see a visualization of the metric over time. You can use the controls above the graph to change the time period that’s displayed.

![prometheus monitoring graphs](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2023%2F01%2Fprometheus-monitoring-graphs.png&w=3840&q=75)

prometheus monitoring graphs

PromQL [queries](https://prometheus.io/docs/prometheus/latest/querying/basics) allow detailed interrogation of your data. Manually running individual queries in the Prometheus UI is an inefficient form of monitoring, however.

Next, let’s use Grafana to visualize metrics conveniently on live dashboards.

- [How to Provision Kubernetes Cluster with Terraform](https://spacelift.io/blog/provision-kubernetes-cluster-terraform-spacelift)
- [15 Kubernetes Best Practices to Follow](https://spacelift.io/blog/kubernetes-best-practices)
- [Common Infrastructure Challenges and How to Solve Them](https://spacelift.io/blog/infrastructure-problems-that-spacelift-solves)

### 3\. Use Grafana dashboards

Start a new Kubectl port forwarding session to access the Grafana UI. Use port 80 as the target because this is what the Grafana service binds to.

You can map it to a different local port, such as 8080, in this example:

```bash
$ kubectl port-forward -n kube-prometheus-stack svc/kube-prometheus-stack-grafana 8080:80
Forwarding from 127.0.0.1:8080 -> 3000
Forwarding from [::1]:8080 -> 3000
```

Next visit `http://localhost:8080` in your browser. You’ll see the Grafana login page. The default user account is admin with a password of `prom-operator`.

![prometheus monitoring Grafana login page](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2023%2F01%2Fprometheus-monitoring-Grafana-login-page.png&w=3840&q=75)

prometheus monitoring Grafana login page

After you’ve logged in, you’ll initially reach the Grafana welcome screen:

![prometheus monitoring Grafana welcome page](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2023%2F01%2Fprometheus-monitoring-Grafana-welcome-page.png&w=3840&q=75)

prometheus monitoring Grafana welcome page

Use the sidebar to switch to the Dashboards screen. Its icon is four squares arranged to resemble panes of glass. This is where all your saved dashboards can be found, including the prebuilt ones that come with Kube-Prometheus-Stack deployments.

![prometheus monitoring Dashboards screen](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2023%2F01%2Fprometheus-monitoring-Dashboards-screen.png&w=3840&q=75)

prometheus monitoring Dashboards screen

### 4\. Explore the Grafana pre-built dashboards

There are several included dashboards that contain the metrics scraped from Node-Exporter, Kube-State-Metrics, and various Kubernetes and Prometheus components. Here are a few notable ones:

#### Monitoring cluster utilization with “Kubernetes / Compute Resources / Cluster”

This dashboard provides an overview of the resource utilization for your entire cluster. Headline statistics are displayed at the top, with more detailed information presented in panels below.

![prometheus Monitoring Cluster Utilization](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2023%2F01%2Fprometheus-Monitoring-Cluster-Utilization.png&w=3840&q=75)

prometheus Monitoring Cluster Utilization

#### Viewing a node’s resource consumption with “Node Exporter / Nodes”

Data collected by Node-Exporter is provided by this dashboard. It shows detailed resource utilization information on a per-Node basis. You can change the selected Node using the “instance” dropdown at the top of the dashboard.

![Prometheus and Grafana stack](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2023%2F01%2Fprometheus-Viewing-a-Nodes-Resource-Consumption.png&w=3840&q=75)

Prometheus and Grafana stack

#### Viewing the resource consumption of individual pods with “Kubernetes / Compute Resources / Pod”

This dashboard shows the resource requests, limits, quotas, and utilization for individual Pods. You can select the namespace and Pod to view from the dropdowns at the top of the screen.

![prometheus Viewing the Resource Consumption](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2023%2F01%2Fprometheus-Viewing-the-Resource-Consumption.png&w=3840&q=75)

prometheus Viewing the Resource Consumption

The time frame can be customized on all Grafana dashboards using the controls in the top-right corner of the screen. You can refresh the data or change the auto-refresh interval with the button next to the time frame selector.

### 5\. Configure alerts with Alertmanager

Monitoring must be automated to be effective. You need to receive alerts when important metric stops meeting expectations, such as when a spike in memory consumption occurs. Otherwise, you have to continually check your dashboards or run queries to determine whether you need to take action.

Prometheus includes [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager) to send you a notification when your metrics trigger an alert. Alertmanager supports multiple [receivers](https://prometheus.io/docs/alerting/latest/configuration) that act as destinations for your alerts, such as email, Slack, messaging apps, and your own webhooks.

Kube-Prometheus-Stack’s bundled Alertmanager is configured by merging in custom chart values when you deploy the stack with Helm. First, prepare a [YAML](https://spacelift.io/blog/yaml) file that nests your Alertmanager settings under the top-level alertmanager key. Here’s an example that sends all alerts to a webhook URL:

```yaml
alertmanager:
  config:
    global:
      resolve_timeout: 5m
    route:
      receiver: demo-webhook
      group_wait: 5s
      group_interval: 10s
      repeat_interval: 1h
    receivers:
      - name: "null"
      - name: demo-webhook
        webhook_configs:
          - url: http://example.com/webhook
            send_resolved: true
```

The route section specifies that alerts should be directed to the `demo-webhook` receiver. This is configured to send a POST request to `http://example.com/webhook` each time an alert is triggered or resolved. The request’s payload is described in [the Alertmanager documentation](https://prometheus.io/docs/alerting/latest/configuration/#webhook_config). Note that the extra `"null”` receiver is required [due to a bug](https://github.com/prometheus-community/helm-charts/issues/255) that otherwise prevents your route from working.

Save your YAML file to `alertmanager-config.yaml` in your working directory. Next run the following command to redeploy the Prometheus stack and apply your Alertmanager settings:

```bash
$ helm upgrade --reuse-values \
  -f alertmanager-config.yaml \
  -n kube-prometheus-stack \
  kube-prometheus-stack
  prometheus-community/kube-prometheus-stack
```

Don’t worry – you won’t lose any of your existing data. The command performs an in-place upgrade of your deployment.

It could take a few minutes for Alertmanager to reload its configuration after the deployment completes. You’ll then begin to receive requests to your webhook URL, as alerts are triggered.

To send a test alert, first start a port forwarding session to your Alertmanager instance:

```bash
$ kubectl port-forward -n kube-prometheus-stack svc/kube-prometheus-stack-alertmanager 9093:9093
```

Next run the following command to simulate triggering a basic alert from a Kubernetes service in a specific namespace:

```bash
$ curl -H 'Content-Type: application/json' -d '[{"labels":{"alertname":"alert-demo","namespace":"demo","service":"demo"}}]' http://127.0.0.1:9093/api/v1/alerts
```

After a few moments, you should receive a request to your webhook URL. The request’s body will describe the alert’s details.

## Kubernetes Prometheus best practices

You should consider the following when using Prometheus with Kubernetes:

1. Use the Prometheus operator for K8s
2. Configure service monitors
3. Leverage K8s labels and annotations
4. Take advantage of persistent storage for Prometheus
5. Setup Alertmanager
6. Monitor Prometheus performance
7. Secure your Prometheus instance
8. Do regular updates

### 1\. Use the Prometheus operator for K8s

Deploying Prometheus using the [Prometheus operator](https://spacelift.io/blog/prometheus-operator), will help with managing Prometheus instances and their configurations automatically.

This will allow you to define also your monitoring requirements declaratively using K8s CRDs.

### 2\. Configure service monitors

Service monitors are used to dynamically discover and configure targets for monitoring inside your K8s cluster based on labels and annotations.

### 3\. Leverage K8s labels and annotations

Labels and annotations are key for organizing resources inside your K8s cluster, making it easier to define what to monitor. Using meaningful labels for your your K8s resources and using these labels for dynamic and flexible monitoring setups is key in making your integration worthwhile.

### 4\. Take advantage of persistent storage for Prometheus

Prometheus uses a time series database to store your data. Without having persistent storage, you risk losing all the data you have gathered if the Prometheus pod is restarted. A best practice here would be to use Persistent Volumes (PV) in K8s to ensure that your Prometheus data is retained across restarts.

### 5\. Setup Alertmanager

Prometheus’ Alertmanager takes care of deduplicating, grouping, and routing alerts sent by client applications. You should configure it to efficiently manage your alerts, and you can also leverage it to send these alerts to email, Slack, or even other notification channels based on their severity or other aspects you define.

### 6\. Monitor Prometheus performance

Even if Prometheus is used for monitoring aspects related to your K8s cluster in this context, it is crucial to monitor its performance as well, to ensure it doesn’t become a bottleneck. If you have a large number of targets or high metrics, adjust the resources allocated to Prometheus as needed.

### 7\. Secure your Prometheus instance

When it comes to infrastructure components, you should always do your due diligence to secure them and Prometheus makes no exception. It is essential to prevent unauthorized access to your monitoring data, so for that you should use K8s RBAC to control access and also enable HTTPS for Prometheus’ endpoints and the web interface.

### 8\. Do regular updates

Keeping Prometheus and the Operator up to date ensures that you have the latest security patches, features, and performance improvements inside your Prometheus instance.

## Key points

Good [observability](https://spacelift.io/blog/devops-tools#8-observability) is essential for Kubernetes clusters running production workloads. You need to understand resource utilization, see where Pods are being scheduled, and track the errors and logs emitted by your applications.

Kube-Prometheus-Stack is a convenient route to setting up monitoring for your cluster. It configures Prometheus, Grafana, Alertmanager, and vital metrics exporters for you, reducing maintenance overheads. The basic installation comes with useful prebuilt dashboards that you can extend with custom queries and metrics scraped from your own applications. Instrumenting a system for Prometheus is a complex topic, but you can get started by [exploring the official client libraries](https://prometheus.io/docs/instrumenting/clientlibs/) for exporting metrics from your code.

Need an even simpler way to manage CI/CD pipelines on Kubernetes? Check out how [Spacelift](https://spacelift.io/) can help you cut down complexity and automate your infrastructure. It’s even got a Prometheus exporter ready to deliver metrics from your Spacelift account to your Grafana dashboards and other tools! Learn more with our tutorial on [Monitoring Your Spacelift Account via Prometheus](https://spacelift.io/blog/prometheus-exporter-for-spacelift).

### The Most Flexible CI/CD Automation Tool

Spacelift is an alternative to using homegrown solutions on top of a generic CI. It helps overcome common state management issues and adds several must-have capabilities for infrastructure management.

[Start free trial](https://spacelift.io/free-trial)

### Written by

![avatar_image_jamesw](https://spacelift.io/yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)

avatar\_image\_jamesw

### James Walker

James Walker is the founder of [Heron Web](https://www.heron-web.com/), a UK-based software development studio providing bespoke solutions for SMEs. He has experience managing complete end-to-end web development workflows with DevOps, CI/CD, Docker, and Kubernetes. James is also a technical writer and has written extensively about the software development lifecycle, current industry trends, and DevOps concepts and technologies.

- [jhwalker.net](https://www.jhwalker.net/)

### Read also

[![Kubernetes Sidecar Container &#8211; Best Practices and Examples](https://spacelift.io/yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)

Kubernetes Sidecar Container – Best Practices and Examples

](https://spacelift.io/blog/kubernetes-sidecar-container)

[Kubernetes](https://spacelift.io/blog/kubernetes) 15 min read

[

##### Kubernetes Sidecar Container – Best Practices and Examples

](https://spacelift.io/blog/kubernetes-sidecar-container)

[![Kustomize vs. Helm &#8211; How to Use &#038; Comparison](https://spacelift.io/yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)

Kustomize vs. Helm – How to Use & Comparison

](https://spacelift.io/blog/kustomize-vs-helm)

[Kubernetes](https://spacelift.io/blog/kubernetes) 11 min read

[

##### Kustomize vs. Helm – How to Use & Comparison

](https://spacelift.io/blog/kustomize-vs-helm)

[![Guide to Kubernetes Liveness Probes with Examples](https://spacelift.io/yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)

Guide to Kubernetes Liveness Probes with Examples

](https://spacelift.io/blog/kubernetes-liveness-probe)

[Kubernetes](https://spacelift.io/blog/kubernetes) 17 min read