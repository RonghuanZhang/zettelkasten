---
type: source-note
title: Installing Helm in Google Kubernetes Engine (GKE)
id: 20250623100653
created: 2025-06-23T10:18:53
source:
  - web
url: https://medium.com/google-cloud/installing-helm-in-google-kubernetes-engine-7f07f43c536e
tags:
  - source-note
  - gcp/gke
processed: false
archived: false
---
[Sitemap](https://medium.com/sitemap/sitemap.xml)## [Google Cloud - Community](https://medium.com/google-cloud?source=post_page---publication_nav-e52cf94d98af-7f07f43c536e---------------------------------------)

[Follow publication](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fcollection%2Fgoogle-cloud&operation=register&redirect=https%3A%2F%2Fmedium.com%2Fgoogle-cloud%2Finstalling-helm-in-google-kubernetes-engine-7f07f43c536e&collection=Google+Cloud+-+Community&collectionId=e52cf94d98af&source=post_page---publication_nav-e52cf94d98af-7f07f43c536e---------------------publication_nav------------------)

[![Google Cloud - Community](https://miro.medium.com/v2/resize:fill:76:76/1*FUjLiCANvATKeaJEeg20Rw.png)](https://medium.com/google-cloud?source=post_page---post_publication_sidebar-e52cf94d98af-7f07f43c536e---------------------------------------)

A collection of technical articles and blogs published or curated by Google Cloud Developer Advocates. The views expressed are those of the authors and don't necessarily reflect those of Google.

[Follow publication](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fcollection%2Fgoogle-cloud&operation=register&redirect=https%3A%2F%2Fmedium.com%2Fgoogle-cloud%2Finstalling-helm-in-google-kubernetes-engine-7f07f43c536e&collection=Google+Cloud+-+Community&collectionId=e52cf94d98af&source=post_page---post_publication_sidebar-e52cf94d98af-7f07f43c536e---------------------post_publication_sidebar------------------)

When I first started to really get into Kubernetes I would go find the docker image to various necessary programs and just create a template file around their code. This was easy enough and seemed like a good option. *Depending on your needs, this is a fine direction to go.*

However as time went on I would keep reading about [Helm](https://helm.sh/) and see that some of the applications I was creating templates for already had extremely well created templates in the form of [Helm Charts](https://docs.helm.sh/developing_charts). With this realization — and me being a lazy developer that wants to increase my development speed — it was time to learn Helm!

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*ImklDHK1qt-p2EbJ23j_7Q.gif)

Helm Does This To Your Kubernetes Development

## What Is Helm? What Will It Do For Me?

First, in case you haven’t read a ton of articles about Helm already, here is all you need to know about Helm. **Helm is a Package Manager for Kubernetes.** If you come from any languages (and like word puzzles) here is something for you to relate to.

> **NodeJS** is to **Kubernetes** as **NPM** is to **HELM!**
> 
> **Ruby** is to **Kubernetes** as **Gems** are to **HELM!**
> 
> **Swift** is to **Kubernetes** as **CocoaPods** are to **HELM!**
> 
> **Java** is to **Kubernetes** as **Maven** is to **HELM!**

I could keep going, and the comparison isn’t perfect but you should get the idea. Basically Helm takes the effort out of making sharable, packaged, template files that can easily be dropped into other Kubernetes Clusters without a lot of effort. This is a big win for you and your time. Some more terminology that you’ll hear that you need to know:

> A **Helm Chart** is a Helm Package. This is the chart or “instructions” of how to put together your releasable package.
> 
> **Tiller** is a server that runs inside your Kubernetes Cluster anytime you install Helm. Tiller manages installations of your Helm Charts. As Tiller installs containers into your Kubernetes Cluster on your behalf, security around this process should be a high priority for you.

Once you have Helm installed in your Kubernetes Cluster and everything going, you can add big functionality with a single line of code.

## Want Redis (Or Anything Else) In Your Kubernetes Cluster?

```hs
helm install stable/redis
```

BOOM! A Redis installation with a Master/Slave configuration for scalability and Persistence Volumes and, for giggles, even a [Prometheus](https://prometheus.io/) metrics exporter.

You want [more options](https://github.com/helm/charts/tree/master/stable)? [Wordpress](https://github.com/helm/charts/tree/master/stable/wordpress)? [Spinnaker](https://github.com/helm/charts/tree/master/stable/spinnaker)? [Sonarqube](https://github.com/helm/charts/tree/master/stable/sonarqube)? [PhpBB](https://github.com/helm/charts/tree/master/stable/phpbb)? [Mysql](https://github.com/helm/charts/tree/master/stable/mysql)? [Jenkins](https://github.com/helm/charts/tree/master/stable/jenkins)? [Drupal](https://github.com/helm/charts/tree/master/stable/drupal)? Yup, there are tons there for you to enjoy.

## I’m Sold! How Do I Install Helm Into My Kubernetes Cluster?!

Great! It is time to start talking code and installation scripts. There are two directions that you can go when installing Helm into your Kubernetes Cluster. The first is a fairly vanilla install that gets you all the basics but may not be as secure as you’d want for your production cluster. [The second will be much more secure using TLS to lockdown your Tiller to Helm connection.](https://medium.com/@jonbcampos/install-secure-helm-in-gke-254d520061f7) In the remainder of this article we are going to look at how to get a basic install working. This sort of installation is best if you have a very locked down Kubernetes Cluster already or if you are running your Kubernetes Cluster in [MiniKube](https://github.com/kubernetes/minikube) (aka, not production).

[In my next article I am going to focus specifically on installing Helm with TLS as it is a bit more involved.](https://medium.com/@jonbcampos/install-secure-helm-in-gke-254d520061f7) To be transparent, Helm does have very [good documentation](https://docs.helm.sh/using_helm/#installing-helm) on the installation process, however things just don’t always go as the documentation promises. As such I’m going to add some lessons learned that slowed me down so you don’t have the same issues.

*Update: I published the second article!*## [Install Secure Helm In GKE](https://medium.com/@jonbcampos/install-secure-helm-in-gke-254d520061f7?source=post_page-----7f07f43c536e---------------------------------------)

In my last post I talked a lot about the joys of Helm and why you should spend the time installing it into your…

medium.com

[View original](https://medium.com/@jonbcampos/install-secure-helm-in-gke-254d520061f7?source=post_page-----7f07f43c536e---------------------------------------)

*If you haven’t gone through or even read the* [*first part of this series*](https://medium.com/@jonbcampos/kubernetes-day-one-30a80b5dcb29) *you might be lost, have questions where the code is, or what was done previously. Remember this assumes you’re using* [*GCP*](https://cloud.google.com/) *and* [*GKE*](https://cloud.google.com/kubernetes-engine/)*. I will always provide the code and how to test the code is working as intended.*## [Kubernetes: Day One](https://medium.com/google-cloud/kubernetes-day-one-30a80b5dcb29?source=post_page-----7f07f43c536e---------------------------------------)

This is the obligatory step one Kubernetes post. If you’re interested in Kubernetes you’ve probably read 100 of these…

medium.com

[View original](https://medium.com/google-cloud/kubernetes-day-one-30a80b5dcb29?source=post_page-----7f07f43c536e---------------------------------------)

## First, Create Your Kubernetes Cluster

To install Helm into your Kubernetes Cluster you’ll first need a Kubernetes Cluster. I’ve created some scripts to make this easier as creating the Cluster isn’t the purpose of this article. If you go to your Google Cloud Shell scripting you can enter in the following commands to create a Google Kubernetes Cluster ready for Helm.

```hs
$ git clone https://github.com/jonbcampos/kubernetes-series.git
$ cd ~/kubernetes-series/helm/scripts
$ sh startup.sh
```

This will take a moment to complete but when you’re done you’ll have a Kubernetes Cluster ready and waiting.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*SUlrHs6MkXkjWa11nA8xbA.png)

Your shiny new Kubernetes Cluster

## Second, Install Helm

With your Kubernetes Cluster up and running you can start adding in Helm. Below you will see the actual scripts (with notes) necessary to add a very basic Helm installation into GKE.

```hs
#!/usr/bin/env bash

echo "install helm"
# installs helm with bash commands for easier command line integration
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
# add a service account within a namespace to segregate tiller
kubectl --namespace kube-system create sa tiller
# create a cluster role binding for tiller
kubectl create clusterrolebinding tiller \
    --clusterrole cluster-admin \
    --serviceaccount=kube-system:tiller

echo "initialize helm"
# initialized helm within the tiller service account
helm init --service-account tiller
# updates the repos for Helm repo integration
helm repo update

echo "verify helm"
# verify that helm is installed in the cluster
kubectl get deploy,svc tiller-deploy -n kube-system
```

==Now that you’ve seen the code necessary, you can be lazy and just run a script to do the install for you.==

```hs
$ cd ~/kubernetes-series/helm/scripts
$ sh add_helm.sh
```

You’ll see everything run through your Shell console very quickly but in the end you should see the following lines, showing that Helm was installed completely.

```hs
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/tiller-deploy   1         1         1            0           1s

NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
svc/tiller-deploy   ClusterIP   10.11.244.223   <none>        44134/TCP   1s
```

You can even do a double check by running a basic Helm command and see the output.

```hs
$ helm ls # empty result as we haven't installed anything
```

## Third, Install A Chart

Now with Helm installed, let’s use it! In the scripts to follow we are going to install Redis into our Kubernetes Cluster with some production values recommended by the Redis Chart maintainers.

```hs
$ helm install stable/redis \
    --values values/values-production.yaml \
    --name redis-system
```

This will only take a moment but we can see that the Redis Chart was successfully deployed by running the following command.

```hs
$ helm ls
NAME            REVISION        UPDATED                         STATUS          CHART           NAMESPACE
redis-system    1               Thu Aug  9 11:02:23 2018        DEPLOYED        redis-3.7.5     default
```

If we give the Pods a moment to startup fully you can return to your `Kubernetes > Workloads` view and see the Pods all ready for you to interact with them.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*ZhUQKqIudmboADSqwX3n5g.png)

Your Redis workloads installed in GKE

That’s it! A production quality Redis instance installed and ready for development in just moments. This is huge! And all thanks to Kubernetes and Helm Charts.

## Profit!

There are more things that Helm can do through the command line interface. This article isn’t intended to be a compendium of Helm knowledge, but more of a starting point that gets you going on your Helm journey. I’m going to give some quick commands though that I’ve found helpful beyond the `install` command.

```hs
helm search
```

Allows you to search through the Helm Repo’s charts for a chart by name. Just a quick way to get you were you need.

```hs
helm list
```

As we’ve seen this will list the deployments you have within your Kubernetes Cluster that Helm is managing.

```hs
helm delete
```

Will delete a Helm Chart from your Kubernetes Cluster.

```hs
helm rollback
```

Will rollback an upgrade to your Helm Chart to a previous chart. This is super helpful if there were unintended consequences to an upgrade.

A full list of commands can be [found here](https://docs.helm.sh/helm).

## Conclusion

That’s it, you have the basics of getting going with Helm with Google Kubernetes Engine. I am already setting up the next installment of this series to go into how to secure your Helm installation with TLS. From the docs, my own struggles, and the amount of questions online around the subject I can assume more help here will be beneficial to everyone.

For now keep moving forward and make sure to share your feedback!

## Teardown

Before you leave make sure to cleanup your project so you aren’t charged for the VMs that you’re using to run your cluster. Return to the Cloud Shell and run the teardown script to cleanup your project. This will delete your cluster and the containers that we’ve built.

```hs
$ cd ~/kubernetes-series/helm/scripts
$ sh teardown.sh
```

## Other Posts In This Series## [Install Secure Helm In GKE](https://medium.com/@jonbcampos/install-secure-helm-in-gke-254d520061f7?source=post_page-----7f07f43c536e---------------------------------------)

In my last post I talked a lot about the joys of Helm and why you should spend the time installing it into your…

medium.com

[View original](https://medium.com/@jonbcampos/install-secure-helm-in-gke-254d520061f7?source=post_page-----7f07f43c536e---------------------------------------)## [Kubernetes: Running Background Tasks With Batch-Jobs](https://medium.com/@jonbcampos/kubernetes-running-background-tasks-with-batch-jobs-56482fbc853?source=post_page-----7f07f43c536e---------------------------------------)

When building amazing applications, there are times that you might want to handle an action outside of a user’s…

medium.com

[View original](https://medium.com/@jonbcampos/kubernetes-running-background-tasks-with-batch-jobs-56482fbc853?source=post_page-----7f07f43c536e---------------------------------------)## [Kubernetes: Run A Pod Per Node With Daemon Sets](https://medium.com/google-cloud/kubernetes-run-a-pod-per-node-with-daemon-sets-f77ce3f36bf1?source=post_page-----7f07f43c536e---------------------------------------)

My initial title to this article was just “Daemon Sets” with the assumption that it would be enough to get the point…

medium.com

[View original](https://medium.com/google-cloud/kubernetes-run-a-pod-per-node-with-daemon-sets-f77ce3f36bf1?source=post_page-----7f07f43c536e---------------------------------------)## [Kubernetes: Cron Jobs](https://medium.com/google-cloud/kubernetes-cron-jobs-455fdc32e81a?source=post_page-----7f07f43c536e---------------------------------------)

Sometimes your work isn’t transactional. Instead of waiting for a user to click a button and have systems light up we…

medium.com

[View original](https://medium.com/google-cloud/kubernetes-cron-jobs-455fdc32e81a?source=post_page-----7f07f43c536e---------------------------------------)## [Kubernetes: DNS Proxy With Services](https://medium.com/google-cloud/kubernetes-dns-proxy-with-services-d7d9e800c329?source=post_page-----7f07f43c536e---------------------------------------)

When building an application it is common that you’ll need to interact with external services to complete your business…

medium.com

[View original](https://medium.com/google-cloud/kubernetes-dns-proxy-with-services-d7d9e800c329?source=post_page-----7f07f43c536e---------------------------------------)## [Kubernetes: Routing Internal Services Through FQDN](https://medium.com/google-cloud/kubernetes-routing-internal-services-through-fqdn-d98db92b79d3?source=post_page-----7f07f43c536e---------------------------------------)

I remember when I was first getting into Kubernetes. Everything was new and shiny and about scale. As I continued…

medium.com

[View original](https://medium.com/google-cloud/kubernetes-routing-internal-services-through-fqdn-d98db92b79d3?source=post_page-----7f07f43c536e---------------------------------------)## [Kubernetes: Liveness Checks](https://medium.com/google-cloud/kubernetes-liveness-checks-4e73c631661f?source=post_page-----7f07f43c536e---------------------------------------)

Recently I put together a quick article about the Kubernetes Readiness Probe and how important it was for your cluster…

medium.com

[View original](https://medium.com/google-cloud/kubernetes-liveness-checks-4e73c631661f?source=post_page-----7f07f43c536e---------------------------------------)## [Kubernetes: Readiness Probe](https://itnext.io/kubernetes-readiness-probe-83f8a06d33d3?source=post_page-----7f07f43c536e---------------------------------------)

In case there was any question about this feature, I am writing about it specifically to state that this is not an…

itnext.io

[View original](https://itnext.io/kubernetes-readiness-probe-83f8a06d33d3?source=post_page-----7f07f43c536e---------------------------------------)## [Kubernetes: Horizontal Pod Scaling](https://medium.com/google-cloud/kubernetes-horizontal-pod-scaling-190e95c258f5?source=post_page-----7f07f43c536e---------------------------------------)

With Pod Autoscaling your Kubernetes Cluster can monitor the load of your existing Pods and determine if we need more…

medium.com

[View original](https://medium.com/google-cloud/kubernetes-horizontal-pod-scaling-190e95c258f5?source=post_page-----7f07f43c536e---------------------------------------)## [Kubernetes: Cluster Autoscaler](https://medium.com/google-cloud/kubernetes-cluster-autoscaler-f1948a0f686d?source=post_page-----7f07f43c536e---------------------------------------)

Autoscaling is a huge (and marketed) feature of Kubernetes. When your site/app/api/project makes it big and the flood…

medium.com

[View original](https://medium.com/google-cloud/kubernetes-cluster-autoscaler-f1948a0f686d?source=post_page-----7f07f43c536e---------------------------------------)## [Kubernetes: Day One](https://medium.com/google-cloud/kubernetes-day-one-30a80b5dcb29?source=post_page-----7f07f43c536e---------------------------------------)

This is the obligatory step one Kubernetes post. If you’re interested in Kubernetes you’ve probably read 100 of these…

medium.com

[View original](https://medium.com/google-cloud/kubernetes-day-one-30a80b5dcb29?source=post_page-----7f07f43c536e---------------------------------------)

Q uestions? Feedback? I’m very interested to hear what issues you might run across or if this helped you understand a bit better. If there is something I missed feel free to share that too. We are all in this together!

[Jonathan Campos](http://jonbcampos.com/) is an avid developer and fan of learning new things. I believe that we should always keep learning and growing and failing. I am always a supporter of the development community and always willing to help. So if you have questions or comments on this story please ad them below. Connect with me on [LinkedIn](https://www.linkedin.com/in/jonbcampos/) or [Twitter](https://twitter.com/jonbcampos) and mention this story.

[![Google Cloud - Community](https://miro.medium.com/v2/resize:fill:96:96/1*FUjLiCANvATKeaJEeg20Rw.png)](https://medium.com/google-cloud?source=post_page---post_publication_info--7f07f43c536e---------------------------------------)

[![Google Cloud - Community](https://miro.medium.com/v2/resize:fill:128:128/1*FUjLiCANvATKeaJEeg20Rw.png)](https://medium.com/google-cloud?source=post_page---post_publication_info--7f07f43c536e---------------------------------------)

[Last published just now](https://medium.com/google-cloud/creating-a-rick-morty-chatbot-with-google-cloud-and-the-gen-ai-sdk-e8108e83dbee?source=post_page---post_publication_info--7f07f43c536e---------------------------------------)

A collection of technical articles and blogs published or curated by Google Cloud Developer Advocates. The views expressed are those of the authors and don't necessarily reflect those of Google.

Excited developer and lover of pizza. CTO at Alto. Google Developer Expert.

## Responses (4)

Write a response[What are your thoughts?](https://medium.com/m/signin?operation=register&redirect=https%3A%2F%2Fmedium.com%2Fgoogle-cloud%2Finstalling-helm-in-google-kubernetes-engine-7f07f43c536e&source=---post_responses--7f07f43c536e---------------------respond_sidebar------------------)[Muhammad Taqi Abdul Aziz](https://medium.com/@svx.annexiv?source=post_page---post_responses--7f07f43c536e----0-----------------------------------)

[

Feb 27, 2022

](https://medium.com/@svx.annexiv/damn-this-was-good-article-d6bbb54fbf62?source=post_page---post_responses--7f07f43c536e----0-----------------------------------)

```hs
Damn, this was good article
```

```hs
script required update. Not a single script working in it current state
```

## More from Jonathan Campos and Google Cloud - Community

## Recommended from Medium

[

See more recommendations

](https://medium.com/?source=post_page---read_next_recirc--7f07f43c536e---------------------------------------)