---
"type:": technology-design-document
"title:": 20250521132739-Hive Technology Design Doc
"id:": 20250521132750
"created:": 2025-05-21T13:27:50
tags:
  - technology-design-document
  - project/hive
"processed:": false
"archived:": false
---

# 1 Document Information

| Version | Author        | Change Description | Date       |
| ------- | ------------- | ------------------ | ---------- |
| 1.0.0   | RonghuanZhang | Initial draft      | 2025-05-21 |

# 2 Background & Motivation

> 1. Brief business context
> 2. Why this system/module is needed
> 3. Existing challenges or goals

# 3 Required Technical Knowledge

## 3.1 What's the Serverless?

Serverless is a cloud-native development model that allows developers to focus on the logic without or with less on infrastructure maintenance. 
* Service
* Function
## 3.2 What's the difference between Service and Function in Serverless?

A Serverless Function (often called Function-as-a-Service, or FaaS) is a small, single-purpose piece of code that runs in the cloud without you managing any servers. 

Features：
* Stateless
* Automatic Scaling
* Short-Lived
* Focus on your code only (You need to support the Dockerfile for Service.)
* Support Event Driven (Service support HTTP originally.)
# 4 Ubiquitous language

| Bounded Context      | English                                   | Chinese    | Explaination                                      |
| -------------------- | ----------------------------------------- | ---------- | ------------------------------------------------- |
| Application          | Application                               | 应用         | Exposed to users.                                 |
|                      | Application Type                          | 应用类型       | Service and Function. Service only for now.       |
|                      | Application Source Provider Configuration | 应用源提供方配置   | Image、GitHub and Source code. Image only for now. |
|                      | Application Deployment Configuration      | 应用部署配置     | Env、Secrect. Env only for now.                    |
| Knative Ochestration | Knative Service                           | Knative 服务 |                                                   |
|                      |                                           |            |                                                   |

# 5 Event Storming

[[Hive Event Storming.excalidraw]]

# 6 Requirements Overview

## 6.1 Functional Requirements
* As a user, I want to apply a token by providing my user id, So that I can get the access to use the Hive CLI to deploy my application.
* As a user, I want to create a new application by providing its name and type, So that the system can initialize the application.
* As a user, I want to configure the application's source provider using an image, So that the system knows what to deploy.
* As a user, I want to configure the application's deployment, including enviroment variables and secrets, So that the application can be deployed with the required settings.
* As a user, I want to deploy the configured application, So that I can get the public endpoint to access it externally.
* As a user, I want to view the application details, So that I can see its name, type, source provider info, deployment info, public endpoint and the alived replicas.

## 6.2 Non-functional Requirements

> 1. Performance
> 2. Scalability
> 3. Security
> 4. Maintainability
> 5. Availability
> 6. Observability

# 7 High-Level Architecture

> System architecture diagram
> Module break-down
> Domain model
> Technology stack

## 7.1 Diagrams

[[Hive Technology Architecture.excalidraw]]

[[Hive Control and Request Flow.excalidraw]]

## 7.2 Module Breakdown

| Module                     | Technology Stack                          | Remark                                                 |
| -------------------------- | ----------------------------------------- | ------------------------------------------------------ |
| Hive API Gateway           | Spring Cloud Gateway                      |                                                        |
| Hive Application           | Java + Spring Boot + MyBatis Plus + MySQL | JDK 21<br>Spring Boot 3.4.0<br>MySQL 8.0               |
| Hive Knative Orchestration | Golang                                    | Golang is better suited for cloud-native development.  |
| Hive Knative Gateway       | Kourier                                   | The Kourier is enough for now. The Istio is too heavy. |
| Hive Knative               | 1. Serving<br>2. Network                  | Pluggable components-based installation                |
| Kubernets                  | GKE                                       |                                                        |

## 7.3 Domain Model

# 8 Module Design

## 8.1 Hive Client Module

### 8.1.1 Responsibility

### 8.1.2 Sequence Diagram

### 8.1.3 Key Configuration

## 8.2 Hive API Gateway Module

### 8.2.1 Responsibility

### 8.2.2 Sequence Diagram

### 8.2.3 Key Configuration

## 8.3 Hive Application Module

### 8.3.1 Responsibility

### 8.3.2 Domain Diagram

### 8.3.3 Sequence Diagram

## 8.4 Hive Knative Orchestration Module

### 8.4.1 Responsibility



### 8.4.2 Domain Diagram

### 8.4.3 Sequence Diagram

## 8.5 Hive Knative Gateway Module

### 8.5.1 Responsibility

### 8.5.2 Sequence Diagram

### 8.5.3 Key Configuration

## 8.6 Hive Knative

### 8.6.1 Responsibility

### 8.6.2 Domain Diagram

### 8.6.3 Sequence Diagram

# 9 Data Design

> Database schema (tables and relationships)
> Entity definitions
> Important fields and constraints
> ER diagram

## 9.1 user_white_list

## 9.2 application

# 10 API Design

> List of REST APIs
> Request/Response Example
> Status codes

# 11 Error Handling & Logging

> 1. Error scenarios and handling strategy
> 2. Logging levels (info, warn, error)
> 3. Structured logging if applicable

# 12 Performance & Scalability

> 1. Concurrency handling
> 2. Caching strategies
> 3. Rate limiting / Circuit breaking
> 4. Horizontal scaling considerations

# 13 Security Considerations

> 1. Authentication/Authorization (e.g., JWT, OAuth2)
> 2. Input validation and sanitization
> 3. Sensitive data protection
> 4. Access control and auditing

## 13.1 User Applied and Auth

The user received an auth token from the Hive administrator and is required to use it to access the Hive CLI.

## 13.2 Resource Isolation

[[Hive Resource Isolation.excalidraw]]


# 14 Deployment & Operations

> 1. Deployment architecture
> 2. CI/CD pipeline overview
> 3. Configuration management (e.g., Spring Config, Nacos)
> 4. Monitoring & alerting (e.g., Prometheus, Grafana)

# 15 Risks & Mitigations

> 1. Known technical or business risks
> 2. Proposed mitigations or fallback plans

## 15.1 Namespace-based Isolation May Introduce Performance Risks



# 16 Reference
* [Project Hive - Infrastructure for Sahara AI Applications Product Requirements Document](https://docs.google.com/document/d/1jeK2VpcEpJqBjvA13-dVi0smqnqJ1pgoJhDdRJnjG40/edit?usp=sharing)
* [[20250509144358-Serverless OpenFaaS And Alternatives Research]]
