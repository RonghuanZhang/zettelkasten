---
type: source-note
title: A visual guide on troubleshooting Kubernetes deployments
id: 20250807150850
created: 2025-08-07T15:56:50
source:
  - web
url: https://learnkube.com/troubleshooting-deployments
tags:
  - source-note
  - kubernetes
  - trouble-shooting
processed: false
archived: false
---
[Daniele Polencic](https://linkedin.com/in/danielepolencic "Daniele Polencic")

May 2024

---

[![A visual guide on troubleshooting Kubernetes deployments](https://static.learnkube.com/fae60444184ca7bd8c3698d866c24617.png)](https://static.learnkube.com/dac10c60ec5d2fe6bd3d3f8736cf0ce0.pdf)

**TL;DR:** here's a diagram to help you debug your deployments in Kubernetes (and you can [download it in the PDF version](https://static.learnkube.com/dac10c60ec5d2fe6bd3d3f8736cf0ce0.pdf) and [PNG](https://static.learnkube.com/7bcf2c9e9dce01269c436a16b77b276f.png)).

This diagram is also translated into the following languages:

- [中文](https://static.learnkube.com/168db7d27bbf0e31a0bd038bf98757fd.pdf), Translated by Addo Zhang ([PDF](https://static.learnkube.com/168db7d27bbf0e31a0bd038bf98757fd.pdf) | [PNG](https://static.learnkube.com/f717e9ad85d60fa9bc7aeac4a0ca9791.png))
- [Português](https://static.learnkube.com/80e47e991c182af3f3c25680fe179c16.pdf) — Translated by Marcelo Andrade ([PDF](https://static.learnkube.com/80e47e991c182af3f3c25680fe179c16.pdf) | [PNG](https://static.learnkube.com/57be67d5506df6774bebd6a298c0fbf1.png))
- [Español mexicano](https://static.learnkube.com/c925d360b887a06a76e4d077c3ae4f6b.pdf) — Translated by Raymundo Escobar & Jorge Vargas ([PDF](https://static.learnkube.com/c925d360b887a06a76e4d077c3ae4f6b.pdf) | [PNG](https://static.learnkube.com/6ea7543350975e150300e3a94573e5c5.png))
- [Español](https://static.learnkube.com/2d7578d7d1cae9519711edadd20e51b9.pdf) — Translated by Jose Angel Muñoz ([PDF](https://static.learnkube.com/2d7578d7d1cae9519711edadd20e51b9.pdf) | [PNG](https://static.learnkube.com/23d26432dfcae1850efef1e49e291ee5.png))
- [한국어](https://static.learnkube.com/72b11ef0d2797ea62d48905a5262ae88.pdf) — Translated by Hoon Jo ([PDF](https://static.learnkube.com/72b11ef0d2797ea62d48905a5262ae88.pdf) | [PNG](https://static.learnkube.com/d9a815a395aef281bd6a2488c5a3d711.png))
- [Français](https://static.learnkube.com/25c8a88cfa025bd48928e6ed532d9b79.pdf) — Translated by Marc Carmier ([PDF](https://static.learnkube.com/25c8a88cfa025bd48928e6ed532d9b79.pdf) | [PNG](https://static.learnkube.com/05d5479a4cb3bed680c64d06927ce336.png))
- [Русский язык](https://static.learnkube.com/b7c16b6b3ec5cb662cb05fbaef460428.pdf) — Translated by Viktor Oreshkin ([PDF](https://static.learnkube.com/b7c16b6b3ec5cb662cb05fbaef460428.pdf) | [PNG](https://static.learnkube.com/a023b4bcfc9287d08d694fa29cb232e0.png))
- [Italiano](https://static.learnkube.com/74e8bb541fb77b78fea1b55104f52129.pdf) — Translated by Stefano Ciaprini ([PDF](https://static.learnkube.com/74e8bb541fb77b78fea1b55104f52129.pdf) | [PNG](https://static.learnkube.com/2261a31e6ab5bf288e09b8845c29f83d.png))
- [繁中](https://static.learnkube.com/e557fbb3bb121d74dfcaaac9b3cb20ff.pdf) — Translated by Erhwen Kuo ([PDF](https://static.learnkube.com/e557fbb3bb121d74dfcaaac9b3cb20ff.pdf) | [PNG](https://static.learnkube.com/cbe079ed7f6e7764445aa746b2c9f295.png))
- [ελληνικά](https://static.learnkube.com/ee84e02f47cf7a62ead149a64f131cc8.pdf) — Translated by Ioannis Bitros ([PDF](https://static.learnkube.com/ee84e02f47cf7a62ead149a64f131cc8.pdf) | [PNG](https://static.learnkube.com/6afcf5afb39ebf7d13925e58e554fddd.png))
- [Türkçe](https://static.learnkube.com/10380e988858aa0f89e1dee305589368.pdf) — Translated by Gulcan Topcu ([PDF](https://static.learnkube.com/10380e988858aa0f89e1dee305589368.pdf) | [PNG](https://static.learnkube.com/28f6010f7a66c0986e9353611c34a67b.png))

If you'd like to contribute with a translation, get in touch at [hello@learnkube.com](https://learnkube.com/).

---

When you wish to deploy an application in Kubernetes, you usually define three components:

- A **Deployment** — a recipe for creating copies of your application.
- A **Service** — an internal load balancer that routes the traffic to Pods.
- An **Ingress** — a description of how the traffic should flow from outside the cluster to your Service.

Here's a quick visual recap.

- ![In Kubernetes, your applications are exposed through two layers of load balancers: internal and external.](https://learnkube.com/a/92543837cbecdd1189ee0a6d68fa9434.svg)
	1 /3
	In Kubernetes, your applications are exposed through two layers of load balancers: internal and external.
- ![The internal load balancer is called Service, whereas the external one is called Ingress.](https://learnkube.com/a/202ec241073585d03480f5d58560ccb2.svg)
	2 /3
	The internal load balancer is called Service, whereas the external one is called Ingress.
- ![Pods are not deployed directly. Instead, the Deployment creates the Pods and watches over them.](https://learnkube.com/a/c0a7fd878b9da1b42ba1d84597a45235.svg)
	3 /3
	Pods are not deployed directly. Instead, the Deployment creates the Pods and watches over them.

Assuming you wish to deploy a simple *Hello World* application, the YAML for such an application should look similar to this:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    track: canary
spec:
  selector:
    matchLabels:
      any-name: my-app
  template:
    metadata:
      labels:
        any-name: my-app
    spec:
      containers:
        - name: cont1
          image: ghcr.io/learnk8s/app:1.0.0
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - port: 80
      targetPort: 8080
  selector:
    name: app
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: my-service
            port:
              number: 80
        path: /
        pathType: Prefix
```

The definition is quite long, and it's easy to overlook how the components relate.

For example:

- *When should you use port 80, and when port 8080?*
- *Should you create a new port for every Service so that they don't clash?*
- *Do label names matter? Should it be the same everywhere?*

Before focusing on the debugging, let's recap how the three components link to each other.

Let's start with Deployment and Service.

## Connecting Deployment and Service

**The surprising news is that Service and Deployment aren't connected at all.**

Instead, the Service directly points to the Pods and skips the Deployment.

You should pay attention to how the Pods and the Services are related to each other.

You should remember three things:

1. The Service selector should match at least one Pod's label.
2. The Service's `targetPort` should match the `containerPort` of the Pod.
3. The Service `port` can be any number. Multiple Services can use the same port because different IP addresses are assigned.

The following diagram summarises how to connect the ports:

- ![Consider the following Pod exposed by a Service.](https://learnkube.com/a/2adc624ed44f39ac5577c42ed9c621bc.svg)
	1 /5
	Consider the following Pod exposed by a Service.
- ![When you create a Pod, you should define the port containerPort for each container in your Pods.](https://learnkube.com/a/441d2ca9f4ce838178e207c20bb17a24.svg)
	2 /5
	When you create a Pod, you should define the port `containerPort` for each container in your Pods.
- ![When you create a Service, you can define a port and a targetPort. But which one should you connect to the container?](https://learnkube.com/a/8454ae73acd4e2bf2ea874e77e768cfa.svg)
	3 /5
	When you create a Service, you can define a `port` and a `targetPort`. *But which one should you connect to the container?*
- ![targetPort and containerPort should always match.](https://learnkube.com/a/d8e5ddb44443d7b78e536aa7809b36f5.svg)
	4 /5
	`targetPort` and `containerPort` should always match.
- ![If your container exposes port 3000, then the targetPort should match that number.](https://learnkube.com/a/79612e74f9a7d386c5558503068171b9.svg)
	5 /5
	If your container exposes port 3000, then the `targetPort` should match that number.

If you look at the YAML, the labels and `ports` / `targetPort` should match:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    track: canary
spec:
  selector:
    matchLabels:
      any-name: my-app
  template:
    metadata:
      labels:
        any-name: my-app
    spec:
      containers:
        - name: cont1
          image: ghcr.io/learnk8s/app:1.0.0
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - port: 80
      targetPort: 8080
  selector:
    any-name: my-app
```

*What about the `track: canary` label at the top of the Deployment?*

*Should that match too?*

That label belongs to the deployment, and it's not used by the Service's selector to route traffic.

In other words, you can safely remove it or assign it a different value.

*And what about the `matchLabels` selector*?

[**It always has to match the Pod's labels**](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#resources-that-support-set-based-requirements), and the Deployment uses it to track the Pods.

*Assuming that you made the correct change, how do you test it?*

You can check if the Pods have the correct label with the following command:

```
kubectl get pods --show-labels
NAME                  READY   STATUS    LABELS
my-deployment-pv6pd   1/1     Running   any-name=my-app,pod-template-hash=7d6979fb54
my-deployment-f36rt   1/1     Running   any-name=my-app,pod-template-hash=7d6979fb54
```

Or if you have Pods belonging to several applications:

```
kubectl get pods --selector any-name=my-app --show-labels
```

Where `any-name=my-app` is the label `any-name: my-app`.

*Still having issues?*

You can also connect to the Pod!

You can use the `port-forward` command in kubectl to connect to the Service and test the connection.

```
kubectl port-forward service/<service name> 3000:80
Forwarding from 127.0.0.1:3000 -> 8080
Forwarding from [::1]:3000 -> 8080
```

Where:

- `service/<service name>` is the name of the service — in the current YAML is "my-service".
- 3000 is the port that you wish to open on your computer.
- 80 is the port the Service exposes in the `port` field.

If you can connect, the setup is correct.

If you can't, you most likely misplaced a label, or the port doesn't match.

## Connecting Service and Ingress

**The next step in exposing your app is to configure the Ingress.**

The Ingress must know how to retrieve the Service to connect the Pods and route traffic.

The Ingress retrieves the correct Service by name and port exposed.

Two things should match in the Ingress and Service:

1. The `service.port` of the Ingress should match the `port` of the Service
2. The `service.name` of the Ingress should match the `name` of the Service

The following diagram summarises how to connect the ports:

- ![You already know that the Service exposes a port.](https://learnkube.com/a/f44dd6eea6391537dd84dee5344b7b83.svg)
	1 /4
	You already know that the Service exposes a `port`.
- ![The Ingress has a field called servicePort.](https://learnkube.com/a/212eb8d3988e4b45ed7021f3546e4a57.svg)
	2 /4
	The Ingress has a field called `servicePort`.
- ![The Service port and the Ingress servicePort should always match.](https://learnkube.com/a/523463c584ac57b4fa7a34f4d2a8316b.svg)
	3 /4
	The Service `port` and the Ingress `servicePort` should always match.
- ![If you decide to assign port 80 to the service, you should change servicePort to 80 too.](https://learnkube.com/a/70fca3015bab436f3bcef5fc7c8a6c96.svg)
	4 /4
	If you decide to assign port 80 to the service, you should change `servicePort` to 80 too.

In practice, you should look at these lines:

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - port: 80
      targetPort: 8080
  selector:
    any-name: my-app
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: my-service
            port:
              number: 80
        path: /
        pathType: Prefix
```

*How do you test that the Ingress works?*

You can use the same strategy as before with `kubectl port-forward`, but instead of connecting to a Service, you should connect to the Ingress controller.

First, retrieve the Pod name for the Ingress controller with:

```
kubectl get pods --all-namespaces
NAMESPACE   NAME                              READY STATUS
kube-system coredns-5644d7b6d9-jn7cq          1/1   Running
kube-system etcd-minikube                     1/1   Running
kube-system kube-apiserver-minikube           1/1   Running
kube-system kube-controller-manager-minikube  1/1   Running
kube-system kube-proxy-zvf2h                  1/1   Running
kube-system kube-scheduler-minikube           1/1   Running
kube-system nginx-ingress-controller-6fc5bcc  1/1   Running
```

Identify the Ingress Pod (which might be in a different Namespace) and describe it to retrieve the port:

```
kubectl describe pod nginx-ingress-controller-6fc5bcc \
 --namespace kube-system \
 | grep Ports
Ports:         80/TCP, 443/TCP, 18080/TCP
```

Finally, connect to the Pod:

```
kubectl port-forward nginx-ingress-controller-6fc5bcc 3000:80 --namespace kube-system
Forwarding from 127.0.0.1:3000 -> 80
Forwarding from [::1]:3000 -> 80
```

At this point, every time you visit port 3000 on your computer, the request is forwarded to port 80 on the Ingress controller Pod.

If you visit [http://localhost:3000](http://localhost:3000/), you should find the app serving a web page.

## Recap on ports

Here's a quick recap on what ports and labels should match:

1. The Service selector should match the label of the Pod
2. The Service `targetPort` should match the `containerPort` of the container inside the Pod
3. The Service port can be any number. Multiple Services can use the same port because different IP addresses are assigned.
4. The `service.port` of the Ingress should match the `port` in the Service
5. The name of the Service should match the field `service.name` in the Ingress

Knowing how to structure your YAML definition is only part of the story.

*What happens when something goes wrong?*

Perhaps the Pod doesn't start, or it's crashing.

## 3 steps to troubleshoot Kubernetes deployments

It's essential to have a well-defined mental model of how Kubernetes works before debugging a broken deployment.

Since every deployment has three components, you should debug all of them in order, starting from the bottom.

1. You should make sure that your **Pods are running**, then
2. Focus on getting the **Service to route traffic** to the Pods and then
3. Check that the **Ingress is correctly configured.**

- ![You should start troubleshooting your deployments from the bottom. First, check that the Pod is Ready and Running.](https://learnkube.com/a/6aa1ab46349a22202f6cfe73022e9a12.svg)
	1 /3
	You should start troubleshooting your deployments from the bottom. First, check that the Pod is *Ready* and *Running*.
- ![If the Pods is Ready, you should investigate if the Service can distribute traffic to the Pods.](https://learnkube.com/a/140d05f49bf4347e37f664e370441486.svg)
	2 /3
	If the Pods is *Ready*, you should investigate if the Service can distribute traffic to the Pods.
- ![Finally, you should examine the connection between the Service and the Ingress.](https://learnkube.com/a/c3f870a281346c4bdc64de1ca81d21a0.svg)
	3 /3
	Finally, you should examine the connection between the Service and the Ingress.

## 1\. Troubleshooting Pods

**Most of the time, the issue is in the Pod itself.**

You should ensure your Pods are *Running* and *Ready*.

*How do you check that?*

```
kubectl get pods
NAME                    READY STATUS            RESTARTS  AGE
app1                    0/1   ImagePullBackOff  0         47h
app2                    0/1   Error             0         47h
app3-76f9fcd46b-xbv4k   1/1   Running           1         47h
```

In the output above, the last Pod is *Running* and *Ready* — however, the first two Pods are neither *Running* nor *Ready*.

*How do you investigate what went wrong?*

There are four valuable commands to troubleshoot Pods:

1. `kubectl logs <pod name>` helps retrieve the logs of the containers of the Pod.
2. `kubectl describe pod <pod name>` is helpful to retrieve a list of events associated with the Pod.
3. `kubectl get pod <pod name>` helps extract the YAML definition of the Pod as stored in Kubernetes.
4. `kubectl exec -ti <pod name> -- bash` is helpful to run an interactive command within one of the containers of the Pod.

*Which one should you use?*

There isn't a one-size-fits-all.

Instead, you should use a combination of them.

Pods can have startup and runtime errors.

Startup errors include:

- ImagePullBackoff
- ImageInspectError
- ErrImagePull
- ErrImageNeverPull
- RegistryUnavailable
- InvalidImageName

Runtime errors include:

- CrashLoopBackOff
- RunContainerError
- KillContainerError
- VerifyNonRootError
- RunInitContainerError
- CreatePodSandboxError
- ConfigPodSandboxError
- KillPodSandboxError
- SetupNetworkError
- TeardownNetworkError

Some errors are more common than others.

The following is a list of the most common errors and how you can fix them.

### ImagePullBackOff

**This error appears when Kubernetes is unable to retrieve the image for one of the pod's containers.**

There are three common culprits:

1. **The image name is invalid** —for example, you misspelt the name, or the image does not exist.
2. You specified a **non-existing tag** for the image.
3. The image you're trying to retrieve belongs to a **private registry**, and Kubernetes doesn't have credentials to access it.

Correcting the image name and tag can solve the first two cases.

Lastly, you should add the credentials to your private registry in a Secret and reference them in your Pods.

[The official documentation has an example of how you could do that.](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

### CrashLoopBackOff

If the container can't start, Kubernetes shows the CrashLoopBackOff message as a status.

Usually, a container can't start when:

1. There's an error in the application that **prevents it from starting.**
2. You [misconfigured the container](https://stackoverflow.com/questions/41604499/my-kubernetes-pods-keep-crashing-with-crashloopbackoff-but-i-cant-find-any-lo).
3. The Liveness probe failed too many times.

**You should retrieve the logs from that container to investigate why it failed.**

If you can't see the logs because your container is restarting too quickly, you can use the following command:

```
kubectl logs <pod-name> --previous
```

Which prints the error messages from the previous container.

### RunContainerError

**The error appears when the container is unable to start.**

That's even before the application inside the container starts.

The issue is usually due to misconfiguration, such as:

- **Mounting a non-existent volume** such as ConfigMap or Secrets.
- Mounting a read-only volume as read-write.

You should use `kubectl describe pod <pod-name>` to inspect and analyse the errors.

### Pods in a Pending state

When you create a Pod, the Pod stays in the *Pending* state.

*Why?*

Assuming that your scheduler component is running fine, here are the causes:

1. **The cluster doesn't have enough resources**, such as CPU and memory, to run the Pod.
2. The current Namespace has a ResourceQuota object, and creating the Pod will increase the Namespace's quota.
3. The Pod is bound to a *Pending* PersistentVolumeClaim.

Your best option is to inspect the *Events* section in the `kubectl describe` command:

```
kubectl describe pod <pod name>
```

For errors that are created as a result of ResourceQuotas, you can inspect the logs of the cluster with the following:

```
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Pods in a not Ready state

If a Pod is *Running* but not *Ready*, the Readiness probe is failing.

**When the Readiness probe fails, the Pod isn't attached to the Service, and no traffic is forwarded to that instance.**

A failing Readiness probe is an application-specific error, so you should inspect the *Events* section in `kubectl describe` to identify the error.

## 2\. Troubleshooting Services

If your Pods are *Running* and *Ready*, but still cannot receive a response from your app, you should check if the Service is configured correctly.

**Services are designed to route the traffic to Pods based on their labels.**

So the first thing you should check is how many Pods the Service targets.

You can do so by checking the Endpoints in the Service:

```
kubectl describe service my-service
Name:                     my-service
Namespace:                default
Selector:                 app=my-app
IP:                       10.100.194.137
Port:                     <unset>  80/TCP
TargetPort:               8080/TCP
Endpoints:                172.17.0.5:8080
```

An endpoint is a pair of `<ip address:port>`, and there should be at least one — when the Service targets (at least) a Pod.

If the "Endpoints" section is empty, there are two explanations:

1. **You don't have any Pod running with the correct label** (hint: you should check if you are in the correct namespace).
2. You have a typo in the `selector` labels of the Service.

If you see a list of endpoints but still can't access your application, the `targetPort` in your service is likely the culprit.

*How do you test the Service?*

Regardless of the type of Service, you can use `kubectl port-forward` to connect to it:

```
kubectl port-forward service/<service-name> 3000:80
```

Where:

- `<service-name>` is the name of the Service.
- `3000` is the port you wish to open on your computer.
- `80` is the port exposed by the Service.

## 3\. Troubleshooting Ingress

If you've reached this section, then:

- The Pods are *Running* and *Ready*.
- The Service distributes the traffic to the Pod.

But you still can't see a response from your app.

**This means that the Ingress is most likely misconfigured.**

Since the Ingress controller is a third-party component in the cluster, there are different debugging techniques depending on the type of Ingress controller.

But before diving into Ingress-specific tools, there's something straightforward that you can check.

The Ingress uses the `service.name` and `service.port` to connect to the Service.

You should check that those are correctly configured.

You can inspect that the Ingress is correctly configured with:

```
kubectl describe ingress my-ingress
Name:             my-ingress
Namespace:        default
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /   my-service:80 (<error: endpoints "my-service" not found>)
```

If the *Backend* column is empty, the configuration must have an error.

If you can see the endpoints in the *Backend* column but still can't access the application, the issue is likely to be:

- How you exposed your Ingress to the public internet.
- How you exposed your cluster to the public internet.

**You can isolate infrastructure issues from Ingress by directly connecting to the Ingress Pod.**

First, retrieve the Pod for your Ingress controller (which could be located in a different namespace):

```
kubectl get pods --all-namespaces
NAMESPACE   NAME                              READY STATUS
kube-system coredns-5644d7b6d9-jn7cq          1/1   Running
kube-system etcd-minikube                     1/1   Running
kube-system kube-apiserver-minikube           1/1   Running
kube-system kube-controller-manager-minikube  1/1   Running
kube-system kube-proxy-zvf2h                  1/1   Running
kube-system kube-scheduler-minikube           1/1   Running
kube-system nginx-ingress-controller-6fc5bcc  1/1   Running
```

Describe it to retrieve the port:

```
kubectl describe pod nginx-ingress-controller-6fc5bcc
 --namespace kube-system \
 | grep Ports
    Ports:         80/TCP, 443/TCP, 8443/TCP
    Host Ports:    80/TCP, 443/TCP, 0/TCP
```

Finally, connect to the Pod:

```
kubectl port-forward nginx-ingress-controller-6fc5bcc 3000:80 --namespace kube-system
Forwarding from 127.0.0.1:3000 -> 80
Forwarding from [::1]:3000 -> 80
```

At this point, every time you visit port 3000 on your computer, the request is forwarded to port 80 on the Pod.

*Does it work now?*

- If it does, **the issue is in the infrastructure.** You should investigate how the traffic is routed to your cluster.
- If it doesn't work, **the problem is in the Ingress controller.** You should debug it.

If you still can't get the Ingress controller to work, you should start debugging it.

There are many different versions of Ingress controllers.

Popular options include Nginx, HAProxy, Traefik, etc.

You should consult the documentation of your Ingress controller to find a troubleshooting guide.

Since [Ingress Nginx](https://github.com/kubernetes/ingress-nginx) is the most popular Ingress controller, we included a few tips for it in the next section.

### Debugging Ingress Nginx

The Ingress-nginx project has an [official plugin for Kubectl](https://kubernetes.github.io/ingress-nginx/kubectl-plugin/).

You can use `kubectl ingress-nginx` to:

- Inspect logs, backends, certs, etc.
- Connect to the Ingress.
- Examine the current configuration.

The three commands that you should try are:

- `kubectl ingress-nginx lint`, which checks the `nginx.conf`.
- `kubectl ingress-nginx backend` to inspect the backend (similar to `kubectl describe ingress <ingress-name>`).
- `kubectl ingress-nginx logs` is used to check the logs.

> Please notice that you might need to specify the correct namespace for your Ingress controller with `--namespace <name>`.

## Summary

Troubleshooting in Kubernetes can be daunting if you don't know where to start.

**You should always remember to approach the problem from the bottom up: start with the Pods and move up the stack with Service and Ingress.**

The same debugging techniques that you learnt in this article can be applied to other objects, such as:

- Failing Jobs and CronJobs.
- StatefulSets and DaemonSets.

Many thanks to [Gergely Risko](https://github.com/errge), [Daniel Weibel](https://medium.com/@weibeld) and [Charles Christyraj](https://www.linkedin.com/in/charles-christyraj-0bab8a36/) for offering some invaluable suggestions.

- What is LearnKube?
	In-depth Kubernetes training that is practical and easy to understand.
- [⎈ Instructor-led workshops ❯](https://learnkube.com/training)
	Deep dive into containers and Kubernetes with the help of our instructors and become an expert in deploying applications at scale.
- [⎈ Online courses ❯](https://learnkube.com/academy)
	Learn Kubernetes online with hands-on, self-paced courses. No need to leave the comfort of your home.
- [⎈ Corporate training ❯](https://learnkube.com/corporate-training)
	Train your team in containers and Kubernetes with a customised learning path — remotely or on-site.