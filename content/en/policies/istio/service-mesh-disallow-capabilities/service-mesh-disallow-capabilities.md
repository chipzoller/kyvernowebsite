---
title: "Service Mesh Disallow Capabilities"
category: Istio, Linkerd, Pod Security Standards (Baseline)
version: 
subject: Pod
policyType: "validate"
description: >
    This policy is a variation of the disallow-capabilities policy that is a part of the Pod Security Standards (Baseline) category. It enforces the same control but with provisions for common service mesh initContainers from Istio and Linkerd which need the additional capabilities, NET_ADMIN and NET_RAW. For more information and context, see the Kyverno blog post at https://kyverno.io/blog/2024/02/04/securing-services-meshes-easier-with-kyverno/.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//istio/service-mesh-disallow-capabilities/service-mesh-disallow-capabilities.yaml" target="-blank">/istio/service-mesh-disallow-capabilities/service-mesh-disallow-capabilities.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: service-mesh-disallow-capabilities
  annotations:
    policies.kyverno.io/title: Service Mesh Disallow Capabilities
    policies.kyverno.io/category: Istio, Linkerd, Pod Security Standards (Baseline)
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.12.3
    kyverno.io/kubernetes-version: "1.28"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      This policy is a variation of the disallow-capabilities policy that is a part of the
      Pod Security Standards (Baseline) category. It enforces the same control but with
      provisions for common service mesh initContainers from Istio and Linkerd which need
      the additional capabilities, NET_ADMIN and NET_RAW. For more information and context,
      see the Kyverno blog post at https://kyverno.io/blog/2024/02/04/securing-services-meshes-easier-with-kyverno/.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: adding-capabilities-istio-linkerd
      match:
        any:
        - resources:
            kinds:
              - Pod
      preconditions:
        all:
        - key: "{{ request.operation || 'BACKGROUND' }}"
          operator: NotEquals
          value: DELETE
      context:
        - name: capabilities
          variable:
            value: ["AUDIT_WRITE","CHOWN","DAC_OVERRIDE","FOWNER","FSETID","KILL","MKNOD","NET_BIND_SERVICE","SETFCAP","SETGID","SETPCAP","SETUID","SYS_CHROOT"]
      validate:
        message: >-
          Any capabilities added beyond the allowed list (AUDIT_WRITE, CHOWN, DAC_OVERRIDE, FOWNER,
          FSETID, KILL, MKNOD, NET_BIND_SERVICE, SETFCAP, SETGID, SETPCAP, SETUID, SYS_CHROOT)
          are disallowed. Service mesh initContainers may additionally add NET_ADMIN and NET_RAW.
        foreach:
          - list: request.object.spec.initContainers[]
            preconditions:
              all:
              - key: "{{ element.image }}"
                operator: AnyIn
                value:
                - "*/istio/proxyv2*"
                - "*/linkerd/proxy-init*"
              - key: "{{ element.securityContext.capabilities.add[] || `[]` }}"
                operator: AnyNotIn
                value:
                  - NET_ADMIN
                  - NET_RAW
                  - "{{ capabilities }}"
            deny:
              conditions:
                all:
                - key: "{{ element.securityContext.capabilities.add[] || `[]` }}"
                  operator: AnyNotIn
                  value: "{{ capabilities }}"
                  message: The service mesh initContainer {{ element.name }} is attempting to add forbidden capabilities.
          - list: request.object.spec.initContainers[]
            preconditions:
              all:
              - key: "{{ element.image }}"
                operator: AnyNotIn
                value:
                - "*/istio/proxyv2*"
                - "*/linkerd/proxy-init*"
            deny:
              conditions:
                all:
                - key: "{{ element.securityContext.capabilities.add[] || `[]` }}"
                  operator: AnyNotIn
                  value: "{{ capabilities }}"
                  message: The initContainer {{ element.name }} is attempting to add forbidden capabilities.
          - list: request.object.spec.[ephemeralContainers, containers][]
            deny:
              conditions:
                all:
                - key: "{{ element.securityContext.capabilities.add[] || `[]` }}"
                  operator: AnyNotIn
                  value: "{{ capabilities }}"
                  message: The container {{ element.name }} is attempting to add forbidden capabilities.
```
