---
layout: post
title: "OCI - Traffic Management"
date: 2019-02-19 09:00:00 +0900
description: "Announcing the Launch of Traffic Management"
img: "trafficmanagement1.png"
tags: ["oracle", "oci", "iaas", "cloud", "traffic", "management", "network", "steering", "dns", "failover","oracle cloud", "오라클 클라우드"] 
---

2019년 2월 Oracle Cloud Infrastructure에 대한 DNS 트래픽 관리 서비스가 출시되었습니다.
Oracle의 [업계 선두 DNS](https://blogs.oracle.com/cloud-infrastructure/introducing-dns-on-oracle-cloud-infrastructure)는 이미 전 세계에서 가장 신뢰할 수 있고 탄력있는 DNS 네트워크 입니다. 이제 customizable steering policy를 사용하여 endpoint availability 및 end-user 위치와 같은 요소를 기반으로 최적의 최종 사용자 환경을 제공할 수 있습니다. OCI DNS는 2개의 OCI Region에 트래픽을 분산하는 것과 같은 전통적인 글로벌 로드 밸런싱 기능 외에도, incoming 쿼리와 이러한 request의 경로 위치를 세밀하게 제어할 수 있게 되었습니다.
<br><br>
Traffic management steering policy는 simple point-A-to-point-B failover에서 가장 복잡한 엔터프라이즈 아키텍처에 이르기까지 다양한 사용 사례를 지원합니다. 이러한 지원을 통해 서비스 차별화, 지리적 시장 타겟팅 및 재해 복구 시나리오에 대한 예측 가능한 비즈니스 기대치를 설정할 수 있습니다. rule set는 글로벌 트래픽을 사용자가 제어할 수 있는 자산으로 보낼 수 있으므로 하이브리드 및 다중 클라우드 시나리오에서 사용자 경험 및 인프라 효율성을 최적화 할 수 있습니다.


# Traffic Management Steering Policies 가 필요한 이유

DNS의 중요한 구성 요소인 Traffic Management를 사용하면 DNS 쿼리에 지능형 응답을 제공하기 위한 라우팅 정책을 구성할 수 있습니다. customer-defined traffic management policy의 논리에 따라 쿼리에 대해 서로 다른 응답을 제공할 수 있으므로 사용자를 인프라의 최적 위치로 보낼 수 있습니다. OCI DNS는 인터넷의 현재 상태를 기반으로 최상의 라우팅 결정을 내리기 위해 최적화되어 있습니다.


# 일반적인 사용 사례

클라우드 및 하이브리드 아키텍처로 이동하는 기업이 늘어남에 따라 고가용성 및 재해 복구가 그 어느때보다 중요합니다. **DNS Failover**는 응답이 없을 경우 endpoint에서 트래픽을 이동시켜 대체 위치로 전송하는 기능을 제공 합니다. 가용성 상태는 [Oracle Health Check](https://blogs.oracle.com/cloud-infrastructure/external-health-checks-on-oracle-cloud-infrastructure)에서 모니터링되며 현재 OCI에서도 사용 가능합니다.
(한글 포스팅은 [여기](https://jesamkim.github.io/OCI_Healthcheck/)를 참조하세요.)

클라우드 기반 **DNS load balancing**은 여러 지리적인 이유로 인프라를 확장하는데 이상적입니다. 일반적인 사용 사례는 새로운 인프라 확장, 클라우드 마이그레이션, 사용자 기반의 새로운 기능 릴리스 제어 등이 포함됩니다. 모든 인프라(on-premises 또는 cloud-based)에서 endpoint의 "Pool"을 설정하고 비율 기반 가중치를 할당하는 것은 쉽습니다.

**Source-based steering**을 사용하면 요청이 어디서 왔는지에 따라 라우팅 결정을 자동화 할 수 있습니다. 이러한 source-based policy는 트래픽을 가장 가까운 지리적 자산으로 라우팅하고, 원래의 쿼리의 IP 접두어를 기반으로 트래픽을 제어하는 "split horizon"을 만들고, preferred providers를 기반으로 의사 결정을 내릴 수 있습니다.

![]({{site.baseurl}}/assets/img/trafficmanagement2.png)

*가장 가까운 지리적 위치로 사용자를 보내 다른 OCI region간에 트래픽을 로드 밸런싱 할 수 있습니다. 특정 region에서 자산에 도달할 수 없는 경우 health check을 사용하여 다른 region으로 사용자를 전환할 수도 있습니다.*


# 시작하기

[Basic DNS를 설정](https://docs.cloud.oracle.com/iaas/Content/DNS/Concepts/dnszonemanagement.htm)한 후, **Edge Services**에서 **Traffic Management Steering Policies**를 선택하면 자체 steering policies를 만들고 커스터마이즈 할 수 있습니다.



## 참고

이 포스트의 원문은 [Announcing the Launch of Traffic Management](https://blogs.oracle.com/cloud-infrastructure/announcing-the-launch-of-traffic-management) 입니다.
