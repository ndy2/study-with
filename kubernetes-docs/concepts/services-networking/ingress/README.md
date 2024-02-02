---
tags:
  - kubernetes
  - services-networking
title: Ingress
author: ndy2
date: 2024-02-02
description: ""
---
클러스터 내부의 서비스에 대한 외부 (HTTP) 접근을 관리하는 API object

> [!note] Terminology
> * Node - Kubernetes 의 Worker Machine, cluster 의 일부
> * Cluster - Kubernetes 에 의해 관리되는 중앙화된 애플리케이션을 실행하는 노드의 집합
> * Edge router - Cluster 에 대한 방화벽 정책을 강제하는 라우터. Cloud Provider 의 게이트웨이 혹은 물리적인 Hardware 로 구성 될 수 있다.
> * Cluster network - Kubernetes netorking model 에 따라 클러스터 내의 communication 을 가능하게 하는 논리적/물리적 링크의 집합
> * Service - Label selector 를 이용해 Pod 의 식별을 수행하는 Kubernetes Service.


# What is Ingress?

인그레스는 클러스터 외부의 HTTP, HTTPS 라우팅 요청을 클러스터 내부의 services 로 전달합니다. Traffic 의 라우팅은 ingress 에 정의된 rule 에 따라 결정됩니다.

HTTP, HTTPS 가 아닌 다른 서비스를 외부에 노출하는 것은 ingress 가 아닌 Service 를 통해 처리합니다.

# The Ingress resources

```yaml
# (1)
apiVersion: networking.k8s.io/v1

# (2)
kind: Ingress

# (3)
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /

# (4)
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```

1. apiVersion
2. resource 의 종류
3. metadata. ingress 의 annotation 에는 종종 ingressClass 별로 필요한 설정들이 정의됩니다.
4. ingress spec 은 load balancer/ proxy 서버를 설정하기 위한 모든 정보를 가집니다. 가장 중요한 것은, spec 은 모든 요청을 어떻게 처리할지에 대한 rule 이 포함된다는 것입니다.

spce 에 `ingressClassName` 이 생략되면 default ingress class 가 사용됩니다.


## Ingress rules

각 HTTP rule 은 다음 정보를 포함합니다.

- `spec.rules.http.host` - an optional host
위 예시에서, host 는 명시되지 않았습니다. 따라서 모든 inbound HTTP traffic 에 rule 이 적용됩니다.

- `spec.rules.http.paths` - paths 의 목록
path 는 service 를 backend 로 가집니다. service 는 `service.name` + `service.port.name` 혹은 `service.name` + `service.port.number` 로 정의 됩니다.

- rule 에 매칭되지 않는 요청을 처리하기 위해 `defaultBackend` 가 정의 될 수 있습니다.

## Backend

Backend 란 ingress 의 뒷단에서 요청을 받아 수행하는 주체를 의미합니다. 가장 기본적인 backend 는 위에서 살펴본 service backend 입니다. kubernetes 의 Service 리소스를 활용해 포트와 함께 정의 됩니다.

### Resource Backends

리소스 백엔드는 보통 정적인 자원 (resource) 를 제공하는 특별한 backend 로 defaultBackend 과 결합되어 활용되곤합니다.

## Path types

- Ingress 의 모든 path 는 pathType 을 가져야합니다.
- `ImplementationSpecific`/ `Exact`/ `Prefix`