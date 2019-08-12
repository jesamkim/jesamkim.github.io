---
layout: post
title: "OCI AD 내 가용성을 위한 Fault Domain"
date: 2018-09-04 00:00:00 +0800
description: "OCI의 단일 AD(=데이터센터) 내에서 가용성을 높일 수 있는 Fault Domain을 소개합니다." # Add post description (optional)
img: fault_domain.png # Add image post (optional)
tags: ["오라클", "클라우드", "oracle", "oci", "iaas", "cloud", "ad", "faultdomain", "가용성", "도메인"] # add tag
---

AD(Availability Domain) 내에서 OCI(Oracle Cloud Infrastructure) VM(Virtual Machine) 및 BM(Bare Metal) 컴퓨트 인스턴스의 가용성을 관리하고 향상시키는 새로운 방법인 Fault Domain을 소개합니다.

현재 2개 이상의 AD를 통해 VM 및 BM의 인스턴스를 배포하여 응용프로그램의 고가용성을 보장하고 있습니다. AD는 물리적으로 격리되어 있으며 리소스(전원, 쿨링, 네트워크 등)를 공유하지 않으므로 한 지역 내의 다수의 AD에 장애가 발생할 가능성은 매우 낮습니다. 하나의 AD에 문제가 다른 AD에서 실행중인 리소스에 영향을 주지 않기 때문에 여러 AD를 사용하여 높은 가용성을 보장할 수 있습니다.

만약 단일 AD 내에서 응용프로그램의 가용성을 보다 세밀하게 제어하려는 경우, Fault Domain을 사용하여 가용성을 높일 수 있습니다. Fault Domain을 사용하면 하나의 AD 내의 동일한 물리적 하드웨어에 Compute 인스턴스를 배포하지 않습니다. Fault Domain은 예상치 못한 하드웨어 오류 또는 Compute 하드웨어의 유지 보수로 인한 정전 사태에 대비하여 응용프로그램을 보호할 수 있습니다.

OCI는 일반적으로 Region 마다 3개의 AD로 설계되며 각 AD에는 3개의 Fault Domain이 있습니다. Compute 하드웨어에서 유지관리를 수행할 떄, OCI는 단 하나의 Fault Domain에 대해 영향을 주며 나머지 Fault Domain의 인스턴스에 대해 가용성을 보장합니다.

![]({{site.baseurl}}/assets/img/fault_domain01.png)

Fault Domain 설정은 매우 간단합니다. API, CLI 또는 웹 콘솔을 사용하여 새로운 Compute Instance를 만들 때 Fault Domain을 지정할 수 있습니다. Fault Domain을 지정하지 않으면 인스턴스는 해당 AD내의 3개의 Fault Domain 중 하나에 자동으로 분류됩니다. 인스턴스가 생성 된 이후 Fault Domain을 수정하려면 인스턴스를 Terminate 한 후 재생성 해야 합니다. 기존에 이미 사용중인 VM 및 BM의 인스턴스들은 AD내 3개의 Fault Domain에 자동으로 배포됩니다.

![]({{site.baseurl}}/assets/img/fault_domain02.png)

인스턴스 세부사항 페이지는 인스턴스에 대한 다른 메타데이터와 함께 Fault Domain 정보를 표시합니다.

![]({{site.baseurl}}/assets/img/fault_domain03.png)

Fault Domain은 무과금 서비스이며, 자세한 내용은 [OCI Public Document](https://docs.cloud.oracle.com/iaas/Content/General/Concepts/regions.htm#fault), [Compute FAQ](https://cloud.oracle.com/compute/faq)에서 확인하실 수 있습니다.
