---
layout: post
title: "OCI Health Checks 소개"
date: 2019-02-18 12:00:00 +0900
description: "External Health Checks on Oracle Cloud Infrastructure"
img: "healthcheck1.png"
tags: ["oracle", "oci", "iaas", "cloud", "health", "check", "network", "ping", "http", "multicloud","oracle cloud", "오라클 클라우드"] 
---

퍼블릭 클라우드에서 솔루션을 실행할 때는 end user에 대한 서비스의 가용성과 성능을 모니터링하고, 호스트 클라우드 ***외부***로부터의 액세스를 확보하는 것이 중요 합니다. 이러한 요구를 충족시키기 위해 클라우드 프로바이더는 전 세계에 걸쳐 리전별 다양한 위치에서 모니터링을 제공해야 합니다.
<br><br>
OCI는 Oracle Cloud 또는 Hybrid 인프라에서 호스팅되는 public-facing 서비스의 가용성 및 성능을 모니터링하는데 Health Checks라는 서비스를 제공합니다.


# Health Checks 란?

External health checks를 통해 전 세계의 Oracle managed vantage points에서 사용자가 지정한 FQDN(fully qualified domain name) 또는 IP 주소에 이르기까지 다양한 프로토콜을 사용하여 scheduled testing을 수행할 수 있습니다. Health checks는 IP 주소 모니터링을 위해 HTTP 및 HTTPS 웹 어플리케이션 검사 및 TCP 및 ICMP 핑을 지원합니다. 또한 각 vantage point에서 10초마다 한 번씩 test를 실행하는 High-Availability 테스트를 선택할 수 있습니다. 일회성 유효성 검사 또는 트러블슈팅 테스트도 REST API를 통해 사용할 수 있습니다.

또한, Health Check 서비스는 [DNS Traffic Management](https://jesamkim.github.io/OCI_Traffic_Management/) 서비스와 완벽하게 통합되어 서비스 장애의 자동 탐지를 가능하게 하고, DNS 장애 이전을 트리거하여 서비스의 연속성을 보장합니다.


# Health Check 생성하기

1. **Edge Services** 메뉴에서, **Health Checks**로 이동하세요. Health Checks 영역에서 **Create Health Check**를 클릭하고 dialog box에 자세한 내용을 입력합니다.
![]({{site.baseurl}}/assets/img/healthcheck2.png)

2. 이 페이지로 돌아갈 때, 이 check의 목적을 기억할 수 있도록 name을 입력합니다.

3. check하고자 하는 target이 있는 Compartment를 선택합니다.

4. 모니터링 할 target endpoint를 추가 합니다. **Target** 필드는 이미 구성된 endpoint(instance 등)의 public IP 주소 중 선택할 수 있습니다. 이러한 endpoint 중 하나를 선택하여 모니터링 하거나 새로운 endpoint를 추가할 수 있습니다.

5. Target을 모니터링할 vantage points를 선택하세요. 이러한 vantage points는 전 세계에 위치하고 있으며 일반적으로 당신의 어플리케이션과 동일한 대륙에 위치한 vantage point를 선택하는 것이 좋습니다.

6. 실행하려는 테스트 유형을 웹페이지에 대해 HTTP 또는 HTTPS로 선택하거나 Public IP 주소에 대해 TCP 또는 ICMP를 선택하세요.

7. 테스트 빈도를 서비스에 필요한 모니터링 수준에 맞게 설정하세요. 현재 옵션에는 기본 테스트에는 매 30초 또는 60초가 포함되며 프리미엄 테스트는 매 10초마다 높은 빈도로 실행됩니다. 프리미엄 테스트를 위해 additional fee가 계산 됩니다.

8. 앞으로 이 체크를 빨리 검색하고 싶다면 Tag를 추가하세요.

9. **Create Health Check**을 클릭하세요.

Check가 생성된 후, 세부정보 페이지에 이 check에 대한 정보가 표시됩니다.

![]({{site.baseurl}}/assets/img/healthcheck4.png)


# Retrieving Metrics

Health Check 서비스는 [OCI Monitoring service](https://blogs.oracle.com/cloud-infrastructure/announcing-oracle-cloud-infrastructure-monitoring)에서 직접 미터법을 제공하여 입력 메트릭을 쿼리하고 보고서를 작성하여 외부 모니터링을 기반으로 경고를 구성할 수 있습니다. 이러한 통합을 기반으로 전 세계 각지에서 볼 수 있는 서비스 장애를 식별할 수 있는 유연성을 제공합니다.

full REST API를 사용하면 90일까지의 hustoric health check 모니터링 데이터에 액세스할 수 있습니다.

Health Check UI 내에서 각 테스트는 프로브의 측정 데이터에 대한 액세스를 제공합니다.

![]({{site.baseurl}}/assets/img/healthcheck3.png)


# Health Checks 중지하기

Health Check을 일시적으로 중지해야하는 경우 (예: 서비스를 유지관리 하거나 변경하려는 경우) 영향을 받는 항목을 선택하고 Health Check details 페이지에서 **Disable**을 클릭하여 정지할 수 있습니다.


# Next Step

Health Check은 유연한 UI나 REST API를 통해 간단하게 구성할 수 있습니다. 서비스를 사용하여 OCI에서만 호스트되는지 아니면 하이브리드 환경에 걸쳐서 제공 중인 경우 공개적으로 노출된 IP 주소나 FQDN을 모니터링 할 것을 권고 합니다.

Health Check는 더 많은 유형의 테스트를 다른 프로토콜에 걸쳐 더 많은 구성 옵션을 포함하도록 확장될 것 입니다.


## 참고

이 포스트의 원문은 [External Health Checks on Oracle Cloud Infrastructure](https://blogs.oracle.com/cloud-infrastructure/external-health-checks-on-oracle-cloud-infrastructure) 입니다.
