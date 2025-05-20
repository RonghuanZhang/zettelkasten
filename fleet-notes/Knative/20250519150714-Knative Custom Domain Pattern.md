---
"type:": fleet-note
"title:": 20250519150714-Knative Custom Domain Pattern
id:: 20250519150728  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-05-19T15:07:28  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---
```shell
kubectl get configmap config-network -o yaml -n knative-serving        
```

```yaml
    # domain-template specifies the golang text template string to use
    # when constructing the Knative service's DNS name. The default
    # value is "{{.Name}}.{{.Namespace}}.{{.Domain}}".
    #
    # Valid variables defined in the template include Name, Namespace, Domain,
    # Labels, and Annotations. Name will be the result of the tag-template
    # below, if a tag is specified for the route.
    #
    # Changing this value might be necessary when the extra levels in
    # the domain name generated is problematic for wildcard certificates
    # that only support a single level of domain name added to the
    # certificate's domain. In those cases you might consider using a value
    # of "{{.Name}}-{{.Namespace}}.{{.Domain}}", or removing the Namespace
    # entirely from the template. When choosing a new value be thoughtful
    # of the potential for conflicts - for example, when users choose to use
    # characters such as `-` in their service, or namespace, names.
    # {{.Annotations}} or {{.Labels}} can be used for any customization in the
    # go template if needed.
    # We strongly recommend keeping namespace part of the template to avoid
    # domain name clashes:
    # eg. '{{.Name}}-{{.Namespace}}.{{ index .Annotations "sub"}}.{{.Domain}}'
    # and you have an annotation {"sub":"foo"}, then the generated template
    # would be {Name}-{Namespace}.foo.{Domain}
    domain-template: "{{.Name}}.{{.Namespace}}.{{.Domain}}"
```



# Reference