---
type: source-note
title: Serverless 选型：深度解读 Serverless 架构及平台选择
id: 20250511200537
created: 2025-05-11T20:28:37
source:
  - web
url: https://zhuanlan.zhihu.com/p/141217056
tags:
  - source-note
  - cloud-native/serverless
processed: false
archived: false
---
![Serverless 选型：深度解读 Serverless 架构及平台选择](https://picx.zhimg.com/70/v2-05a6a81d81e60a712f4f39cf02a598a4_1440w.avis?source=172ae18b&biz_tag=Post)

Serverless 选型：深度解读 Serverless 架构及平台选择

作者 | 悟鹏 阿里巴巴技术专家

> **导读：** 本文尝试以日常开发流程为起点，分析 [开发者](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E5%BC%80%E5%8F%91%E8%80%85&zhida_source=entity) 在每个阶段要面对的问题，然后组合解决方案，提炼面向 Serverless 的开发模型，并与业界提出的 Serverless 产品形态做对应，为开发者采用 Serverless 架构和服务提供参考。

近两年来，Serverless 概念在开发者中交流的越来越多，主题分享呈现爆发趋势，如在云原生领域颇具影响力 KubeCon&CloudNativeCon 会议中，关于 Serverless 的主题，2018 年有 20 个，到 2019 年增长至 35 个。

在 Serverless 产品层面，从最早的 AWS Lambda，到 Azure Functions、Goolge Functions、Google CloudRun，再到国内阿里云 Serverless Kubernetes、Serverless 应用引擎、函数计算等，面向计算的 Serverless 云上基础设施越来越丰富。

新概念、新产品的产生不是凭空出现，它们诞生之初要解决的是当前问题。随着实践者对问题域的理解越来越清晰和深刻，问题的处理方法也会逐步迭代，更接近问题本质的解决方案也会出现。若不从 [问题域](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=2&q=%E9%97%AE%E9%A2%98%E5%9F%9F&zhida_source=entity) 出发来理解解决方案，容易陷入两个极端，即「它能解决一切问题」和「它太超前了，理解不了」。

## 从日常迭代看 Serverless

  

![](https://pic4.zhimg.com/v2-6189c0e540a6e7be3a0d78af92382c6b_1440w.jpg)

  
图 1

上图是一个常用的项目 [迭代模型](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E8%BF%AD%E4%BB%A3%E6%A8%A1%E5%9E%8B&zhida_source=entity) ，其目标是满足客户需求。在这个模型中，项目组通过 **被动迭代** 满足客户提出的需求，同时逐步深刻理解客户需求的本质，通过 **主动迭代** 和客户一起采用更好的方案或从根源解决面对的问题。每次的需求反馈会加深对客户需求的理解，提供更满足需求的服务。每次的 bug 反馈会加深对处理方案的理解，提供更稳定的服务。

在模型启动后，日常的核心问题就集中在了 **如何加速 [迭代](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=6&q=%E8%BF%AD%E4%BB%A3&zhida_source=entity)** 。

如果想要解决迭代加速的问题，就需要了解有哪些制约因素，有的放矢。下述是从开发视角看到的开发模型：

  

![](https://pic2.zhimg.com/v2-baab560f517f2d110533a6d8cf3c97e1_1440w.jpg)

  
图 2

虽然在实际应用中会采用不同的开发语言和架构，但在每个阶段均有通用的问题，如：

  

![](https://pic1.zhimg.com/v2-d6cf00965ed5dd70bbcf94a86bfd9f56_1440w.jpg)

  
图3

除了要解决上述通用问题，还需要提供标准化的方案，降低开发者的学习和使用成本，缩短从想法到上线的时间。

若将上述过程中不同阶段花费的时间做个分析，在项目整个生命周期中会发现：

- 部署&运维 占用的时间和精力，会远大于开发&测试
- 通用逻辑占用的时间和精力，会接近甚至超过业务逻辑

为了加速迭代，需要依次解决占用时间和精力多的部分，如图 4 所示：

  

![](https://pic3.zhimg.com/v2-338be7e413788e9c71e3c6cb94e46a54_1440w.jpg)

  
图 4

从左至右，通过下放不同层次的运维工作，降低「部署&运维」成本。在降低了运维工作成本后，在「通用逻辑」层面降低成本。二者结合起来，在迭代过程中更深入聚焦业务。该过程也是从 Cloud Hosting 到 Cloud Native 的过程，充分享受云原生带来的 [技术红利](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E6%8A%80%E6%9C%AF%E7%BA%A2%E5%88%A9&zhida_source=entity) 。

由于软件设计架构和部署架构与当时环境耦合度高，面对新的理念和服务、产品，存量应用迭代过程中采用的技术需要有相应调整，即开发和部署方式需要有一定的改造。对于新应用进行开发和部署时，应用新的理念有一定的学习和实践成本。

故上述过程不能一蹴而就，需要根据业务当前的痛点优先级来选择匹配的服务和产品，并根据未来的规划提前进行技术预研，在不同的阶段选择适合的服务和产品。

## Serverless 简介

维基百科对于 Serverless 有较为完备的 [定义](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Serverless_computing):

> Serverless computing is a cloud computing [execution](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=execution&zhida_source=entity) model in which the cloud provider runs the server, and dynamically manages the allocation of machine resources. Pricing is based on the actual amount of resources consumed by an application, rather than on pre-purchased units of capacity.  
> [无服务器计算](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E6%97%A0%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%AE%A1%E7%AE%97&zhida_source=entity) 是一种云计算执行模型，云厂商提供程序运行的服务器，并动态管理机器资源的分配。云厂商基于应用程序消耗的实际资源量进行定价，而不是用户预先购买的容量。

在这种计算模型下，会给用户带来如下收益：  

> Serverless computing can simplify the process of deploying code into production. Scaling, capacity planning and maintenance operations may be hidden from the developer or operator. Serverless code can be used in conjunction with code deployed in traditional styles, such as microservices. Alternatively, applications can be written to be purely serverless and use no provisioned servers at all.  
> 无服务器计算可以简化代码部署到生产环境的过程，且扩缩容、容量规划和运维操作可以做到对开发人员透明化。无服务器代码可以与以传统方式（如 [微服务](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E5%BE%AE%E6%9C%8D%E5%8A%A1&zhida_source=entity) ）部署的代码结合使用，或者，开发者可以按照无服务器计算的模式编写应用，完全不用提前 [配置服务器](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E9%85%8D%E7%BD%AE%E6%9C%8D%E5%8A%A1%E5%99%A8&zhida_source=entity).

概念本质上是对问题域的抽象，是对问题 [域特征](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E5%9F%9F%E7%89%B9%E5%BE%81&zhida_source=entity) 的总结。通过特征来理解概念，可以避免注意力集中在文字描述而非概念的价值本身。

站在用户角度，我们可以抽象出 Serverless 的如下特征：

- 免运维 (服务器运维、容量管理、弹性伸缩等)
- 按资源的使用量付费

在一定规模的公司中，若严格区分开发和运维的角色，这种计算形态其实是已经存在的，并非全新的事物。但目前的技术趋势，是期望借助云的规模和技术红利优势，通过上云来降低业务在技术侧的成本，并通过技术红利反哺业务。故业界对于 Serverless 的讨论，注意力是集中在云上的服务和产品所体现的 Serverless 能力。

## Serverless 开发模型

Martin Fowler 的 [这篇文章](https://link.zhihu.com/?target=https%3A//martinfowler.com/articles/serverless.html) 站在架构的角度，对 Serverless 开发模型做了充分的阐述，这里做个简单的总结，核心围绕三点：

- Event-driven 开发模型
- 自动弹性伸缩
- OpenAPI

Serverless 开发采用 **[Event-driven 模型](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Event-driven_architecture)** ，围绕 HTTP/HTTPS 请求、时间、消息等 Event 的生产和响应进行 [架构设计](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1&zhida_source=entity) 。在这样的模型中，Event 的生产、处理流程是核心，通过 Event 驱动整个服务流程，注意力集中在整个处理流程。对业务理解越深刻，Event 类型和业务会越匹配，技术和业务的相互促进作用会越有效。

Event-driven 模型，使得 **服务常驻** 这种理念从必选项转变为可选项，可以更好应对业务请求量的变化，如自动弹性伸缩。同时服务非常驻，可以降低所需的资源成本和维护成本，加速项目迭代。

通过 [文章](https://link.zhihu.com/?target=https%3A//martinfowler.com/articles/serverless.html) 的两幅图可以更直观理解：

  

![](https://picx.zhimg.com/v2-0c0ca9b3ce98e93a681d47bd5598fd2f_1440w.jpg)

  
图 5

图 5 是当前常见的开发模型，Click Processor 服务是个常驻服务，响应来自用户的所有点击请求。生产环境中，通常会多实例部署， **常驻** 是个关键特征，日常的运维重点在确保常驻服务的稳定性方面。

  

![](https://pic1.zhimg.com/v2-51f3e20880811b7c52aacdebe2fc0cc6_1440w.jpg)

  
图 6

图 6 是 Event-driven 开发模型，关注重心前移，集中在 Event 的产生和响应方面，响应服务是否常驻是个可选项。

Serverless 在概念上与 PaaS (Platform as a Service)、CaaS (Container as a Service) 的区别，重点是在是否将 **自动弹性伸缩** 作为概念诞生之初的核心特征。

结合 Event-driven 的开发模型，Serverless 场景中自动弹性伸缩需要对开发者透明度加深，开发者开发过程对处理能力的关注重心从静态转为动态，更好应对上线后业务请求量的不确定性。

在开发方面，交付时可以采用镜像，也可以采用语言层面的打包 (如 Java 中的 war/jar) ，由平台负责运行时相关的工作。还可以更进一步，采用 FaaS 的理念，依托于平台或标准化 FaaS 解决方案，只提供业务 [逻辑函数](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E9%80%BB%E8%BE%91%E5%87%BD%E6%95%B0&zhida_source=entity) ，由平台负责请求入口、请求调用和自动弹性伸缩等运行时事宜。

不论哪种交付方式，在云上均可以使用 [BaaS](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Mobile_backend_as_a_service) 的理念，将部分逻辑通过云平台或第三方的 OpenAPI 实现，如 [权限管理](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E6%9D%83%E9%99%90%E7%AE%A1%E7%90%86&zhida_source=entity) 、 [中间件](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E4%B8%AD%E9%97%B4%E4%BB%B6&zhida_source=entity) 管理等，开发过程中注意力更加聚焦在业务层面。

## Serverless 服务模型

Serverless 服务模型关注云厂商对于 Serverless 计算形态的支持，不同的服务和产品形态主要差异点主要集中在对 Serverless 特征的理解和满足程度方面：

- 免运维 (服务器运维、容量管理、 [弹性伸缩](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=7&q=%E5%BC%B9%E6%80%A7%E4%BC%B8%E7%BC%A9&zhida_source=entity) 等)
- 按资源的使用量付费

在免运维维度，最基本的是免去服务器运维成本，开发者可以按量申请资源。在容量管理、弹性伸缩、 [流量管理](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E6%B5%81%E9%87%8F%E7%AE%A1%E7%90%86&zhida_source=entity) 、日志/监控/告警等常规的运维层面，不同的服务和产品会根据自身定位、目标客户特征等，有侧重采用适合的方式来满足。

在计费形态方面，云厂商一方面会根据自身定位确定收费维度，如资源、请求量等，一方面也会根据当前的技术能力确定收费的 [粒度](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E7%B2%92%E5%BA%A6&zhida_source=entity) 。

通过上述分析可知， [云厂商](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=5&q=%E4%BA%91%E5%8E%82%E5%95%86&zhida_source=entity) 不同 Serverless 服务模型不是静态的，会伴随产品定位、目标客户特征、技术能力等持续迭代，和客户共同成长。

Serverless 服务模型需要满足实际需求，再回到图 4，云厂商的 Serverless 服务模型可以分为如下几类：

- 资源实例平台
- [调度平台](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E8%B0%83%E5%BA%A6%E5%B9%B3%E5%8F%B0&zhida_source=entity)
- 应用管理平台
- 业务逻辑管理平台

综合起来，即：

  

![](https://pic1.zhimg.com/v2-572717d0e0ee9aa5a6509bb325fb8b30_1440w.jpg)

  
图7

## 业界 Serverless 产品

目前国内外云厂商均提供有不同维度的 Serverless 产品，如下做个简单的总结：

  

![](https://pic2.zhimg.com/v2-1408887f7890427069cf78205b3cf5bf_1440w.jpg)

  
图 8

## 资源实例平台

国外 AWS Fargate / Azure ACI、国内阿里云 ECI / 华为 CCI 影响力较大，对用户提供容器组服务。 [容器组](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=2&q=%E5%AE%B9%E5%99%A8%E7%BB%84&zhida_source=entity) 作为一个整体，提供类似 Kubernetes 中 Pod 的概念。用户可以通过 OpenAPI 调用直接创建容器组，不用在部署服务前进行服务器的购买、配置等工作，下放资源相关的容量管理运维工作。用户可以将资源实例平台作为容量足够大的资源池使用，在容器组级别进行 [细粒度](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E7%BB%86%E7%B2%92%E5%BA%A6&zhida_source=entity) 的资源申请，配合动态扩缩容进行应用级别的容量管理。

一般在生产环境中，用户通常不会直接使用此类资源管理服务，而是借助应用编排服务，将这类服务透明化，关注重点放在应用编排维度。

## 调度平台

Kubernetes 是容器调度的事实标准，国外 AWS EKS、国内阿里云 Serverless Kubernetes 等一方面托管 Kubernetes Master 组件，一方面借助资源管理服务，如 VirtualKubelet + AWS Fargate 或 VirtualKubelet + 阿里云 ECI，提供 Kubernetes Node 层服务。

对于期望直接使用 Kubernetes 能力，同时希望低成本运维 Kubernetes、不保有资源池的用户，此类产品比较契合需求。

## 应用管理平台

国外 Google GAE / CloudRun、国内阿里云 Serverless 应用引擎等进一步将运维工作服务化，如发布管理 (打包/灰度/分批/回滚/ [版本控制](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6&zhida_source=entity) 等)、日志/监控/告警、流量管理、弹性伸缩等，用户可以进一步专注在业务需求，低成本运维。

对于期望零成本改造存量应用、低学习成本使用，又期望最大限度减少运维工作的用户，此类平台与需求匹配度较高。但由于应用管理层面的运维工作在业界暂无标准化方案，不同的项目会存在个性化需求，故采用此类产品过程中需要加强沟通，不断向平台反馈，通过共建的方式磨合 Serverless 平台和自身业务。

## 业务逻辑管理平台

国外 AWS Lambda / Azure Functions / Google Functions、国内阿里云函数计算 / 腾讯云云函数 / 华为函数工作流等在应用管理的基础上，进一步将开发过程中的通用逻辑透明化，用户仅用关心业务逻辑的实现。这个过程可以类比开发过程中的单元测试编写过程，输入、输出是通用的，仅在处理逻辑存在差异。这类 Serverless 产品也是业界讨论最多的形态，代表业界对于理想的开发流程的抽象，可以进一步加快迭代流程，缩短想法到上线的时间。这类 Serverless 产品与云平台其他类型的产品集成更紧密，以 [BaaS](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Mobile_backend_as_a_service) 形态使用云平台的服务实现通用逻辑，如存储、缓存等，对 [云平台](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=4&q=%E4%BA%91%E5%B9%B3%E5%8F%B0&zhida_source=entity) 产品丰富度有一定隐性需求。

处理过程对外部依赖较少或偏计算类的场景，如前端、多媒体处理处理等，采用此类 Serverless 产品学习和使用成本相对低，易于上手。随着服务、组件的抽象程度越来越高，会有越来越多的业务场景适用，用户的运维工作会更为透明化，同时开发过程中可以直接享受到业界的 [最佳实践](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5&zhida_source=entity) ，服务的稳定性、性能、吞吐等方面借助平台的能力做到最大化。

## 选型

综上，用户在进行 Serverless 产品选型时，需要先整理当前业务技术所处的阶段和痛点，确定对云上方案的需求，然后再根据云厂商的产品形态做对应，选择适合当前阶段的服务和云产品。

该对应关系重点是了解云产品定位是否可以长期满足业务需求，如：

- 业务技术目前所处的阶段是否与云产品定位匹配
- 业务快速迭代是否会受限于云产品自身的发展
- 云产品的稳定如何
- 云产品是否可以持续为业务带来技术红利

同时还需要了解云产品是否可以伴随业务发展，重点是业务对技术的需求中，哪些是云产品层面由于定位带来的限制，哪些是当前云产品的技术实现带来的限制。

若是云产品定位带来的限制，那么就需要考虑使用和业务需求定位更匹配的云产品。若是当前技术实现的限制，那么有机会和云产品共同成长，及时给云产品反馈，使得云产品可以更好满足自身的业务需求。

除此之外，业务层面还需关注云厂商自身服务类型的丰富性，云厂商自身服务越丰富，规模越大，越会产生 [规模效应](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E8%A7%84%E6%A8%A1%E6%95%88%E5%BA%94&zhida_source=entity) ，进而给业务带来更丰富的技术红利和成本优势。

幸运的是，云产品通常都会有丰富的文档，也有相应的用户群，可以直面产品经理和研发，及时反馈需求，以共建的理念协同发展。

## 小结

Serverless 本质上是一个问题域，将研发流程中非业务核心却影响业务迭代的问题 [抽象化](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E6%8A%BD%E8%B1%A1%E5%8C%96&zhida_source=entity) ，并提出相应的解决方案。该概念不是突然产生的，大家或多或少已经将其理念应用到日常的工作中 ，只是伴随着 [云计算](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=2&q=%E4%BA%91%E8%AE%A1%E7%AE%97&zhida_source=entity) 浪潮，云上的 Serverless 服务和产品更系统、更具有竞争力，可以基于规模优势和丰富的产品线，面对问题域持续提供更满足业务需求的服务。

Serverless 理念不仅在 [中心化](https://zhida.zhihu.com/search?content_id=119513176&content_type=Article&match_order=1&q=%E4%B8%AD%E5%BF%83%E5%8C%96&zhida_source=entity) 的云端蓬勃发展，目前也逐步在边缘端发展，使得服务的运行更加广泛化，更好满足业务自身的客户，提供更低延时、稳定的服务。

本篇文章尝试从项目、开发的日常流程出发，协助读者从日常实践角度来理解 Serverless 概念，根据所处的阶段选择适合的 Serverless 服务和产品。同时作者本人在阿里云 Serverless 应用引擎中负责底层研发工作，尝试从云产品内部的视角，传递云产品和用户共建的观念，通过协同更好传递和创造价值。

## References

- [维基百科对 Serverless 较为完备的定义](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Serverless_computing)
- [Serverless Architectures](https://link.zhihu.com/?target=https%3A//martinfowler.com/articles/serverless.html)
- [Event-driven 模型](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Event-driven_architecture)
- [BaaS](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Mobile_backend_as_a_service)
- [从 DevOps 到 NoOps，Serverless 技术的落地方式探讨](https://link.zhihu.com/?target=https%3A//yq.aliyun.com/articles/754236)
- [阿里云弹性容器实例 ECI](https://link.zhihu.com/?target=https%3A//www.aliyun.com/product/eci)
- [阿里云容器服务 ACK](https://link.zhihu.com/?target=https%3A//www.aliyun.com/product/kubernetes)
- [阿里云 Serverless Kubernetes（ASK）](https://link.zhihu.com/?target=https%3A//help.aliyun.com/document_detail/86366.html)
- [阿里云 Serverless 应用引擎 (SAE)](https://link.zhihu.com/?target=https%3A//www.aliyun.com/product/sae)
- [阿里云函数计算](https://link.zhihu.com/?target=https%3A//www.aliyun.com/product/fc)
- [阿里云 Serverless 工作流](https://link.zhihu.com/?target=https%3A//www.aliyun.com/product/fnf)
- [Cloud Programming Simplified: A Berkeley View on Serverless Computing](https://link.zhihu.com/?target=https%3A//www2.eecs.berkeley.edu/Pubs/TechRpts/2019/EECS-2019-3.pdf%3Fspm%3Data.13261165.0.0.24c275f8HV8b9B%26file%3DEECS-2019-3.pdf)

## 课程推荐

为了更多开发者能够享受到 Serverless 带来的红利，这一次，我们集结了 10+ 位阿里巴巴 Serverless 领域技术专家，打造出最适合开发者入门的 Serverless 公开课，让你即学即用，轻松拥抱云计算的新范式——Serverless。

点击即可免费观看课程： [developer.aliyun.com/le](https://link.zhihu.com/?target=https%3A//developer.aliyun.com/learning/roadmap/serverless)

> “ [阿里巴巴云原生](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzUzNzYxNjAzMg%3D%3D%26mid%3D2247490471%26idx%3D1%26sn%3D10dbb21fc15076626b34e1a2bec5d6dc%26chksm%3Dfae51068cd92997ef0eb8da091998bfdd45c86b85161f29917531d81adb8ce1d11b778060cfb%26token%3D850285981%26lang%3Dzh_CN%23rd) 关注微服务、Serverless、容器、Service Mesh 等技术领域、聚焦云原生流行技术趋势、云原生大规模的落地实践，做最懂云原生开发者的公众号。”

[所属专栏 · 2021-04-21 14:33 更新](https://zhuanlan.zhihu.com/c_1289174124607795200)

[![](https://picx.zhimg.com/v2-8802e691f829532855526b2c6de6833b_720w.jpg?source=172ae18b)](https://zhuanlan.zhihu.com/c_1289174124607795200)

[Serverless 观察](https://zhuanlan.zhihu.com/c_1289174124607795200)

[

GrumpyAmber

67 篇内容 · 414 赞同

](https://zhuanlan.zhihu.com/c_1289174124607795200)

[

最热内容 ·

\[译\]简化云编程：伯克利关于Serverless计算的观点

](https://zhuanlan.zhihu.com/c_1289174124607795200)

发布于 2020-05-15 18:43[前端架构](https://www.zhihu.com/topic/19710481)[Serverless](https://www.zhihu.com/topic/20086226)[云计算](https://www.zhihu.com/topic/19550358)