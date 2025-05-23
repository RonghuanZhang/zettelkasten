---
type: source-note
title: Best Practices for Kubernetes Namespaces
id: 20250522110512
created: 2025-05-22T11:02:12
source:
  - web
url: https://www.appvia.io/blog/best-practices-for-kubernetes-namespaces
tags:
  - source-note
  - kubernetes/namespace
processed: false
archived: false
---
![](https://cdn.prod.website-files.com/6527fe8ad7301efb15574cc7/65535f03a0b3a611c69e2a04_1.Best-Practices-for-Kubernetes-Namespaces-2048x1159.png)

[Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) are a vital feature of Kubernetes. They allow you to separate uniquely named resources into logical groups, though names don’t need to be unique between different namespaces. Namespaces can enforce separation between different deployment environments, as well as divide a cluster’s resources between multiple users and groups of users using [resource quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/).  
[命名空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) 是 Kubernetes 的一项重要功能。它允许您将具有唯一名称的资源分成逻辑组，尽管不同命名空间之间的名称不必唯一。命名空间可以强制隔离不同的部署环境，并使用 [资源配额](https://kubernetes.io/docs/concepts/policy/resource-quotas/) 将集群资源分配给多个用户和用户组。

There are multiple benefits to using namespaces, but using them improperly can also cause difficulties with your Kubernetes projects. That’s why it’s so important to understand how to use namespaces well. In this guide, you’ll learn some best practices to follow for using namespaces.  
使用命名空间有很多好处，但使用不当也会给 Kubernetes 项目带来麻烦。正因如此，了解如何正确使用命名空间至关重要。在本指南中，您将学习一些使用命名空间的最佳实践。

##### KEY TAKEAWAYS 关键要点

- <sup>Namespaces in Kubernetes organise resources and users for efficient management.</sup>  
	Kubernetes 中的命名空间组织资源和用户，以实现高效管理。
- <sup>Follow best practices for namespace usage, including clear naming and RBAC.</sup>  
	遵循命名空间使用的最佳实践，包括清晰的命名和 RBAC。
- <sup>Use multiple namespaces judiciously to avoid inefficiencies.</sup>  
	明智地使用多个命名空间以避免效率低下。
- <sup>Implement resource quotas for fair resource distribution.</sup>  
	实施资源配额，实现资源公平分配。
- <sup>Consider additional clusters for enhanced isolation when needed.</sup>  
	在需要时考虑增加集群以增强隔离。
- <sup>Products like </sup> [<sup>Wayfinder</sup>](https://www.appvia.io/features/workload-isolation) <sup>can help you easily organise and isolate your resources.</sup>  
	[<sup>Wayfinder</sup>](https://www.appvia.io/features/workload-isolation) <sup>等产品</sup> <sup>可以帮助您轻松组织和隔离您的资源。</sup>

### About the List 关于名单

Keep in mind that the following list isn’t exhaustive or objective. Best practices are generally a good place to start, especially if you’re new to the subject matter, but they’re rarely unbreakable rules. There will always be situations in which it makes sense to try something else.  
请记住，以下列表并非详尽无遗或客观。最佳实践通常是一个不错的起点，尤其是在您刚接触该主题时，但它们很少是不可打破的规则。总会有一些情况需要尝试其他方法。

With that in mind, here are some recommended best practices to follow.  
考虑到这一点，这里推荐一些值得遵循的最佳实践。

### Use Semantic, Scalable Names使用语义化、可扩展的名称

Names are important, but it’s [widely acknowledged](https://hilton.org.uk/blog/why-naming-things-is-hard) that naming things well is challenging, especially in the context of computer science. Kubernetes namespaces are no different. It’s too easy to give them bad names, inconsistent names, or names that don’t assist with clear communication.  
名称很重要，但 [众所周知](https://hilton.org.uk/blog/why-naming-things-is-hard) ，命名一个好的名称并非易事，尤其是在计算机科学领域。Kubernetes 命名空间也不例外。很容易给它们起一些不好听的名字、不一致的名字，或者一些不利于清晰沟通的名字。

When you only have one namespace, you might be tempted to name it after whatever application will live within it. For example, if you have a bookstore application, you might call the namespace bookstore. When you create more namespaces for deployment environments like develop and testing, though, you’ll have bookstore, bookstore-develop, and bookstore-testing. This can quickly expand as you introduce more environments, more applications, or more use cases for your namespaces.  
当您只有一个命名空间时，您可能会倾向于用其中运行的应用程序来命名它。例如，如果您有一个书店应用程序，您可能会将命名空间命名为 bookstore。但是，当您为开发和测试等部署环境创建更多命名空间时，您将拥有 bookstore、bookstore-develop 和 bookstore-testing。随着您为命名空间引入更多环境、更多应用程序或更多用例，命名空间的规模可能会迅速扩大。

You need to come up with and stick to a scalable naming convention. If you create production, develop, and testing to represent your deployment environments, all applications will go into one of these namespaces instead of different namespaces per application.  
您需要制定并遵循可扩展的命名约定。如果您创建生产、开发和测试环境来代表您的部署环境，则所有应用程序都将进入其中一个命名空间，而不是每个应用程序使用不同的命名空间。

You can also leverage other Kubernetes features like [labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) to distinguish between resources within a given namespace. If you label your resources with their respective application names, versions, and other necessary metadata, your namespaces stay clean and scalable while still allowing you to filter and search for particular resources as needed.  
您还可以利用其他 Kubernetes 功能（例如 [标签）](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) 来区分给定命名空间内的资源。如果您使用各自的应用程序名称、版本和其他必要的元数据标记资源，您的命名空间将保持整洁且可扩展，同时仍允许您根据需要筛选和搜索特定资源。

### Don’t Put Everything in the Default Namespace不要把所有东西都放在默认命名空间中

Kubernetes comes with a few namespaces out of the box. One of these is the aptly named default namespace, where Kubernetes places resources when another one isn’t specified. Try to avoid using this namespace much, if at all. It makes much more sense to use semantically named namespaces in production and other live environments than to just dump all of your resources into default.  
Kubernetes 自带了一些命名空间。其中一个是名为 default 的命名空间，当未指定其他命名空间时，Kubernetes 会将资源放置在该命名空间中。尽量避免使用此命名空间。在生产环境和其他实时环境中，使用语义命名的命名空间比将所有资源都放入 default 命名空间更有意义。

You can easily make assumptions about what developers will be doing with their machines, but your assumptions can just as easily be wrong. Consider a scenario where a developer creates a postgres pod in their default namespace, possibly for personal projects or experiments. This resource could conflict with your application if deployed alongside it. If your application needs a pod with the same name, being in a different namespace will avoid this conflict.  
您可以轻松地假设开发人员将如何使用他们的机器，但您的假设也很容易出错。设想这样一种场景：一位开发人员在其默认命名空间中创建了一个 Postgres Pod，可能用于个人项目或实验。如果将此资源与您的应用程序一起部署，可能会发生冲突。如果您的应用程序需要同名的 Pod，那么将其部署在不同的命名空间中可以避免这种冲突。

It’s a good idea to treat all environments, whether local or remote, with the same care when creating namespaces, because this helps ensure idempotency in your deployment builds.  
在创建命名空间时，最好以同样的谨慎对待所有环境（无论是本地还是远程），因为这有助于确保部署构建中的幂等性。

### Leverage Role-Based Access Control利用基于角色的访问控制

Kubernetes offers the ability to enforce [role-based access control (RBAC) authorisation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) as a means of controlling which users can access resources. RBAC can work with either the ClusterRole resource type, which is scoped to the entire cluster, or the Role resource type, which is scoped to a specific namespace. This means that you can limit the resources a user is allowed to access, either globally or within a particular namespace.  
Kubernetes 提供了强制 [基于角色的访问控制 (RBAC) 授权的](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) 功能，以此来控制哪些用户可以访问资源。RBAC 可以与 ClusterRole 资源类型（作用域为整个集群）或 Role 资源类型（作用域为特定命名空间）配合使用。这意味着您可以全局或特定命名空间内限制用户可访问的资源。

Because you can issue cluster-wide roles, you can restrict who is allowed to create new namespaces, among other resources. This helps you avoid a namespace “explosion” in which self-servicing users create resources in an uncontrolled fashion, making it harder for you to administer the cluster.  
由于您可以发布集群范围的角色，因此您可以限制哪些人有权创建新的命名空间以及其他资源。这有助于避免命名空间“爆炸”——自助服务用户以不受控制的方式创建资源，从而增加您管理集群的难度。

![role based access control](https://cdn.prod.website-files.com/6527fe8ad7301efb15574cc7/654cd76416fd0c35c9a3cefd_diagrams-V2-02-1024x576.png)

role based access control

### Know When to Use Multiple Namespaces了解何时使用多个命名空间

Without a set namespace strategy, you might be tempted to use namespaces in suboptimal ways. This could mean putting everything in the default namespace, as mentioned above, or the other extreme of using too many namespaces when it doesn’t make sense to do so.  
如果没有设定命名空间策略，您可能会倾向于以次优的方式使用命名空间。这可能意味着将所有内容都放在默认命名空间中（如上所述），或者另一个极端是使用过多的命名空间（尽管这样做毫无意义）。

The [official Kubernetes documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/#when-to-use-multiple-namespaces) suggests that you only use namespaces when you need the features they offer, not simply for the sake of using them. Small teams of users, for instance, are unlikely to need namespaces. This approach can help reduce the overhead of dealing with yet another construct.  
[Kubernetes 官方文档](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/#when-to-use-multiple-namespaces) 建议，仅在需要命名空间提供的功能时才使用命名空间，而不是仅仅为了使用而使用。例如，小型用户团队不太可能需要命名空间。这种方法可以帮助减少处理额外构造的开销。

At the same time, namespaces can be beneficial even for small teams. One textbook example is using namespaces for different deployment environments, such as production, develop, or testing. Each of these environments would likely be running more or less the same application, but you wouldn’t want to share resources between them.  
同时，即使对于小型团队来说，命名空间也能带来益处。一个典型的例子是将命名空间用于不同的部署环境，例如生产、开发或测试。这些环境可能运行着大致相同的应用程序，但您肯定不希望它们之间共享资源。

Multiple namespaces can also make sense in an organisation with distinct workgroups. This could be teams, or collections of teams, who work on different projects that seldom overlap from a DevOps perspective. If the workgroups are separated, users can work together with relative freedom while isolated from unrelated groups. This helps ensure that the resources and constraints of one group don’t affect any others.  
对于拥有不同工作组的组织来说，多个命名空间也很有意义。这些工作组可能是团队，也可能是多个团队的集合，他们负责不同的项目，从 DevOps 的角度来看，这些项目很少重叠。如果将工作组分开，用户可以相对自由地协同工作，同时与不相关的组隔离。这有助于确保一个组的资源和约束不会影响其他组。

### Know What Belongs in a Namespace了解命名空间中的内容

Namespaces are primarily a separation mechanism, but there are some intricacies to keep in mind when using them so that you don’t run into any surprises. One important thing to note is that namespace scoping doesn’t apply to all resources; it’s only available for namespaced objects such as pods, deployments, or services, and not for cluster-wide resources like StorageClasses, nodes, or PersistentVolumes. This is a key concept for naming Kubernetes resources, so your mental model of how things fit together must be accurate.  
命名空间主要是一种隔离机制，但在使用时需要注意一些细节，以免出现意外。需要注意的是，命名空间作用域并非适用于所有资源；它仅适用于命名空间对象（例如 Pod、部署或服务），而不适用于集群范围的资源（例如 StorageClass、节点或持久卷）。这是命名 Kubernetes 资源的关键概念，因此您必须准确理解资源如何协同工作。

![Kubernetes namespaces](https://cdn.prod.website-files.com/6527fe8ad7301efb15574cc7/654cd76416fd0c35c9a3cf12_diagrams-V2-01-1024x576.png)

Kubernetes namespaces

For a complete list of resources that are and are not namespaced, run the following commands:  
要获取具有和不具有命名空间的资源的完整列表，请运行以下命令：

<sup># namespaced resources<br>kubectl api-resources --namespaced=true<br><br># non-namespaced resources<br>kubectl api-resources --namespaced=false</sup>  
\# 命名空间资源 kubectl api-resources --namespaced=true # 非命名空间资源 kubectl api-resources --namespaced=false

### Limit Resource Usage for Specific Namespaces限制特定命名空间的资源使用

If you’re running multiple namespaces on a cluster with limited resources, it might make sense to limit the resource usage of specific namespaces. For example, if you’re running namespaces for production, develop, and testing, you’d want production to have access to more resources than the others—maybe even unlimited resources. But you wouldn’t want testing or develop to consume so many resources that performance in production is degraded.  
如果您在资源有限的集群上运行多个命名空间，那么限制特定命名空间的资源使用量可能是合理的。例如，如果您同时运行用于生产、开发和测试的命名空间，那么您希望生产环境能够比其他环境访问更多资源，甚至可能是无限的资源。但您也不希望测试或开发环境消耗过多的资源，以至于降低生产环境的性能。

You can solve this through the application of [resource quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/). If you run the aforementioned command to see all namespaced API resources, you’ll note that ResourceQuotas is among them. This means that each namespace can have its own quota that limits the amount of resources it can use from the node. The Kubernetes documentation offers [this example](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/admin/resource/quota-mem-cpu.yaml) of a simple ResourceQuota:  
你可以通过应用 [资源配额](https://kubernetes.io/docs/concepts/policy/resource-quotas/) 来解决这个问题。如果你运行上述命令查看所有命名空间 API 资源，你会注意到 ResourceQuotas 也在其中。这意味着每个命名空间都可以拥有自己的配额，以限制其从节点使用的资源量。Kubernetes 文档提供了一个简单的 ResourceQuota [示例](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/admin/resource/quota-mem-cpu.yaml) ：

<sup>apiVersion: v1<br>kind: ResourceQuota<br>metadata:<br>name: mem-cpu-demo<br>spec:<br>hard:<br>requests.cpu: "1"<br>requests.memory: 1Gi<br>limits.cpu: "2"<br>limits.memory: 2Gi</sup>  
api 版本：v1 种类：ResourceQuota 元数据： 名称：mem-cpu-demo 规格： 难的： 请求.cpu：“1” 请求内存：1Gi 限制.cpu:“2” 限制内存：2Gi

This ResourceQuota would enforce these restrictions on all pods in the namespaces. This is one method for dividing cluster resources between multiple user groups, proving that namespaces can serve as a powerful mechanism for achieving various outcomes.  
此 ResourceQuota 会将这些限制强制应用于命名空间中的所有 Pod。这是在多个用户组之间划分集群资源的一种方法，证明了命名空间可以作为一种强大的机制来实现各种结果。

One of the key benefits of namespaces is that they provide a mechanism for separation. Resources in one namespace are generally isolated from resources in other namespaces. Sometimes this isolation level isn’t enough, and you need to leverage different separation mechanisms, such as separate clusters.  
命名空间的主要优势之一是它提供了一种隔离机制。一个命名空间中的资源通常与其他命名空间中的资源相互隔离。有时，这种隔离级别还不够，您需要利用不同的隔离机制，例如单独的集群。

That might be the right solution if you serve an application in multiple geographic regions. You \*could\* make other namespaces for each region, but that isn’t ideal, since the [cluster will still be chiefly located in a single](https://www.appvia.io/resources/video/single-or-multi-tenantclusters-when-to-and-why-to/) physical region. In cases like this, it’s better for the resources to be physically closer to users to reduce latency as well as to address various [data sovereignty issues](https://www.archive360.com/blog/data-sovereignty-and-the-gdpr-do-you-know-where-your-data-is).  
如果您在多个地理区域为应用程序提供服务，这或许是正确的解决方案。您可以为每个区域创建不同的命名空间，但这并不理想，因为 [集群仍然主要位于单个](https://www.appvia.io/resources/video/single-or-multi-tenantclusters-when-to-and-why-to/) 物理区域。在这种情况下，最好将资源物理上靠近用户，以减少延迟并解决各种 [数据主权问题](https://www.archive360.com/blog/data-sovereignty-and-the-gdpr-do-you-know-where-your-data-is) 。

The better solution might be to have separate clusters for each geographic region in which each cluster contains similarly named namespaces. This approach gives you all the benefits of a rigid separation between regions and the scalable naming and discoverability of consistent namespace names.  
更好的解决方案可能是为每个地理区域设置单独的集群，每个集群包含名称相似的命名空间。这种方法可以带来以下优势：区域之间严格划分，以及命名空间名称的可扩展性和可发现性。

### Conclusion 结论

Namespaces are a key concept in Kubernetes and any business that uses Kubernetes is likely to deal with them in one way or another. This is why it’s essential for you to understand their strengths and limitations.  
命名空间是 Kubernetes 中的一个关键概念，任何使用 Kubernetes 的企业都可能以某种方式与其打交道。因此，了解命名空间的优势和局限性至关重要。

If you know when to create a new namespace, use an existing one, or rely on a different construct altogether, you’ll be better able to use this powerful feature and your Kubernetes workflows will improve.  
如果您知道何时创建新的命名空间、使用现有的命名空间或完全依赖不同的构造，您将能够更好地使用这一强大的功能，并且您的 Kubernetes 工作流程也会得到改善。