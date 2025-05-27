---
"type:": fleet-note
"title:": 20250526200548-K8s Go Client Access Knative
id:: 20250526200557  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-05-26T20:05:57  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---
To use the Kubernetes Go client (`client-go`) to access Knative resources, you need to understand that Knative resources (like `Service`, `Route`, `Revision`, etc.) are implemented as Custom Resource Definitions (CRDs) in Kubernetes. Accessing them from Go can be done in two main ways:

### 1. Using the Dynamic Client

Knative resources are not part of the core Kubernetes API, so you can use the dynamic client to interact with arbitrary CRDs.

**Example: Accessing a Knative Service using the dynamic client**
```go
import (
    "context"
    "fmt"
    "os"

    "k8s.io/client-go/dynamic"
    "k8s.io/client-go/tools/clientcmd"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/runtime/schema"
)

func main() {
    kubeconfig := os.Getenv("KUBECONFIG")
    config, err := clientcmd.BuildConfigFromFlags("", kubeconfig)
    if err != nil {
        panic(err)
    }

    dynClient, err := dynamic.NewForConfig(config)
    if err != nil {
        panic(err)
    }

    // Knative Service GVR
    gvr := schema.GroupVersionResource{
        Group:    "serving.knative.dev",
        Version:  "v1",
        Resource: "services",
    }

    // List Knative services in a namespace
    svcList, err := dynClient.Resource(gvr).Namespace("default").List(context.TODO(), metav1.ListOptions{})
    if err != nil {
        panic(err)
    }

    for _, svc := range svcList.Items {
        fmt.Println("Knative Service:", svc.GetName())
    }
}
```
- Change the `Group`, `Version`, and `Resource` values to match the Knative resource you want.

### 2. Using Generated Knative Client Libraries

Knative projects publish Go clients for their resources, e.g., `knative.dev/serving/pkg/client/clientset/versioned`.

**Example: Using the Knative Serving clientset**
```go
import (
    "context"
    "fmt"
    "os"

    knativeclientset "knative.dev/serving/pkg/client/clientset/versioned"
    "k8s.io/client-go/tools/clientcmd"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

func main() {
    kubeconfig := os.Getenv("KUBECONFIG")
    config, err := clientcmd.BuildConfigFromFlags("", kubeconfig)
    if err != nil {
        panic(err)
    }

    knClient, err := knativeclientset.NewForConfig(config)
    if err != nil {
        panic(err)
    }

    svcList, err := knClient.ServingV1().Services("default").List(context.TODO(), metav1.ListOptions{})
    if err != nil {
        panic(err)
    }

    for _, svc := range svcList.Items {
        fmt.Println("Knative Service:", svc.Name)
    }
}
```
- Install the client with:  
  ```
  go get knative.dev/serving/pkg/client
  ```

### Summary

- **For generic access:** Use the [dynamic client](https://pkg.go.dev/k8s.io/client-go/dynamic).
- **For typed access:** Use the Knative Go client libraries.

#### References
- [Knative Go Client Example](https://github.com/knative/client/blob/main/docs/coding-guidelines.md)
- [Kubernetes Dynamic Client Example](https://github.com/kubernetes/client-go/blob/master/examples/dynamic-create-update-delete/main.go)
- [Knative Serving Go Client](https://pkg.go.dev/knative.dev/serving/pkg/client/clientset/versioned)

Let me know if you want a full working sample or more details on a specific Knative resource!

# Reference