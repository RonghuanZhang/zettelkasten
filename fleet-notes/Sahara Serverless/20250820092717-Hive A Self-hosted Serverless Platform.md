---
"type:": fleet-note
"title:": 20250820092717-Hive A Self-hosted Serverless Platform
"id:": 20250820093025
"created:": 2025-08-20T09:30:25
url:
tags:
  - fleet-note
  - project/hive
"processed:": false
"archived:": false
---
# Development Before Serverless

Before Serverless, building and running an application involved many manual steps:
- **Environment setup**: Developers had to prepare servers, install operating systems, runtimes, and libraries.
- **Application deployment**: Code needed to be packaged, uploaded, and deployed manually or through CI/CD tools.
- **Scaling**: Teams had to add or remove servers by themselves to handle traffic changes.
- **Operations**: Monitoring, logging, fixing issues, applying patches, and security updates were all managed by developers or dedicated ops teams.
- **Cost management**: Servers were often reserved for peak traffic, which meant resources stayed idle and wasted during low usage.
In short, developers had to spend time not only on writing business logic but also on managing infrastructure, which slowed down delivery.

# Development and Deployment With Serverless

With Serverless, the process becomes much simpler:
- **Focus on code only**: Developers just write business logic; they don’t need to manage servers or runtime environments.
- **Event-driven execution**: Code runs only when triggered, for example, by an HTTP request, a scheduled job, or a message queue.
- **Automatic scaling**: The platform scales up when requests increase and scales down to zero when not in use.
- **Pay-per-use**: You pay only for the actual execution time and resources, not for idle servers.
- **Built-in operations**: Logging, monitoring, and system updates are handled by the platform.

This allows developers to deliver features faster, reduce costs, and focus more on innovation instead of infrastructure.
# The History of Serverless.

* 2006
	* Zimki
* 2011
	* Parse (Acquired by Facebook)
* 2012
	* Firebase
	*   Ken Fromm proposed the concept of Serverless in his blog.
* 2014
	* Google acquired Firebase. Firebase became the complete BaaS platform of Google Cloud. 
	* AWS Lambda. It is considered the beginning of the practice of Serverless architecture.
		* pay-per-call
		* Automatic scaling
* 2016
	* IBM Cloud Functions (Based on Apache OpenWhisk)
	* Google Cloud Functions
	* Microsoft Azure Cloud Functions
* 2017
	* Tencent Serverless Cloud Function
	* Alibaba Cloud Function Compute
* 2018
	* Google Cloud Function GA
	* Zeit Release Now 2.0 （Based on AWS Lambda）
* 2019
	* [Cloud Programming Simplified: A Berkeley View on Serverless Computing](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2019/EECS-2019-3.pdf) Serverless = BaaS + FaaS
	* Google Cloud Run (Based on Knative)
* 2020
	* Zeit -> Vercel
* 2021
	* Vercel Edge Functions
* 2022
	* Vercel Edge Functions GA
	* Serverless + Edge Computing

# What is Serverless？
## Overview

Serverless lets you build and run applications without managing servers. Key benefits include:
- **Zero operations**: no provisioning, updating, or maintaining infrastructure.
- **Automatic scaling**: resources scale up or down based on demand, without user intervention.
- **Pay-per-use**: no cost when idle.
- **Business focus**: developers spend more time on logic, less on ops.

**When to use Serverless?**
* Release the resource when it is not in use to avoid paying for idle resources.
* Scaling demand is unpredictable. 
* Asynchronous, concurrent, easy to parallelize into independent units of work.
* Focus is on business logic, not infrastructure. 

**When not to use Serverless?**
* Long-running or resource-intensive tasks.
* Predictable, steady workloads.
* High performance. Low lentency.
* Complex stateful system.

There are two implementations of Serverless
* FaaS (Function as a Service)
* BaaS (Backend as a Service)

## FaaS
![FaaS(Function as a Service).png](https://images.hnzhrh.com/note/20250820094614671.png)
## BaaS
![BaaS (Backend as a Service).png](https://images.hnzhrh.com/note/20250820095652369.png)

## Use Cases for Serverless

There are many Serverless platforms and applications:
* [Google Cloud Run](https://cloud.google.com/run?hl=en)
* [AWS Lambda](https://aws.amazon.com/lambda/)
* [Serverless](https://www.serverless.com/)
* [Alibaba Cloud Serverless](https://serverless.aliyun.com/)
* [Vercel](https://vercel.com/)
### Web Applications
* **Company websites and landing pages** – deploy static or dynamic web apps quickly.
* **Backend services** – authentication, user management, and content management systems.
* **E-commerce websites** – handle unpredictable traffic with automatic scaling.
### APIs and Microservices
* **REST and GraphQL APIs** – build scalable APIs for web or mobile clients.
* **Event-driven microservices** – trigger functions on demand, reducing idle costs.
* **Internal APIs** – support integration across enterprise applications.
###  Data and Media Processing
* **File processing pipelines** – image resizing, video transcoding, or document conversion.
* **Data transformation** – ETL jobs, log processing, or analytics pre-processing.
* **Batch tasks** – run background jobs without maintaining servers.
### Automation and Integration
* **Scheduled tasks** – cron jobs such as database cleanup, email notifications, or report generation.
* **Third-party integrations** – connect SaaS applications like Slack, GitHub, or Stripe.
* **Workflow orchestration** – glue services together without custom servers.
### AI and Machine Learning
* **Inference endpoints** – run ML models for prediction, classification, or recommendation.
* **Data preprocessing** – clean and normalize data before training.
* **Event-driven AI pipelines** – trigger workflows when new data arrives.
# What is Hive？
## Overview

Hive is a self-built Serverless platform that lets you deploy applications—both services and functions—easily via the command-line interface (CLI).
- **Services**: Deploy directly using container images.
- **Functions**: Currently supported in Golang, Node.js, and Spring Boot.

Once deployed, you can access services and invoke functions through standard HTTP requests.

A typical function workflow looks like this:  
Initialize the function → Write the code → Push to your Git repository → Run `apply & deploy` → Access it. Done.

![iShot_2025-08-15_09.09.36.gif](https://images.hnzhrh.com/note/20250820142016707.gif)
## Core concepts

* Service refers to the microservice. Support multiple endpoints.
* Function, code fragment. Only one entrance. Focus on one feature.

![[Hive Business Architecture.excalidraw]]
## What’s Next for the Platform?
- **New Features**
    - Deploy services directly from source code (no Dockerfile required).
    - Support additional programming languages for functions.
    - Enable multi-version services and functions.
    - And more to come...
- **Feature Enhancements**
    - Provide more deployment parameters for services and functions.
    - Improve the CLI experience with richer feedback and guidance.
    - And more to come...
- **Core Improvements**
    - Strengthen isolation, reliability, maintainability, security, and observability
- **Exploration & Research**
    - Explore IaaS (Intelligence as a Service). If you have requirements or ideas, feel free to reach out.
	    - Build AI functions as marketable assets.
	    - Provide AI-focused BaaS (Backend as a Service), such as vector databases.
## How Hive Works? 
### System Architecture

![[Hive Technology Architecture.excalidraw]]
### Key Components
- **Service & Function Layer**  
    Built on top of Knative, Hive provides custom abstractions for both _Service_ and _Function_, enabling developers to deploy workloads in a serverless manner.
- **Application & Orchestration Layer**  
    The application layer is implemented in **Java**, while the orchestration layer is written in **Golang**. Together, they coordinate the build and deployment process efficiently.
- **Multi-Tenancy Support**  
    Hive is designed with tenant isolation, ensuring secure resource sharing and scalable management for multiple users.
- **Observability**  
    Logging, monitoring, and alerting are integrated into the platform to provide visibility into system health and application performance.
# Reference
* [Hive Knative Pre-Research](https://saharalabs.atlassian.net/wiki/spaces/H/pages/366477313/Hive+Knative+Pre-Research)
* [Home - Knative](https://knative.dev/docs/)
* [Kubernetes](https://kubernetes.io/)
* [Hive Technology Design Document](https://saharalabs.atlassian.net/wiki/spaces/H/pages/374046728/Hive+Technology+Design+Document+TBD)
* [CNCF Landscape](https://landscape.cncf.io/?group=serverless)
* [美团Serverless平台Nest的探索与实践 - 美团技术团队](https://tech.meituan.com/2021/04/21/nest-serverless.html)