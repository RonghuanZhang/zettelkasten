---
"type:": fleet-note
"title:": 20250514143813-Install Knative On GKE
"id:": 20250514143826
"created:": 2025-05-14T14:38:26
url: 
tags:
  - fleet-note
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---

Version：1.18.0


## Install the Knative Serving Components

Check the file content before installing.

```shell
curl -LO https://github.com/knative/serving/releases/download/knative-v1.18.0/serving-crds.yaml

curl -LO https://github.com/knative/serving/releases/download/knative-v1.18.0/serving-core.yaml
```

Kubectl install.

```shell
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.18.0/serving-crds.yaml

# Output:
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
customresourcedefinition.apiextensions.k8s.io/certificates.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/configurations.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/clusterdomainclaims.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/domainmappings.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/ingresses.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/metrics.autoscaling.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/podautoscalers.autoscaling.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/revisions.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/routes.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/serverlessservices.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/services.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/images.caching.internal.knative.dev created
```


```shell

kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.18.0/serving-core.yaml


# Output：
namespace/knative-serving created
role.rbac.authorization.k8s.io/knative-serving-activator created
clusterrole.rbac.authorization.k8s.io/knative-serving-activator-cluster created
clusterrole.rbac.authorization.k8s.io/knative-serving-aggregated-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/knative-serving-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/knative-serving-namespaced-admin created
clusterrole.rbac.authorization.k8s.io/knative-serving-namespaced-edit created
clusterrole.rbac.authorization.k8s.io/knative-serving-namespaced-view created
clusterrole.rbac.authorization.k8s.io/knative-serving-core created
clusterrole.rbac.authorization.k8s.io/knative-serving-podspecable-binding created
serviceaccount/controller created
clusterrole.rbac.authorization.k8s.io/knative-serving-admin created
clusterrolebinding.rbac.authorization.k8s.io/knative-serving-controller-admin created
clusterrolebinding.rbac.authorization.k8s.io/knative-serving-controller-addressable-resolver created
serviceaccount/activator created
rolebinding.rbac.authorization.k8s.io/knative-serving-activator created
clusterrolebinding.rbac.authorization.k8s.io/knative-serving-activator-cluster created
customresourcedefinition.apiextensions.k8s.io/images.caching.internal.knative.dev unchanged
certificate.networking.internal.knative.dev/routing-serving-certs created
customresourcedefinition.apiextensions.k8s.io/certificates.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/configurations.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/clusterdomainclaims.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/domainmappings.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/ingresses.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/metrics.autoscaling.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/podautoscalers.autoscaling.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/revisions.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/routes.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/serverlessservices.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/services.serving.knative.dev unchanged
image.caching.internal.knative.dev/queue-proxy created
configmap/config-autoscaler created
configmap/config-certmanager created
configmap/config-defaults created
configmap/config-deployment created
configmap/config-domain created
configmap/config-features created
configmap/config-gc created
configmap/config-leader-election created
configmap/config-logging created
configmap/config-network created
configmap/config-observability created
configmap/config-tracing created
horizontalpodautoscaler.autoscaling/activator created
poddisruptionbudget.policy/activator-pdb created
deployment.apps/activator created
service/activator-service created
deployment.apps/autoscaler created
service/autoscaler created
deployment.apps/controller created
service/controller created
horizontalpodautoscaler.autoscaling/webhook created
poddisruptionbudget.policy/webhook-pdb created
deployment.apps/webhook created
service/webhook created
validatingwebhookconfiguration.admissionregistration.k8s.io/config.webhook.serving.knative.dev created
mutatingwebhookconfiguration.admissionregistration.k8s.io/webhook.serving.knative.dev created
validatingwebhookconfiguration.admissionregistration.k8s.io/validation.webhook.serving.knative.dev created
secret/webhook-certs created
```



### Check the Resource After Knative-serving Is Installed

```shell
kubectl get pods -n knative-serving
NAME                          READY   STATUS    RESTARTS   AGE
activator-58dbdbb654-g4xxg    1/1     Running   0          29m
autoscaler-d7cf99fdf-jdgsd    1/1     Running   0          29m
controller-785659d9fc-k829k   1/1     Running   0          29m
webhook-57bcff8f56-b9pvx      1/1     Running   0          29m

erpang@erpangdeMacBook-Pro gke-knative-install % kubectl get deployment -n knative-serving
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
activator    1/1     1            1           29m
autoscaler   1/1     1            1           29m
controller   1/1     1            1           29m
webhook      1/1     1            1           29m

erpang@erpangdeMacBook-Pro gke-knative-install % kubectl get sts -n knative-serving
No resources found in knative-serving namespace.

erpang@erpangdeMacBook-Pro gke-knative-install % kubectl get service -n knative-serving
NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                   AGE
activator-service            ClusterIP   34.118.234.53    <none>        9090/TCP,8008/TCP,80/TCP,81/TCP,443/TCP   29m
autoscaler                   ClusterIP   34.118.225.53    <none>        9090/TCP,8008/TCP,8080/TCP                29m
autoscaler-bucket-00-of-01   ClusterIP   34.118.227.171   <none>        8080/TCP                                  29m
controller                   ClusterIP   34.118.237.246   <none>        9090/TCP,8008/TCP                         29m
webhook                      ClusterIP   34.118.234.173   <none>        9090/TCP,8008/TCP,443/TCP                 29m

erpang@erpangdeMacBook-Pro gke-knative-install % kubectl get configmap -n knative-serving
NAME                     DATA   AGE
config-autoscaler        1      30m
config-certmanager       1      30m
config-defaults          1      30m
config-deployment        2      30m
config-domain            1      30m
config-features          1      30m
config-gc                1      30m
config-leader-election   1      30m
config-logging           1      30m
config-network           1      30m
config-observability     1      30m
config-tracing           1      30m
kube-root-ca.crt         1      30m

erpang@erpangdeMacBook-Pro gke-knative-install % kubectl get secret -n knative-serving
NAME            TYPE     DATA   AGE
webhook-certs   Opaque   3      29m

erpang@erpangdeMacBook-Pro gke-knative-install % kubectl get serviceaccount -n knative-serving
NAME         SECRETS   AGE
activator    0         34m
controller   0         34m
default      0         34m

erpang@erpangdeMacBook-Pro gke-knative-install % kubectl get role -n knative-serving
NAME                        CREATED AT
knative-serving-activator   2025-05-14T06:59:14Z

erpang@erpangdeMacBook-Pro gke-knative-install % kubectl get rolebinding -n knative-serving
NAME                        ROLE                             AGE
knative-serving-activator   Role/knative-serving-activator   45m

```


## Install the Network Layer.

I will install the Kourier for the Knative gateway.

```shell
kubectl apply -f https://github.com/knative/net-kourier/releases/download/knative-v1.18.0/kourier.yaml

#Output
namespace/kourier-system created
configmap/kourier-bootstrap created
configmap/config-kourier created
serviceaccount/net-kourier created
clusterrole.rbac.authorization.k8s.io/net-kourier created
clusterrolebinding.rbac.authorization.k8s.io/net-kourier created
deployment.apps/net-kourier-controller created
service/net-kourier-controller created
deployment.apps/3scale-kourier-gateway created
service/kourier created
service/kourier-internal created
horizontalpodautoscaler.autoscaling/3scale-kourier-gateway created
poddisruptionbudget.policy/3scale-kourier-gateway-pdb created


# Configure Knative Serving to use Kourier by default by running the command:
kubectl patch configmap/config-network \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"ingress-class":"kourier.ingress.networking.knative.dev"}}'
```

Fetch the EXTERNAL IP address or CNAME by the command:

```shell
kubectl --namespace kourier-system get service kourier
```

You can get the external IP and port.

```shell
NAME      TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
kourier   LoadBalancer   34.118.225.152   <EXTERNAL-IP>   80:32575/TCP,443:30133/TCP   21h
```

You can configure the domain by adding an A record to map the EXTERNAL-IP.

## NoDNS Method

```shell
kubectl patch configmap/config-domain \
      --namespace knative-serving \
      --type merge \
      --patch '{"data":{"svc.cluster.local":""}}'
```

# Reference
* [Install Serving with YAML - Knative](https://knative.dev/docs/install/yaml-install/serving/install-serving-with-yaml/#configure-dns)
* [Testing Knative on GKE Autopilot – William Denniss](https://wdenniss.com/testing-knative-on-gke-autopilot)
* [GitHub - meteatamel/knative-tutorial: A collection of samples for Knative Serving, Knative Eventing and Knative-GCP projects.](https://github.com/meteatamel/knative-tutorial)