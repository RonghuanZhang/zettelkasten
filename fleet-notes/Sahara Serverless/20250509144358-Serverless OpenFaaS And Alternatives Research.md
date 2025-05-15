---
"type:": fleet-note
"title:": 20250509144358-Serverless OpenFaaS And Alternatives Research
"id:": 20250509151525
"created:": 2025-05-09T15:15:25
source: 
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---
# Core Requirments

* Support service deployment. 
* Support function deployment. (FaaS)
* Support job deployment. (Optional)
* Autoscaling
	* Based on traffic.
	* Support scale to zero.
* Support invoking HTTP.
* Support trigger function.
* Authentication and authorization.
	* The Open API, SDK and CLI auth
	* HTTP Invoke
	* Function Trigger
* Multi tenancy

# Open Source Software

## [OpenFaaS](https://www.openfaas.com/) - Golang

> Serverless Functions, Made Simple. OpenFaaS® makes it simple to deploy both functions and existing code to Kubernetes.

**Not meeting requirements**
* [Only the Enterprise has the IAM(Identity Access Management)](https://docs.openfaas.com/openfaas-pro/iam/overview/)
* Only [OpenFaaS Pro supports the service account for functions](https://docs.openfaas.com/reference/workloads/#custom-service-account).
* ==The [Scale to Zero functionality of OpenFaaS Pro ](https://docs.openfaas.com/openfaas-pro/scale-to-zero) can be used to scale idle functions down to 0 replicas, to save on resources.== (There are two editions of OpenFaaS Pro - Standard and for Enterprises.)
## [Apache OpenWhisk](https://openwhisk.apache.org/) - Scala

> Apache OpenWhisk is an open source, distributed [Serverless](https://en.wikipedia.org/wiki/Serverless_computing) platform that executes functions (fx) in response to events at any scale. OpenWhisk manages the infrastructure, servers and scaling using Docker containers so you can focus on building amazing and efficient applications.
> 
> The OpenWhisk platform supports a programming model in which developers write functional logic (called [Actions](https://github.com/apache/openwhisk/blob/master/docs/actions.md#openwhisk-actions)), in any supported programming language, that can be dynamically scheduled and run in response to associated events (via [Triggers](https://github.com/apache/openwhisk/blob/master/docs/triggers_rules.md#creating-triggers-and-rules)) from external sources ([Feeds](https://github.com/apache/openwhisk/blob/master/docs/feeds.md#implementing-feeds)) or from HTTP requests. The project includes a REST API-based Command Line Interface (CLI) along with other tooling to support packaging, catalog services and many popular container deployment options.

[Apache OpenWhisk is an open cloud platform for serverless computing that uses cloud computing resources as services. Compared to other open-source projects (Fission, Kubeless, IronFunctions), Apache OpenWhisk is characterized by a large codebase, high-quality features, and the number of contributors. However, the overly large tools for this platform (CouchDB, Kafka, Nginx, Redis, and Zookeeper) cause difficulties for developers. In addition, this platform is imperfect in terms of security.](https://www.cncf.io/blog/2020/04/13/serverless-open-source-frameworks-openfaas-knative-more/)

OpenWhisk relies on too many components, making learning and maintaining challenging.

The project had only two versions.

## [Knative](https://knative.dev/docs/) - Golang

> Knative is an Open-Source Enterprise-level solution to build Serverless and Event Driven Applications

Most used in the 2021 CNCF report.
Based on the K8s. Easy to migrate from one cloud to another.

## [KEDA](https://keda.sh/) - Golang

>**KEDA** is a [Kubernetes](https://kubernetes.io/)-based Event Driven Autoscaler. With KEDA, you can drive the scaling of any container in Kubernetes based on the number of events needing to be processed.
>
>**KEDA** is a single-purpose and lightweight component that can be added into any Kubernetes cluster. KEDA works alongside standard Kubernetes components like the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) and can extend functionality without overwriting or duplication. With KEDA, you can explicitly map the apps you want to use event-driven scale, with other apps continuing to function. This makes KEDA a flexible and safe option to run alongside any number of any other Kubernetes applications or frameworks.

Only support the autoscaling—infrastructure layer.

## [Fission](https://fission.io/) - Golang

>Fission is a framework for serverless functions on Kubernetes.
>
>Write short-lived functions in any language, and map them to HTTP requests (or other event triggers).
>
>Deploy functions instantly with one command. There are no containers to build, and no Docker registries to manage.

Only support function. Fission provides autoscaling for functions based on CPU usage. It's not enough to use.

# Summary


| Serverless Engine | Service | Function | Cron Trigger                                                                         | Autoscaling                                                                      | HTTP Invocation | Function Trigger | Auth                                                                                                  | Multi-tenancy              |
| ----------------- | ------- | -------- | ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- | --------------- | ---------------- | ----------------------------------------------------------------------------------------------------- | -------------------------- |
| **OpenFaaS**      | ✅       | ✅        | ✅ Trigger function cron. Need to deploy manually.<br>Cron connector or K8s cron job. | ✅ CE only supports the QPS autoscaler.<br><br>❌ CE doesn't support scale-to-zero | ✅               | ✅                | ❌ REST API or SDK: CE supports only basic auth.<br><br>❌ Only Pro supports the IAM auth for functions | ❌ Only for Enterprise      |
| **Knative**       | ✅       | ✅        | ✅ Trigger function cron. By Cron Event.                                              | ✅ QPS<br>✅ Concurrency<br>✅ CPU<br>✅ Memory<br>✅ Custom Metrics                  | ✅               | ✅                | ❌ Depend ing on gthe ateway or Istio                                                                  | ❌ Depends on K8s or others |
|                   |         |          |                                                                                      |                                                                                  |                 |                  |                                                                                                       |                            |


---
# Questions

## What's the future roadmap of the Sahara AI infrustructure?

## Should we consider the migration to another cloud platform?

## Should we build our serverless platform based on a serverless engine?



# Reference
* [CNCF Landscape](https://landscape.cncf.io/?group=serverless)

* [Infrastructure as Code Tool Recommendation for GCP : r/googlecloud](https://www.reddit.com/r/googlecloud/comments/10soesp/infrastructure_as_code_tool_recommendation_for_gcp/)
* [Open-source alternatives to Pulumi Cloud : r/pulumi](https://www.reddit.com/r/pulumi/comments/1fp6mej/opensource_alternatives_to_pulumi_cloud/?rdt=44972)
* [Knative or openfaas? : r/kubernetes](https://www.reddit.com/r/kubernetes/comments/ar7mqr/knative_or_openfaas/)
* [Are there anyone using OpenFaaS self hosted in production? : r/OpenFaaS](https://www.reddit.com/r/OpenFaaS/comments/nopttb/are_there_anyone_using_openfaas_self_hosted_in/)
