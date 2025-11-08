---
"type:": fleet-note
"title:": 20251013132936-ISO IEC 25010 2023 Systems and software Quality Requirements and Evaluation (SQuaRE) - Product quality model
id:: 20251013132952  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-10-13T13:29:52  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---

# What is product quality model?

## Functional Suitability
### Functional Completeness

> Capability of a product to provide a set of functions that covers all the specified tasks and intended users'objectives

### Functional Correctness

> Capability of a product to provide accurate results when used by intended users
> Note 1 to entry: Precision is one of the attributes of correctness.

### Functional Appropriateness

> Capability of a product to provide functions that facilitate the accomplishment of specified tasks and objectives.
> EXAMPLE: A product provides the necessary and sufficient steps to complete a task, excluding any unnecessary steps.

## Performance Efficiency
### Time Behavior

> Capability of a product to perform its specified function under specified conditions so that the response time and throughput rates meet the requirements

### Resource Utilization

> Capability of a product to use no more than the specified amount of resources to perform its function under specified conditions

* CPU
* Memory
### Capacity

> Capability of a product to meet requirements for the maximum limits of a product parameter

* Concurrency
* Storage
* Bandwidth
## Compatibility
### Co-existence

> Capability of a product to perform its required functions efficiently while sharing a common environment and resources with other products, without detrimental impact on any other product.

Virtualization.
### Interoperability

> Capability of a product to exchange information with other products and mutually use the information that has been exchanged.

Standard communication schema without platform-lock?
## Interaction Capability
### Appropriateness Recognizability

> Capability of a product to be recognized by users as appropriate for their needs.

README

### Learnability

> Capability of a product to have specified users learn to use specified product functions within a specified amount of time.

Document, guide and help.

### Operability

> Capability of a product to have functions and attributes that make it easy to operate and control.

### User Error Protection

> Capability of a product to prevent operation errors.

### User Engagement

> Capability of a product to present functions and information in an inviting and motivating manner encouraging continued interaction

### Inclusivity

> Capability of a product to be utilised by people of various backgrounds
> Note 1 to entry: Backgrounds include (and are not limited to) people of various ages, abilities, cultures, ethnicities, languages, genders, economic situations, education, geographical locations and life situations.

### User Assistance

> Capability of a product to be used by people with the widest range of characteristics and capabilities to achieve specified goals in a specified context of use.

### Self-descriptiveness

> Capability of a product to present appropriate information, where needed by the user, to make its capabilities and use immediately obvious to the user without excessive interactions with a product or other resources.

## Reliability
### Faultlessness

> Capability of a product to perform specified functions without fault under normal operation.

### Availability

> Capability of a product to be operational and accessible when required for use.

SLO
Master-slave, Cluster

### Fault Tolerance

> Capability of a product  to operate as intended despite the presence of hardware or software faults

* Timeout
* Retry
* Rate Limit
* Circuit breaker

[服务容错模式 - 美团技术团队](https://tech.meituan.com/2016/11/11/service-fault-tolerant-pattern.html)
### Recoverability

> Capability of a product in the event of an interruption or a failure to recover the data directly affected and re-establish the desired state of the system

Backup

## Security
### Confidentiality

> Capability of a product to ensure that data are accessible only to those authorized to have access

* Encryption
* Access control
* Authorization

### Integrity

> Capability of a product to ensure that the state of its system and data are protected from unauthorized modification or deletion either by malicious action or computer error

* Hash
* Checksum
### Non-repudiation

> Capability of a product to prove that actions or events have taken place, so that the events or actions cannot be repudiated later

* Digital signatures
### Accountability

> Capability of a product to enable actions of an entity to be traced uniquely to the entity.

* Audit
### Authenticity

> Capability of a product to prove that the identity of a subject or resource is the one claimed.

* Authentication
### Resistance

> Capability of a product to sustain operations while under attack from a malicious actor.

* Vulnerability scan

## Maintainability
### Modularity

> Capability of a product to limit changes to one component from affecting other components.

### Reusability

> Capability of a product to be used as assets in more than one system, or in building other assets

### Analysability

> Capability of a product to be effectively and efficiently assessed regarding the impact of an intended change to one or more of its parts, to diagnose it for deficiencies or causes of failures, or to identify parts to be modified

* Trace
* Log
* Metrics
### Modifiability

> Capability of a product to be effectively and efficiently modified without introducing defects or degrading existing product quality.

### Testability

> Capability of a product to enable an objective and feasible test to be designed and performed to determine whether a requirement is met.

## Flexibility
### Adaptability

> Capability of a product to be effectively and efficiently adapted for or transferred to different hardware, software or other operational or usage environments.

### Scalability

> Capability of a product to handle growing or shrinking workloads or to adapt its capacity to handle variability.

### Installability

> Capability of a product to be effectively and efficiently installed successfully and/or uninstalled in a specified environment.

### Replaceability

> Capability of a product to replace another specified product for the same purpose in the same environment.

## Safety
### Operational Constraint
### Risk Identification
### Fail Safe
### Hazard Warning
### Safe Integration
# Reference
