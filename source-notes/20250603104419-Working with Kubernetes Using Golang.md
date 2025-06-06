---
type: source-note
title: Working with Kubernetes Using Golang
id: 20250603100619
created: 2025-06-03T10:44:19
source:
  - web
url: https://medium.com/cloud-native-daily/working-with-kubernetes-using-golang-a3069d51dfd6
tags:
  - source-note
  - kubernetes
  - golang
processed: false
archived: false
---
[Sitemap](https://medium.com/sitemap/sitemap.xml)

[Mastodon](https://me.dm/@adeesh)## [Cloud Native Daily](https://medium.com/cloud-native-daily?source=post_page---publication_nav-dd347d5c5a2b-a3069d51dfd6---------------------------------------)

[Follow publication](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fcollection%2Fcloud-native-daily&operation=register&redirect=https%3A%2F%2Fmedium.com%2Fcloud-native-daily%2Fworking-with-kubernetes-using-golang-a3069d51dfd6&collection=Cloud+Native+Daily&collectionId=dd347d5c5a2b&source=post_page---publication_nav-dd347d5c5a2b-a3069d51dfd6---------------------publication_nav------------------)

[![Cloud Native Daily](https://miro.medium.com/v2/resize:fill:76:76/1*MoZ3voEYDtTTwoQ3azpccg.png)](https://medium.com/cloud-native-daily?source=post_page---post_publication_sidebar-dd347d5c5a2b-a3069d51dfd6---------------------------------------)

A blog for Devs and DevOps covering tips, tools, and developer stories about all things cloud-native

[Follow publication](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fcollection%2Fcloud-native-daily&operation=register&redirect=https%3A%2F%2Fmedium.com%2Fcloud-native-daily%2Fworking-with-kubernetes-using-golang-a3069d51dfd6&collection=Cloud+Native+Daily&collectionId=dd347d5c5a2b&source=post_page---post_publication_sidebar-dd347d5c5a2b-a3069d51dfd6---------------------post_publication_sidebar------------------)

![](https://miro.medium.com/v2/resize:fit:1400/0*7mhKLw0_8v0HVyXT)

ref: https://developers.redhat.com/blog/2020/12/16/create-a-kubernetes-operator-in-golang-to-automatically-manage-a-simple-stateful-application

These days Kubernetes is gaining popularity, as it is the best choice for deploying large-scale microservice applications. Kubernetes is the orchestration platform for containers.

The 3 main advantages of using Kubernetes are ***High Availability, high scalable and Disaster Recovery***. These are typical requirements for large-scale microservices applications. There are a lot of blogs, YouTube videos and documentation which describe its usage. In this article, I will briefly go over how we can interact with Golang.

- **Creating Simple Kubernetes ClientSet**  
	All the K8s resources can be accessed using the client set. In Go, we can generate two types of clientsets. **Normal client set** and **dynamic client set**. The normal clientset is generated from the Kubernetes API Swagger specification. It provides a strongly typed API with methods specific to each resource type, enabling easy and type-safe interaction with the API. However, it requires pre-defined client objects for each resource type.  
	The dynamic client set offers a more flexible and dynamic way to interact with the Kubernetes API. It provides a lower-level interface that allows you to interact with the API using dynamic objects and unstructured data.  
	The following code shows how to create the client set.
```hs
// There are two types of Client Set creation, interact from outside 
// one from inside

// ClientSet from Outside
config, err := clientcmd.BuildConfigFromFlags("", "<HomePath>/.kube/config")
if err != nil {
    return fmt.Errorf("Fail to build the k8s config. Error - %s", err)
}

// ClientSet from Inside
config, err := rest.InClusterConfig()
if err != nil {
    return fmt.Errorf("Fail to build the k8s config. Error - %s", err)
}

// build the client set
clientSet, err := kubernetes.NewForConfig(config)
if err != nil {
    return fmt.Errorf("Fail to create the k8s client set. Errorf - %s", err)
}

// inorder to create the dynamic Client set
dynamicClientSet, err := dynamic.NewForConfig(config)
if err != nil {
    return fmt.Errorf("Fail to create the dynamic client set. Errorf - %s", err)
}
```
- **Creating Kubernetes ClientSet With Service Account and Token  
	**Every time we can’t rely on the simple service client. We might need some Authentication mechanism in order to interact with the pod. There is one component in K8s architecture, called “Service Account”. It is an identity associated with a pod or a group of pods. It allows applications running within pods to authenticate and interact with the Kubernetes API server or other resources in the cluster.  
	Service accounts and their associated tokens allow applications running within pods to make API calls to the Kubernetes API server. This can be useful for interacting with the Kubernetes API to retrieve information about the cluster, create or manage resources, or perform other administrative tasks. This also provides secure communication between pods within the cluster. Applications can use the service account token to authenticate and authorize requests made to other pods or services running in the same cluster, ensuring secure and trusted communication  
	The following code shows how to create a service account and token which is associated with clientset.
```hs
import (
     corev1 "k8s.io/api/core/v1"
     rbacv1 "k8s.io/api/rbac/v1"
     metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
     "k8s.io/client-go/dynamic"
     "k8s.io/client-go/kubernetes"
     "k8s.io/client-go/rest"
     "k8s.io/client-go/tools/clientcmd"
)

func GetSecureClientSet() (*kubernetes.ClientSet, error){
// generate the client set
clientSet, err := // use the above code generate the simple client set

// Metadata for creating the ServiceAccount
 saRequest := &corev1.ServiceAccount{
  ObjectMeta: metav1.ObjectMeta{
   Name: "Name of service account",
  },
 }

 // Metadata for creating secret
 secretRequest := &corev1.Secret{
  ObjectMeta: metav1.ObjectMeta{
   Name: "Name of Secret",
   Annotations: map[string]string{
    "kubernetes.io/service-account.name": "Name of service account",
   },
  },
  Type: corev1.SecretTypeServiceAccountToken,
  Data: map[string][]byte{
   "token": []byte(generateToken()),
  },
 }

// We need to define the rule for the ClientSet associated Service account
// get the Role
role := getRoleObject()

// get the Role binding object
roleBinding := getRoleBinding()

// Context to consider
ctx := context.Background()

// Create the Role
 _, err := clientSet.RbacV1().Roles("namespace").Create(ctx, role, metav1.CreateOptions{})
 if err != nil {
  return fmt.Errorf("Found error, while creating the Role, err - %s", err)
 }

// Create Role binding
_, err = clientSet.RbacV1().RoleBindings("Namespace").Create(ctx, roleBinding, metav1.CreateOptions{})
if err != nil {
    return fmt.Errorf("Found error while creating the rolebinding. Error - %s", err)
}

// Now create the new client set with Service account information
// add the service account info to the config
config.Impersonate = rest.ImpersonationConfig{
  UserName: fmt.Sprintf("system:serviceaccount:%s:%s", "Namespace", "Service account name"),
}

// return client set with respect to new config
return kubernetes.NewForConfig(config), nil
}
```

Creating Roles and Role binding:

```hs
// Generate the Random 256 byte token
func generateToken() string {
 token := make([]byte, 256)
 _, err := rand.Read(token)
 if err != nil {
  logger.Errorf("Found error while generating the secret token - %v", err)
  panic(err)
 }

 base64EncodedToken := base64.StdEncoding.EncodeToString(token)
 return base64EncodedToken
}

// get Role Object
func getRoleObject() *rbacv1.Role {
 return &rbacv1.Role{
  ObjectMeta: metav1.ObjectMeta{
   Name:      "my-role", // Name of the Role resource
   Namespace: c.Namespace,
  },
  Rules: []rbacv1.PolicyRule{
   {
    APIGroups: []string{"", "apps"}, // add required API groups
    Resources: []string{"*"}, // add required resources
    Verbs:     []string{"create", "get", "update", "delete", "list", "post"}, // Add all the operations
   },
  },
 }
}

// get RoleBinding Object
func getRoleBindingObject() *rbacv1.RoleBinding {
 return &rbacv1.RoleBinding{
  ObjectMeta: metav1.ObjectMeta{
   Name:      "deployment-creator-binding", // name of the role binding Resource
   Namespace: "Namespace",
  },
  RoleRef: rbacv1.RoleRef{
   APIGroup: rbacv1.GroupName,
   Kind:     "Role",
   Name:     "my-role", // This should be the name of created role
  },
  Subjects: []rbacv1.Subject{
   {
    Kind:      "ServiceAccount",
    Name:      "Service account name", // Name of the service account
    Namespace: "namespace", // Name of the Namespace
   },
  },
 }
}
```
- **Apply any type of Kubernetes config file  
	**Usually, when we want to apply any Kubernetes resources we use the `kubectl` tool. i.e. `kubectl apply -f config.yaml`. Config.yaml can contain any type of Kubernetes resources. The same can be achieved using the Go-Kubernetes dynamic client. The following code shows how we can create a dynamic client.
```hs
// Get the generic dynamic Resource Client
func getGenericResourceClient(resourceType string, group string, version string) dynamic.ResourceInterface {
 genericSchema := schema.GroupVersionResource{
  Group:    group,
  Version:  version,
  Resource: resourceType,
 }

  dynamicClient = // Generate the dynamic Client from above code.
   
  // Attach the given resource and schema
  return dynamicClient.Resource(genericSchema).Namespace("Namespace")
}

// Main method to deploy any type of kubernets kind
func genericDeployment(resourceType string, resourceName string, resourceObject runtime.Object, group string, version string) error {
  // Context to consider
Ctx = context.Background()

// 1. Get the dynamic resouce client
resourceClient := getGenericResourceClient(resourceType, group, version)
 
// 2. convert runtimeObject to unstructured
 unstructuredObj, err := runtime.DefaultUnstructuredConverter.ToUnstructured(resourceObject)
 if err != nil {
  return fmt.Errorf("Found error while coverting resource to unstructured err - %s", err)
 }
 unstructuredResource := &unstructured.Unstructured{Object: unstructuredObj}

 // 3. try to see if the resource exists
 existingResource, err := resourceClient.Get(c.Ctx, resourceName, metav1.GetOptions{})
 if err != nil
  if errors.IsNotFound(err) {
   // Resource doesn't exist, so create one
   _, err = resourceClient.Create(Ctx, unstructuredResource, metav1.CreateOptions{})
   if err != nil {
    return fmt.Errorf("Found error while creating the Resource : %s, err - %s", resourceName, err)
   }
  } else {
   return fmt.Errorf("Found error initialising the client : %s, err - %s", resourceType, err)
  }
 } else {
  // Resource already exists, so update the existing resource
  existingResource.Object = unstructuredObj
  _, err := resourceClient.Update(Ctx, existingResource, metav1.UpdateOptions{})
  if err != nil {
   return fmt.Errorf("Found error while Updating %s, - err : %s", resourceType, err)
  }
 }
 return nil
}
```

The following method provides us with both Group and version information by providing the **Kind** name. This will search for all the resource and returns once it gets the given Kind Name.

```hs
// getGroupVersionForResourceType returns the schema.GroupVersion, which
// contains both group and version, for given Kind name
func getGroupVersionFromKind(kindName string) (schema.GroupVersion, error) {
 discoveryClient := discovery.NewDiscoveryClientForConfigOrDie(c.RestConfig)

 apiResourceLists, err := discoveryClient.ServerPreferredResources()
 if err != nil {
  return schema.GroupVersion{}, err
 }

 for _, apiResourceList := range apiResourceLists {
  for _, resource := range apiResourceList.APIResources {
   if resource.Kind == kindName {
    group, version := getGroupVersion(apiResourceList.GroupVersion)
    return schema.GroupVersion{Group: group, Version: version}, nil
   }
  }
 }
 return schema.GroupVersion{}, nil
}

// Split into group and version
func getGroupVersion(groupVersion string) (string, string) {
 if strings.Contains(groupVersion, "/") {
  arr := strings.Split(groupVersion, "/")
  return arr[0], arr[1]
 }
 return "", groupVersion
}
```
- **Retrieve the Status of Kubernetes resource  
	**Once we have the Generic Dynamic client, we can perform any sort of query to any resource. Here’s how we get the status of the Kubernetes resource which was just deployed.
```hs
func getDeploymentStatus(resourceType string, group string, version string, deploymentName string){
// Context to consider
Ctx := context.Background()
 
 // Get the generic Dynamic client
  resourceClient := getGenericResourceClient(resourceType, group, version)
 status, err := resourceClient.Get(Ctx, deploymentName, metav1.GetOptions{})
 if err != nil {
  return Failed, err
 }
  // Status of the deployment
  fmt.Printf("%#v", status)
}
```

## Further Reading:## [Kubernetes Monitoring with OpenTelemetry](https://gethelios.dev/blog/kubernetes-monitoring-opentelemetry/?utm_source=medium&utm_medium=referral&utm_campaign=cloud+native+daily&source=post_page-----a3069d51dfd6---------------------------------------)

Learn how to monitor Kubernetes using OpenTelemetry with real-time visibility and granular error data - Reduce MTTR by…

gethelios.dev

[View original](https://gethelios.dev/blog/kubernetes-monitoring-opentelemetry/?utm_source=medium&utm_medium=referral&utm_campaign=cloud+native+daily&source=post_page-----a3069d51dfd6---------------------------------------)## [Helping Go teams implement OpenTelemetry - A new approach](https://gethelios.dev/blog/helping-go-teams-implement-opentelemetry-a-new-approach/?utm_source=medium&utm_medium=cloud+native+daily&source=post_page-----a3069d51dfd6---------------------------------------)

Here's how to implement OTel with Go, using Helios' new approach to OTel Go instrumentation - non-intrusive and easy to…

gethelios.dev

[View original](https://gethelios.dev/blog/helping-go-teams-implement-opentelemetry-a-new-approach/?utm_source=medium&utm_medium=cloud+native+daily&source=post_page-----a3069d51dfd6---------------------------------------)## [Golang Distributed Tracing - OpenTelemetry Based Observability -](https://gethelios.dev/blog/golang-distributed-tracing-opentelemetry-based-observability/?utm_source=medium&utm_medium=cloud+native+daily&source=post_page-----a3069d51dfd6---------------------------------------)

Read about the challenges of integrating OpenTelemetry (OTel) Golang tracing and how to make the process faster and…

gethelios.dev

[View original](https://gethelios.dev/blog/golang-distributed-tracing-opentelemetry-based-observability/?utm_source=medium&utm_medium=cloud+native+daily&source=post_page-----a3069d51dfd6---------------------------------------)

[![Cloud Native Daily](https://miro.medium.com/v2/resize:fill:96:96/1*MoZ3voEYDtTTwoQ3azpccg.png)](https://medium.com/cloud-native-daily?source=post_page---post_publication_info--a3069d51dfd6---------------------------------------)

[![Cloud Native Daily](https://miro.medium.com/v2/resize:fill:128:128/1*MoZ3voEYDtTTwoQ3azpccg.png)](https://medium.com/cloud-native-daily?source=post_page---post_publication_info--a3069d51dfd6---------------------------------------)

[Last published Apr 4, 2025](https://medium.com/cloud-native-daily/serverless-video-transcoding-on-aws-using-lambda-mediaconvert-and-cloudfront-b4103ec16d49?source=post_page---post_publication_info--a3069d51dfd6---------------------------------------)

A blog for Devs and DevOps covering tips, tools, and developer stories about all things cloud-native

Hi there!!. I’m a Software Developer (mostly backend development), I often post blogs whenever I discover some unique feature/ idea

## More from Adeesh Acharya and Cloud Native Daily

## Recommended from Medium

[

See more recommendations

](https://medium.com/?source=post_page---read_next_recirc--a3069d51dfd6---------------------------------------)