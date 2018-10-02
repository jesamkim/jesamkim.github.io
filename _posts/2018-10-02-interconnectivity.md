---
layout: post
title: "[번역] Oracle Cloud와 타 클라우드의 연결"
date: 2018-10-02 10:00:00 +0900
description: "Oracle Cloud와 타 클라우드(AWS,Azure 등) 간 연결" # Add post description (optional)
img: blog_interconnectivity_1.png # Add image post (optional)
tags: [oracle, oci, iaas, cloud, interconnect, fastconnect, dedicated, megaport, network] # add tag
---

 멀티 클라우드 아키텍처는 둘 이상의 클라우드 서비스 프로바이더를 이용 합니다. 기업은 복원력 제공, 재해 복구 계획 수립, 성능 향상, 비용 절감 등 여러 가지 이유로 클라우드 프로바이더가 두 개 이상 있습니다. 클라우드 리소스를 한 클라우드 프로바이더에서 다른 클라우드 프로바이더로 마이그레이션하려는 경우 클라우드 간 액세스 및 네트워킹이 필요합니다.
 Oracle Cloud Infrastructure는 Oracle VCN(Virtual Cloud Network)을 인터넷, 온프레미스 데이터센터 또는 기타 클라우드 프로바이더와 연결하기 위한 IGW(Internet Gateway) 및 DRG(Dynamic Routing Gateway) 서비스 게이트웨이 옵션을 제공합니다.  

 이 포스트에서는 Oracle Cloud에 대한 네트워크 연결을 일반적으로 계획하는데 도움이 되는 연결 서비스 옵션에 대해 설명하고 클라우드 프로바이더 간의 연결 옵션에 대해 설명합니다.  


 # Connectivity Option Overview

 모든 주요 클라우드 서비스 프로바이더(CSP)는 세 가지 고유한 네트워크 연결 서비스 옵션을 제공합니다:  
    - Public internet  
    - IPSec VPN  
    - Dedicated connectrions (오라클 서비스는 Oracle Cloud Infrastructure FastConnect라고 합니다)  


워크로드 및 전송해야 하는 데이터 양에 따라 세 가지 네트워크 연결 서비스 옵션이 모두 필요합니다.  

||Max (Mb/s)|Latency|Jitter|Cost|Secure|
|---|---|---|---|---|---|
|Public internet|< 10,000|Variable|Variable|Variable|No|
|IPSec VPN|< 250|Variable|Variable|Variable|Yes|
|FastConnetc|< 100,000|Predictable|Predictable|Predictable|Yes|

    - Public internet은 인터넷에 연결된 모든 장치에서 액세스 할 수 있는 기능을 제공합니다.  
    - IPSec VPN 은 네트워크를 클라우드로 확장하여 액세스를 제공하는 보안 암호화 네트워크 입니다.  
    - FastConnect는 전용 연결을 제공하고 인터넷에 대한 대체 연결을 제공합니다. 이 서비스의 독점적 특성으로 인해 보다 안정적이고 짧은 지연 시간, 전용 대역폭 및 보안 액세스를 제공합니다.  


FastConnect는 다음과 같은 [연결 모델](https://cloud.oracle.com/en_US/fastconnect/connectivity-models)을 제공합니다:  
    - Oracle 네트워크 프로바이더 또는 Exchange partner를 통한 연결  
    - 데이터센터 내에서 직접 피어링을 통한 연결  
    - Third-party 네트워크의 전용 회로를 통한 연결  


# Connectivity Option Details

다음은 최적의 연결 옵션 입니다. 속도, 비용 및 시간을 기준으로 옵션을 비교하려면 "Choosing Your Connectivity Option" 섹션을 참조하십시오.  

## Option 1: IPSec VPN을 통한 연결

IPSec VPN은 데이터 트래픽을 암호화하여 보안을 추가합니다. VPN을 통해 대역폭은 250Mbps로 제한됩니다. 따라서 전송할 총 데이터 양과 필요한 전송 속도에 따라 VPN 터널이 여러 개 필요할 수 있습니다.  

Oracle Cloud Infrastructure와 기타 클라우드 프로바이더 간의 보안 연결을 생성하기 위한 단계별 치침은 [Secure Connection between Oracle and Other Cloud Providers](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/libreswan.htm?tocpath=Services%7CNetworking%7C_____14) 에서 확인할 수 있습니다.  


## Option 2: Cloud Exchange를 통한 연결

Exchange 프로바이더는 온프레미스와 Exchange 프로바이더 간의 동일한 전용 물리적 연결을 통해 클라우드 프로바이더의 대규모 에코 시스템에 대한 연결을 제공할 수 있습니다. 사용할 수 있는 프로바이더로는 Megaport, Equinix 및 Digital Realty가 있습니다.  

클라우드 간에 라우팅 할 수 있는 옵션은 다음과 같습니다:  
    - Exchange 프로바이더의 가상 라우터 서비스 - 예 : Megaport Cloud Router (MCR)  
    - Physical customer edge (CE) 장치를 Exchange 프로바이더와 연결

다음 테이블은 위 2가지 경우에 대한 장단점 입니다.
||Pros|Cons|
|---|---|---|
|Virtual router service 사용|* 간편한 구축<br>* 온디맨드 대역폭 제공<br>*구축 및 유지보수 비용 효율|* Cloud Exchange 지원 범위에 포함되는 라우팅 경로의 유연성<br>* Public IP Communication 불가|
|Dedicated physical router 사용|*라우팅 기능의 유연한 관리<br>*원하는 하드웨어를 구현할 수 있는 기능 제공|*비교적 긴 배치 시간<br>* 확장 제한<br>*하드웨어 유지보수 비용|

이 포스트의 범위는 파트너에 관계 없이 최적의 연결 옵션을 제공하는 것이지만 가상 라우터 서비스를 쉽게 구축하고 제공할 수 있기 때문에 예로서 MCR(Megaport Cloud Router) 옵션을 사용합니다. Oracle은 또한 AWS(Amazon Web Services)를 예시로 사용하고 있지만, Megaport는 Azure 및 Google Cloud Platform을 비롯한 여러 클라우드 프로바이터에 대한 연결을 지원합니다.

![]({{site.baseurl}}/assets/img/blog_interconnectivity_1.png)

연결 설정에는 다음 단계가 포함됩니다:
    1. Oracle Cloud Infrastructure 콘솔을 통해 FastConnect를 Megaport에 연결합니다.  
    2. AWS 콘솔을 통해 AWS Direct Connect를 Megaport와 연결합니다.  
    3. MCR을 생성합니다:  
        a. MCR에서 FastConnect에 대한 VXC (Virtual Cross Connect) 연결을 생성합니다.
        b. MCR에서 연결 클라우드 프로바이더 (예: AWS Direct Connect)에 대한 VXC 연결을 생성합니다.

![]({{site.baseurl}}/assets/img/blog_interconnectivity2.png)

FastConnect, MCR 및 클라우드 프로바이더와의 연결 (예: AWS Direct Connect, Azure ExpressRoute 또는 Google Cloud Platform)을 설정한 후에는 전용 IP 주소를 통해 리소스에 low-latency 및 high-bandwidth로 액세스할 수 있습니다.


# Choosing Your Connectivity Option

다음의 고급 정보를 사용하여 연결 옵션을 선택할 수 있습니다. 그러나 최상의 연결 옵션은 사용 사례에 따라 다를 수 있습니다. AWS Direct Connect에 대한 정보가 예시로 제공 됩니다.

## Speed
    * FastConnect는 1G 및 10G port speed를 제공합니다.
    * Direct Connect는 50M, 100M, 200M, 300M, 400M, 500M, 1G 및 10G port speed를 제공합니다.

## Cost
    * Oracle FastConnect는 port 시간 당 요금을 부과하며 데이터 전송 비용은 정구되지 않습니다. 자세한 내용은 [Oracle FastConnect Pricing](https://cloud.oracle.com/en_US/fastconnect/pricing) 을 참조하십시오.  
    * Oracle IPSec VPN 서비스는 인바운드 데이터 전송에 대해 요금을 부과하지 않으며, 아웃바운드 데이터 전송을 최대 10TB까지 무료이며, 10TB 제한을 초과한 후에는 약간의 요금이 부과됩니다. 자세한 내용은 [Oracle IPSec VPN Pricing](https://cloud.oracle.com/networking/pricing) 을 참조하십시오.  
    * Amazon 가격에는 port 요금과 데이터 전송 요금이 있습니다. 인바운드 데이터는 미터링 되지 않지만 아웃바운드 데이터는 미터링 및 과금 됩니다. 자세한 내용은 [Amazon Direct Connect Pricing](https://aws.amazon.com/directconnect/pricing/) 을 참조하십시오.  

## Time
데이터 전송 시간은 각 홉에서 이루어지는 속도 선택에 따라 달라집니다. 전용 연결과 IPSec VPN을 비교하여 전용 연결은 전용 미디어를 사용하고 보다 안정적이고 일관적 입니다.

다음 테이블에는 AWS에서 Oracle Cloud Infrastructure로 데이터를 전송하는 데 소요되는 시간대에 따른 가상 비용 시나리오 입니다.

![]({{site.baseurl}}/assets/img/blog_interconnectivity3.png)


# Summary

이 포스트는 일반적으로 사용 가능한 클라우드 간 연결 옵션과 Oracle Cloud Infrastructure를 통해 멀티 클라우드 액세스를 구현하는 방법에 대해 설명합니다. 또한 연결 경로를 정의하고 사용 사례에 맞는 최적의 연결을 선택하는데 사용할 수 있는 연결 옵션을 비교할 수 있는 high-level indicator를 제공합니다. 연결에 대한 자세한 내용과 단계별 지침은 [Migrating Oracle Databases from Amazon Web Services to Oracle Cloud Infrastructure Database](https://cloud.oracle.com/iaas/whitepapers/database_migration_aws_to_oci_database.pdf) 백서를 참조하십시오.  


## 참조
이 포스트의 원문은 [Interconnecting Clouds with Oracle Cloud Infrastructure](https://blogs.oracle.com/cloud-infrastructure/interconnecting-clouds-with-oracle-cloud-infrastructure) 입니다.  