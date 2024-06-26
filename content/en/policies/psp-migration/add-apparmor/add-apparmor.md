---
title: "Add AppArmor Annotations"
category: PSP Migration
version: 
subject: Pod,Annotation
policyType: "mutate"
description: >
    In the earlier Pod Security Policy controller, it was possible to define a setting which would enable AppArmor for all the containers within a Pod so they may be assigned the desired profile. Assigning an AppArmor profile, accomplished via an annotation, is useful in that it allows secure defaults to be defined and may also result in passing other validation rules such as those in the Pod Security Standards. This policy mutates Pods to add an annotation for every container to enabled AppArmor at the runtime/default level.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//psp-migration/add-apparmor/add-apparmor.yaml" target="-blank">/psp-migration/add-apparmor/add-apparmor.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-apparmor-annotations
  annotations:
    policies.kyverno.io/title: Add AppArmor Annotations
    policies.kyverno.io/category: PSP Migration
    policies.kyverno.io/subject: Pod,Annotation
    kyverno.io/kyverno-version: 1.10.0
    kyverno.io/kubernetes-version: "1.24"
    pod-policies.kyverno.io/autogen-controllers: none
    policies.kyverno.io/description: >-
      In the earlier Pod Security Policy controller, it was possible to define
      a setting which would enable AppArmor for all the containers within a Pod so
      they may be assigned the desired profile. Assigning an AppArmor profile, accomplished
      via an annotation, is useful in that it allows secure defaults to be defined and may
      also result in passing other validation rules such as those in the Pod Security Standards.
      This policy mutates Pods to add an annotation for every container to enabled AppArmor
      at the runtime/default level.
spec:
  rules:
  - name: apparmor-runtime-default
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
          - CREATE
          - UPDATE
    mutate:
      foreach:
      - list: request.object.spec.[ephemeralContainers, initContainers, containers][]
        patchStrategicMerge:
          metadata:
            annotations:
              container.apparmor.security.beta.kubernetes.io/{{element.name}}: runtime/default
```
