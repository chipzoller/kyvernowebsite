---
title: "Check for matching VerticalPodAutoscaler (VPA)"
category: Other
version: 
subject: Deployment, StatefulSet, ReplicaSet, DaemonSet, VerticalPodAutoscaler
policyType: "validate"
description: >
    VerticalPodAutoscaler (VPA) is useful to automatically adjust the resources assigned to Pods.  It requires defining a specific target resource by kind and name. There are no built-in  validation checks by the VPA controller to ensure that the target resource is associated with it.  This policy ensures that the matching kind has a matching VPA. 
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/check-vpa-configuration/check-vpa-configuration.yaml" target="-blank">/other/check-vpa-configuration/check-vpa-configuration.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-vpa-configuration
  annotations:
    policies.kyverno.io/title: Check for matching VerticalPodAutoscaler (VPA)
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.11.4 
    kyverno.io/kubernetes-version: "1.27"     
    policies.kyverno.io/subject: Deployment, StatefulSet, ReplicaSet, DaemonSet, VerticalPodAutoscaler
    policies.kyverno.io/description: >-
      VerticalPodAutoscaler (VPA) is useful to automatically adjust the resources assigned to Pods. 
      It requires defining a specific target resource by kind and name. There are no built-in 
      validation checks by the VPA controller to ensure that the target resource is associated with it. 
      This policy ensures that the matching kind has a matching VPA. 
spec:
  validationFailureAction: Audit
  background: false
  rules:
    - name: check-vpa-configuration
      match:
        any:
        - resources:
            kinds:
              - Deployment
              - StatefulSet
              - ReplicaSet
              - DaemonSet
      context:
        - name: vpas
          apiCall:
            urlPath: "/apis/autoscaling.k8s.io/v1/namespaces/{{request.object.metadata.namespace}}/verticalpodautoscalers"
            jmesPath: "items[?spec.targetRef.kind=='{{ request.object.kind }}'].spec.targetRef.name"
      validate:
        message: >-
          Workload '{{request.object.kind}}/{{request.object.metadata.name}}' 
          requires a matching VerticalPodAutoscaler (VPA) in the 
          '{{request.object.metadata.namespace}}' namespace.
        deny:
          conditions:
            all:
            - key: "{{ request.object.metadata.name }}" 
              operator: NotIn 
              value: "{{ vpas }}" 
```
