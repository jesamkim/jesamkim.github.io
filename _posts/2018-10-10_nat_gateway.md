---
layout: post
title: "[번역] OCI NAT Gateway 소개"
date: 2018-10-10 10:00:00 +0900
description: "Oracle Cloud Infrastructure NAT Gateway를 통해 Public Internet의 리소스에 접속할 수 있습니다." # Add post description (optional)
img: nat_gateway.png   # Add image post (optional)
tags: [oracle, oci, iaas, cloud, nat, gateway, natgateway, private, network] # add tag
---
대부분의 Oracle Cloud Infrastructure 고객은 개인 정보, 보안 또는 운영 문제를 위해 Private Subnet에 연결되는 Compute Instance를 VCN (Virtual Cloud Network)에 보유하고 있습니다. 소프트웨어 업데이트, CRL 검사 등을 위해 이러한 리소스에 대한 Public Internet 액세스 권한을 부여하기 위해, 고객의 유일한 옵션은 Private IP 주소를 사용하여 Public Subnet에 NAT Instance를 생성하고 해당 인스턴스를 통해 트래픽을 라우팅 하는 것이었습니다. 많은 사람들이 이 방법을 성공적으로 사용했지만, 쉽게 확장되지 않고 수많은 관리 및 운영 과제가 생겼습니다.

이러한 과제를 해결하고 Oracle Cloud Infrastructure 고객에게 네트워킹 보안 요구사항을 해결하는 단순하고 직관적인 툴이 NAT Gateway 입니다.
NAT Gateway는 다음과 같은 기능을 제공합니다:

* **Highly Scalable and Fully Managed:**  Private 서브넷의 인스턴스는 Public Internet에 연결 될수 있습니다. 인터넷에서 시작된 연결은 차단됩니다.
* **Secure:** NAT Gateway를 통한 트래픽은 버튼 클릭만으로 비활성화 할 수 있습니다.
* **Dedicated IP Addresses:** 각 NAT Gateway는 보안 화이트리스트에 안정적으로 추가할 수 있는 전용 IP 주소가 할당됩니다.

아래부터는 NAT Gateway를 통해 Private 인스턴스에서 Public Internet에 액세스하는 방법에 대해 설명합니다.

![]({{site.baseurl}}/assets/img/nat_gateway_2.png)

NAT Gateway 이전에는 Private 인스턴스는 (Public) NAT 인스턴스를 통해 Public Internet에 액세스 했습니다. VCN에는 associated route table, security list 및 DHCP 옵션이 있는 하나의 Public 서브넷과 하나의 Private 서브넷이 있습니다.

Bastion 호스트(표시되지 않음)를 통해 SSH를 사용하여 Private 인스턴스에 연결하고 Public Internet의 리소스에 액세스 합니다.

![]({{site.baseurl}}/assets/img/nat_gateway_3.png)


하지만 이제는 VCN에 NAT Gateway를 간단하게 만들 수 있습니다!

![]({{site.baseurl}}/assets/img/nat_gateway_4.png)

VCN에 대한 NAT Gateway 목록에서 새로 생성된 게이트웨이를 볼 수 있습니다.

![]({{site.baseurl}}/assets/img/nat_gateway_5.png)

마지막으로 NAT 인스턴스를 가리키는 route rule을 NAT Gateway를 가리키도록 변경합니다.

![]({{site.baseurl}}/assets/img/nat_gateway_6.png)

몇 단계만 거치면, Private 서브넷의 모든 인스턴스에 인터넷 리소스에 대한 액세스 권한을 부여할 수 있습니다. 다른 OCI 게이트웨이(서비스, 인터넷 등)와 마찬가지로 NAT Gateway는 가용성이 높고 대역폭 요구사항을 충족하기 위해 탄력적으로 확장됩니다. 이제 더 이상 필요하지 않은 Public NAT 인스턴스를 삭제할 수 있습니다.

Private 서브넷의 인스턴스에 인터넷 액세스를 부여하는 경우 선호되는 방법으로 NAT Gateway를 사용하는 것이 좋습니다. [Networking document](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/NATgateway.htm)에서 NAT Gateway에 대한 자세한 내용을 볼 수 있습니다.


## 참조
원문은 [Access Resources on the Public Internet Through an Oracle Cloud Infrastructure NAT Gateway](https://blogs.oracle.com/cloud-infrastructure/access-resources-on-the-public-internet-through-an-oracle-cloud-infrastructure-nat-gateway-v2) 입니다.
