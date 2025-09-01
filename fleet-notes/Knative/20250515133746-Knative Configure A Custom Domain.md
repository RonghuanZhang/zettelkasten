---
"type:": fleet-note
"title:": 20250515133746-Knative Configure A Custom Domain
"id:": 20250515133803
"created:": 2025-05-15T13:38:03
url: 
tags:
  - fleet-note
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---

## Step 1: Configure the domain to add the custom domain.

```shell
kubectl patch configmap/config-domain \
      --namespace knative-serving \
      --type merge \
      --patch '{"data":{"hiverun.io":""}}'
```

Or you can edit the config-domain configmap directly by using the edit cli.

```shell
kubectl edit configmap config-domain -n knative-serving
```

## Step 2: Get the IP of the Knative gateway

```shell
kubectl --namespace kourier-system get service kourier
```

![image.png](https://images.hnzhrh.com/note/20250515135607523.png)

## Step 3: Recognize the service EXTERNAL NAME.

```shell
kubectl get ksvc  
```

![image.png](https://images.hnzhrh.com/note/20250515135854884.png)

The EXTERNAL NAME pattern is:  `<service>.<namespaece>.<domain>`.

## Step 4: Verify the route.

```shell
kubectl get route
```

![image.png](https://images.hnzhrh.com/note/20250515140353334.png)

## Step 5: Curl from the client for a test.

You should add the HOST header to make sure the route is correct.

```shell
# No public domain map
curl -H "Host: <service-name>.<namespace>.<custome-domain>" http://<gateway external ip>
```

## Step 6: Configure the A record.

```shell
*.<custome-domain> ==> gateway external ip
```

## Step 7: Access the service by using the public domain.

```shell
curl http://<service>.<namespace>.<custom-domain>
```

# Reference
* [Configuring domain names - Knative](https://knative.dev/docs/serving/using-a-custom-domain/)
* [Configuring domain names - Knative](https://knative.dev/docs/serving/using-a-custom-domain/)