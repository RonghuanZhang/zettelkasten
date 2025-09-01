---
type: source-note
title: Good REST api structure of go gin project? go gin 项目的良好 REST api 结构？
id: 20250621090646
created: 2025-06-21T09:21:46
source:
  - web
url: https://www.reddit.com/r/golang/comments/11vkr1d/good_rest_api_structure_of_go_gin_project/
tags:
  - source-note
  - golang
  - golang/gin
processed: false
archived: false
---
I have seen lot of project structures in Go and confused to choose one, but specific to go gin with Postgres DB can anyone suggest a good structure. Please note it is not for a micro service's. Mostly it will be a monolith only but expecting it should be maintainable for at least 3-5 years with lots of modules. It is going to be a private internal software with 200-1000 users.

---

## Comments

> **mr\_no\_it\_alll** • [20 points](https://reddit.com/r/golang/comments/11vkr1d/comment/jcuwrrl/) • 2023-03-19
> 
> You should check out go-kit  
> 你应该看看 go-kit
> 
> > Go kit is a programming toolkit for building microservices (or elegant monoliths) in Go. We solve common problems in distributed systems and application architecture so you can focus on delivering business value.  
> > Go Kit 是一个用于用 Go 语言构建微服务（或优雅的单体式应用）的编程工具包。我们致力于解决分布式系统和应用架构中的常见问题，让您专注于交付业务价值。
> 
> I really like clean architecture and I think Go-Kit does a good job of following it (although it doesn't say explicitly :).  
> 我真的很喜欢干净的架构，我认为 Go-Kit 很好地遵循了它（尽管它没有明确说明:)。
> 
> For your purpose, look specifically at the example `shipping` at: [https://github.com/go-kit/examples](https://github.com/go-kit/examples). It was introduced into go-kit, and has a great blog post (iMO) that walkthrough it:  
> 为了达到您的目的，请特别查看 [https://github.com/go-kit/examples](https://github.com/go-kit/examples) 上的 `shipping` 。它被引入到 go-kit 中，并且有一篇很棒的博客文章 (iMO) 对其进行了演示：
> 
> - [part 1  第 1 部分](https://www.citerus.se/go-ddd/)
> - [part 2  第 2 部分](https://www.citerus.se/part-2-domain-driven-design-in-go/)
> - [part 3  第 3 部分](https://www.citerus.se/part-3-domain-driven-design-in-go/)
> 
> Note that a lot of blog posts about clean architecture in Go, are just following structures from other languages and frameworks, like Java and Ruby on Rails, and they have some really strong Go anti-patterns with package structuring. (like naming a package `models`/`services`/etc)  
> 请注意，许多关于 Go 中清洁架构的博客文章只是遵循其他语言和框架的结构，例如 Java 和 Ruby on Rails，并且它们在包结构方面有一些非常强大的 Go 反模式。（例如将包命名为 `models` / `services` /等）
> 
> Few links for a better picture of DDD and application and package structuring in go:  
> 以下几个链接可以更好地展示 DDD 以及 Go 中的应用程序和包结构：
> 
> - There is a very good talk by Kat about how to structure Go Applications: [https://www.youtube.com/watch?v=oL6JBUk6tj0](https://www.youtube.com/watch?v=oL6JBUk6tj0)  
> 	Kat 有一个关于如何构建 Go 应用程序的精彩演讲： [https://www.youtube.com/watch?v=](https://www.youtube.com/watch?v=oL6JBUk6tj0) oL6JBUk6tj0
> - Amazing talk by Bill Kenedy about Package Oriented Design in Go, that I think it's a must for anyone writing in Go: [https://www.youtube.com/watch?v=spKM5CyBwJA](https://www.youtube.com/watch?v=spKM5CyBwJA)  
> 	Bill Kenedy 关于 Go 中面向包设计的精彩演讲，我认为对于任何使用 Go 编写代码的人来说这都是必听的： [https://www.youtube.com/watch?v](https://www.youtube.com/watch?v=spKM5CyBwJA) =spKM5CyBwJA
> - And lastly, a great talk by Eddy Kiselman about DDD approaches in Go applications [https://www.youtu](https://www.youtube.com/watch?v=YfLPZOpJQjY).  
> 	最后，Eddy Kiselman 就 Go 应用程序中的 DDD 方法进行了精彩的演讲 [https://www.youtu](https://www.youtube.com/watch?v=YfLPZOpJQjY) 。
> 
> Hope it somehow helps  
> 希望它能有所帮助

> **Feisty-Assignment393** • [3 points](https://reddit.com/r/golang/comments/11vkr1d/comment/jcx5kqd/) • 2023-03-20
> 
> The structure I use is that used by Alex Edwards in Let's go further. Although he didn't use Gin specifically but you could adapt that structure for all your Go RESTAPI projects.  
> 我使用的结构是 Alex Edwards 在“让我们更进一步”一文中使用的。虽然他没有专门使用 Gin，但你可以将该结构应用于所有 Go RESTAPI 项目。

> **ArieHein** • [2 points](https://reddit.com/r/golang/comments/11vkr1d/comment/jctr3a7/) • 2023-03-19
> 
> Follow some videos from Anthony, he isn't using gin, but concepts are similar.First learn to do it simple, then think about more complex scenario.  
> 跟随安东尼的一些视频，他没有使用杜松子酒，但概念相似。首先学习做简单的事，然后思考更复杂的情况。
> 
> [https://www.youtube.com/@anthonygg\_](https://www.youtube.com/@anthonygg_)
> 
> > **krishnadaspc** • [2 points](https://reddit.com/r/golang/comments/11vkr1d/comment/jcuqnwy/) • 2023-03-19
> > 
> > I didn't find a video related to go in this link. Is this url correct?  
> > 我在这个链接里没有找到与 Go 相关的视频。这个网址正确吗？

> **cnowacek** • [1 points](https://reddit.com/r/golang/comments/11vkr1d/comment/jcvuaij/) • 2023-03-19
> 
> Ask chatgpt. You could probably get some good insights.  
> 问问 chatgpt。你或许能得到一些有用的见解。

> **joeyjiggle** • [\-9 points](https://reddit.com/r/golang/comments/11vkr1d/comment/jctol6u/) • 2023-03-19
> 
> I recommend this layout, but skip using pkg directory.
> 
> [https://github.com/golang-standards/project-layout](https://github.com/golang-standards/project-layout)
> 
> Assuming that this is what you mean, and not the code itself. Also checkout Iris.

> **\[已删除\]** • [\-7 points](https://reddit.com/r/golang/comments/11vkr1d/comment/jcti9bv/) • 2023-03-19
> 
> Please look up Clean Architecture. /domain houses use cases which are the business logic logic managers. Things like AddUserUseCase and CheckoutUseCase. This is also where you define Services, which are just simple interfaces like ReadUser or WriteOrder. Please do not expose a database transaction or connection here. You use case is IO agnostic. Things like transactions are handled in a different layer. /main houses your main method and factory to make use cases and such. My factories tend to be created per request. This allows the controller to get a fresh database transaction with ever call. Because you do this, the domain layer doesn’t need to know about the transaction. All services are injected at time of the request and the transaction is just part of the construction. /web houses your templates and presenters. A presenter takes the result from the use case and augments that into what ever the client will need. In a web project for example it would make the database calls to get values for drop downs and such. The UseCase does not know about that. /db would house data as related code. This is where you would implement the interfaces defined in the domain.
> 
> This should help. [https://github.com/deusdat/cleango](https://github.com/deusdat/cleango)

> **jrwren** • [1 points](https://reddit.com/r/golang/comments/11vkr1d/comment/jcw3ag1/) • 2023-03-20
> 
> No third party will be importing you code so  
> 没有第三方会导入你的代码，所以
> 
> 1. It doesn’t matter and  
> 	没关系，
> 2. All in one pkg in root module  
> 	根模块中的一体化包

> **amritmishra91** • [1 points](https://reddit.com/r/golang/comments/11vkr1d/comment/jcwld66/) • 2023-03-20
> 
> with the Go projects, I have worked in the past pretty much functionality isolated structures has worked well for me.  
> 对于 Go 项目，我过去曾使用过很多功能隔离结构，这些结构对我来说非常有效。
> 
> like every functional unit having it's own package hierarchy nested into it, for models, repositories, services, etc..  
> 就像每个功能单元都有自己的嵌套包层次结构一样，例如模型、存储库、服务等。
> 
> any cross functional or shared code can sit in common packages  
> 任何跨功能或共享代码都可以放在公共包中