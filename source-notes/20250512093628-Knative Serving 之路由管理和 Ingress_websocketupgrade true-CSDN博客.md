---
type: source-note
title: "Knative Serving 之路由管理和 Ingress_websocketupgrade: true-CSDN博客"
id: 20250512090528
created: 2025-05-12T09:36:28
source:
  - web
url: https://blog.csdn.net/yunqiinsight/article/details/99716486
tags:
  - source-note
  - cloud-native/serverless/knative
processed: false
archived: false
---
Knative 默认会为每一个 Service 生成一个域名，并且 Istio Gateway 要根据域名判断当前的请求应该转发给哪个 Knative Service。Knative 默认使用的主域名是 example.com，这个域名是不能作为线上服务的。本文我首先介绍一下如何修改 默认主域名，然后再深入一层介绍如何添加自定义域名以及如何根据 path 关联到不同的 Knative Service。

### Knative Serving 的默认域名 example.com

首先需要部署一个 Knative Service。如果你已经有了一个 Knative 集群，那么直接把下面的内容保存到 helloworld.yaml 文件中。然后执行一下 `kubectl apply -f helloworld.yaml ` 即可把 hello 服务部署到 helloworld namespace 中。

```cobol
---

apiVersion: v1

kind: Namespace

metadata:

  name: helloworld

 

---

apiVersion: serving.knative.dev/v1alpha1

kind: Service

metadata:

  name: hello

  namespace: helloworld

spec:

  template:

    metadata:

      labels:

        app: hello

      annotations:

        autoscaling.knative.dev/target: "10"

    spec:

      containers:

        - image: registry.cn-hangzhou.aliyuncs.com/knative-sample/simple-app:132e07c14c49

          env:

            - name: TARGET

              value: "World!"
```

现在我们来看一下 Knative Service 自动生成的域名配置:

```cobol
└─# kubectl -n helloworld get ksvc

NAME    URL                                   LATESTCREATED   LATESTREADY   READY   REASON

hello   http://hello.helloworld.example.com   hello-wsnvc     hello-wsnvc   True
```

现在使用 curl 指定 Host 就能访问服务了。

- 首先获取到 Istio Gateway IP
```cobol
└─# kubectl get svc istio-ingressgateway --namespace istio-system --output jsonpath="{.status.loadBalancer.ingress[*]['ip']}"

47.95.191.136
```
- 访问 hello 服务
```cobol
└─# curl -H "Host: hello.helloworld.example.com" http://47.95.191.136/

Hello World!!
```

如果想要在浏览器中访问 hello 服务需要先做 host 绑定，把域名 hello.helloworld.example.com 指向 47.95.191.136 才行。这种方式还不能对外提供服务。

### 使用自定义主域名

下面我来介绍一下如何把默认的 example.com 改成我们自己的域名，假设我们自己的域名是：serverless.kuberun.com，现在执行 `kubectl edit cm config-domain --namespace knative-serving` 如下图所示，添加 serverless.kuberun.com 到 ConfigMap 中，然后保存退出就完成了自定义主域名的配置。

![](https://i-blog.csdnimg.cn/blog_migrate/79f1cfb27dfcc57fd665ed04318b611e.png)

再来看一下 Knative Service 的域名, 如下所示已经生效了。

```cobol
└─# kubectl -n helloworld get ksvc

NAME    URL                                              LATESTCREATED   LATESTREADY   READY   REASON

hello   http://hello.helloworld.serverless.kuberun.com   hello-wsnvc     hello-wsnvc   True
```

**泛域名解析**  
Knative Service 默认生成域名的规则是 servicename.namespace.use-domain 。所以不同的 namespace 会生成不同的子域名，每一个 Knative Service 也会生成一个唯一的子域名。为了保证所有的 Service 服务都能在公网上面访问到，需要做一个泛域名解析。把 `*.serverless.kuberun.com` 解析到 Istio Gateway 47.95.191.136 上面去。如果你是在阿里云(万网)上面购买的域名，你可以通过如下方式配置域名解析：

![](https://i-blog.csdnimg.cn/blog_migrate/8c7c5da6031c6123c9259c5c822b6dba.png)

现在直接通过浏览器访问 [http://hello.helloworld.serverless.kuberun.com/](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fhello.helloworld.serverless.kuberun.com%2F) 就可以直接看到 helloworld 服务了：

![](https://i-blog.csdnimg.cn/blog_migrate/3d9c0f4be7c499a99354f9b0f1169944.png)

\## 自定义服务域名  
刚才我们给 Knative 指定了一个主域名，使得 Service 基于主域名生成自己的唯一域名。但自动生成的域名不是很友好，比如刚才部署的 helloworld 的域名 `hello.helloworld.serverless.kuberun.com` 对于普通用户来说意义不明显、不好记忆。  
如果能通过 `hello.kuberun.com` 访问 hello world 服务那就完美了，接下来我来介绍实现方法：

- 先在万网上面修改域名解析，把 hello.kuberun.com 的 A 记录指向 Istio Gateway 47.95.191.136

![](https://i-blog.csdnimg.cn/blog_migrate/401d2c8e22e6e56451b7a780db47e08c.png)

- hello.kuberun.com 解析到 Istio Gateway 以后 Istio Gateway 并不知道此应该转发到哪个服务，所以还需要配置 VirtualService 告知 Istio 如何转发，把下面的内容保存到 `hello-ingress-route.yaml` 文件：
	```cobol
	apiVersion: networking.istio.io/v1alpha3
	kind: VirtualService
	metadata:
	name: hello-ingress-route
	namespace: knative-serving
	spec:
	gateways:
	- knative-ingress-gateway
	hosts:
	- hello.helloworld.serverless.kuberun.com
	- hello.kuberun.com
	http:
	- match:
	 - uri:
	     prefix: "/"
	 rewrite:
	   authority: hello.helloworld.svc.cluster.local
	 retries:
	   attempts: 3
	   perTryTimeout: 10m0s
	 route:
	 - destination:
	     host: istio-ingressgateway.istio-system.svc.cluster.local
	     port:
	       number: 80
	   weight: 100
	 timeout: 10m0s
	 websocketUpgrade: true
	```
	现在打开 [http://hello.kuberun.com/](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fhello.kuberun.com%2F) 就能看到 helloworld 服务了：

![](https://i-blog.csdnimg.cn/blog_migrate/980ecf8fcc20d5fee55c3fd175ffe92c.png)

**基于路径的服务转发**  
真实线上服务的场景可能是一个路径后端对应着一个应用，现在我们对刚才的 hello.kuberun.com 进行一下扩展。让 /blog 开头的路径映射到 blog service，其他的路径还是原样打到 hello service 上面。  
把下面的内容保存到 `blog.yaml` 文件，然后执行： ` kubectl apply -f blog.yaml` 即可完成 blog 服务的部署。

```cobol
---

apiVersion: v1

kind: Namespace

metadata:

 name: blog

 

---

apiVersion: serving.knative.dev/v1alpha1

kind: Service

metadata:

 name: hello-blog

 namespace: blog

spec:

 template:

   metadata:

     labels:

       app: hello

     annotations:

       autoscaling.knative.dev/target: "10"

   spec:

     containers:

       - image: registry.cn-hangzhou.aliyuncs.com/knative-sample/simple-app:132e07c14c49

         env:

           - name: TARGET

             value: "Blog!"
```

查看 blog 服务的默认域名：

```cobol
└─# kubectl -n blog get ksvc

NAME    URL                                        LATESTCREATED   LATESTREADY   READY   REASON

hello   http://hello-blog.blog.serverless.kuberun.com   hello-zbm7q     hello-zbm7q   True
```

现在使用浏览器打开 [http://hello-blog.blog.serverless.kuberun.com](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fhello-blog.blog.serverless.kuberun.com) 就可以访问刚刚部署的服务了：

![](https://i-blog.csdnimg.cn/blog_migrate/b5229c3c27793461efcff9bd9f708699.png)

这是默认域名，我们的需求是想要通过 [http://hello.kuberun.com/blog](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fhello.kuberun.com%2Fblog) 访问, 所以还需要修改 Istio VirtualService 的配置。如下所示在 hello-ingress-route.yaml 增加 /blog 的配置：

```cobol
apiVersion: networking.istio.io/v1alpha3

kind: VirtualService

metadata:

  name: hello-ingress-route

  namespace: knative-serving

spec:

  gateways:

  - knative-ingress-gateway

  hosts:

  - hello.helloworld.serverless.kuberun.com

  - hello.kuberun.com

  http:

  - match:

    - uri:

        prefix: "/blog"

    rewrite:

      authority: hello-blog.blog.svc.cluster.local

    retries:

      attempts: 3

      perTryTimeout: 10m0s

    route:

    - destination:

        host: istio-ingressgateway.istio-system.svc.cluster.local

        port:

          number: 80

      weight: 100

  - match:

    - uri:

        prefix: "/"

    rewrite:

      authority: hello.helloworld.svc.cluster.local

    retries:

      attempts: 3

      perTryTimeout: 10m0s

    route:

    - destination:

        host: istio-ingressgateway.istio-system.svc.cluster.local

        port:

          number: 80

      weight: 100

    timeout: 10m0s

    websocketUpgrade: true
```

现在就能在浏览器中打开 [http://hello.kuberun.com/blog](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fhello.kuberun.com%2Fblog) 如下所示：

![](https://i-blog.csdnimg.cn/blog_migrate/32273a4c10690eb06bf0144cdf475894.png)

### 小结

本文主要围绕 Knative Service 域名展开介绍了 Knative Service 的路由管理。您应该了解到如下内容：

- Knative Service 默认的主域名是 example.com, 所有 Knative Service 生成的独立域名都是这个主域名的子域名
- Knative Service 生成的域名规范
- 如何配置 Knative Service 使用自定义的主域名，以及如何配置公网域名解析
- 如何基于 Istio VirtualService 实现 Knative Service 的个性化 Ingress 配置，提供生产级别的服务路由

  
[原文链接](https://yq.aliyun.com/articles/714691?utm_content=g_1000072538)  
本文为云栖社区原创内容，未经允许不得转载。

实付 元

[使用余额支付](https://blog.csdn.net/yunqiinsight/article/details/)

点击重新获取

扫码支付

钱包余额 0

抵扣说明：

1.余额是钱包充值的虚拟货币，按照1:1的比例进行支付金额的抵扣。  
2.余额无法直接购买下载，可以购买VIP、付费专栏及课程。

[余额充值](https://i.csdn.net/#/wallet/balance/recharge)

举报

 [![](https://csdnimg.cn/release/blogv2/dist/pc/img/toolbar/Group.png) 点击体验  
DeepSeekR1满血版](https://ai.csdn.net/?utm_source=cknow_pc_blogdetail&spm=1001.2101.3001.10583) 隐藏侧栏 ![程序员都在用的中文IT技术交流社区](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_app.png)

程序员都在用的中文IT技术交流社区

![专业的中文 IT 技术社区，与千万技术人共成长](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_wechat.png)

专业的中文 IT 技术社区，与千万技术人共成长

![关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_video.png)

关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！

客服 返回顶部

![](https://i-blog.csdnimg.cn/blog_migrate/79f1cfb27dfcc57fd665ed04318b611e.png) ![](https://i-blog.csdnimg.cn/blog_migrate/8c7c5da6031c6123c9259c5c822b6dba.png) ![](https://i-blog.csdnimg.cn/blog_migrate/3d9c0f4be7c499a99354f9b0f1169944.png) ![](https://i-blog.csdnimg.cn/blog_migrate/401d2c8e22e6e56451b7a780db47e08c.png) ![](https://i-blog.csdnimg.cn/blog_migrate/980ecf8fcc20d5fee55c3fd175ffe92c.png) ![](https://i-blog.csdnimg.cn/blog_migrate/b5229c3c27793461efcff9bd9f708699.png) ![](https://i-blog.csdnimg.cn/blog_migrate/32273a4c10690eb06bf0144cdf475894.png)