---
"type:": source-note
"title:": 20250520164827-Kubernets Resource Group and Version
"id:": 20250520164917
"created:": 2025-05-20T16:49:17
source:
  - AI
url: 
tags:
  - source-note
  - kubernetes
  - kubernetes/restful-api
"processed:": false
"archived:": false
---

Below is a concise mapping between the URL path segments `GROUP` and `VERSION` in the Kubernetes API and the corresponding fields in your resource YAML.

**Summary:**  
In every Kubernetes manifest you write, there is an `apiVersion` field of the form

`apiVersion: <GROUP>/<VERSION>`
- **`<GROUP>`** is the API group name (e.g. `apps`, `batch`, `serving.knative.dev`).
- **`<VERSION>`** is the version within that group (e.g. `v1`, `v1beta1`).
For built-in “core” resources (Pods, Services, etc.), the group is omitted and you just write `apiVersion: v1`.

When defining a CustomResourceDefinition (CRD) itself, you specify the group under `spec.group` and the versions under `spec.versions[].name`.

## 1. Resource manifests: `apiVersion`

Every resource YAML you create or apply to the cluster begins with an `apiVersion` field. This single field actually encodes both the GROUP and VERSION:

```yaml
apiVersion: <GROUP>/<VERSION>   # e.g. "apps/v1" or "serving.knative.dev/v1"
kind: <Kind>                    # e.g. Deployment, Service, Knative Service, etc.
metadata:
  name: my-thing
  namespace: default
spec:
  …
```

* **GROUP**:

  * For built-in “core” resources (Pods, Namespaces, Services), you omit the group and use `apiVersion: v1`.
  * For extended or custom resources, you precede the slash with the group name—these often look like domain names (e.g. `networking.k8s.io`, `serving.knative.dev`) ([Kubernetes][1]).
* **VERSION**:

  * Indicates the stability level of the API (e.g. `v1alpha1`, `v1beta1`, `v1`) and sits after the slash .

---

## 2. Core vs. Named API Groups

### 2.1 Core (legacy) group

* **URL prefix:** `/api/v1/...`
* **Manifest:** `apiVersion: v1`
* **Example:**

  ```yaml
  apiVersion: v1
  kind: Pod
  ```
* **Group in URL:** omitted
* **Version in URL:** `v1` ([Kubernetes][1])

### 2.2 Named (extension) groups

* **URL prefix:** `/apis/<GROUP>/<VERSION>/...`
* **Manifest:** `apiVersion: <GROUP>/<VERSION>`
* **Examples:**

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  ```

  ```yaml
  apiVersion: serving.knative.dev/v1
  kind: Service
  ```
* **Group in URL:** `apps` or `serving.knative.dev`
* **Version in URL:** `v1`

---

## 3. Defining CRDs: `spec.group` & `spec.versions`

When you author a **CustomResourceDefinition** itself, you declare the group and supported versions explicitly in the CRD manifest:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: widgets.example.com
spec:
  group: example.com            # ← this is the GROUP
  versions:
    - name: v1                  # ← this is one VERSION
      served: true
      storage: true
    - name: v1beta1             # ← another VERSION if you need it
      served: true
      storage: false
  scope: Namespaced
  names:
    plural: widgets
    singular: widget
    kind: Widget
```

* **`spec.group`** → maps to the `<GROUP>` segment in `/apis/<GROUP>/<VERSION>`.
* **each `spec.versions[].name`** → corresponds to the `<VERSION>` segment. ([Kubernetes][2])

---

## 4. Putting it all together

| URL segment         | YAML field                   | Core vs. Named group             |
| ------------------- | ---------------------------- | -------------------------------- |
| `GROUP`             | `apiVersion` before slash    | empty for core; domain for named |
| `VERSION`           | `apiVersion` after slash     | e.g. `v1`, `v1beta1`             |
| for CRDs: `GROUP`   | `spec.group` in CRD manifest | —                                |
| for CRDs: `VERSION` | `spec.versions[].name`       | —                                |

Thus, **every** Kubernetes resource’s GROUP/VERSION in the HTTP path directly reflects the `apiVersion` in its YAML (or `spec.group` + `spec.versions[].name` for CRDs).

[1]: https://kubernetes.io/docs/reference/using-api/api-concepts/?utm_source=chatgpt.com "Kubernetes API Concepts"
[2]: https://kubernetes.io/docs/concepts/overview/kubernetes-api/?utm_source=chatgpt.com "The Kubernetes API"


# Reference