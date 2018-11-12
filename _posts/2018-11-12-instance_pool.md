---
layout: post
title: "Instance Configuration 및 Instance Pool을 이용한 Compute Instance Management"
date: 2018-11-12 14:00:00 +0900
description: "Enhanced Compute Instance Management on Oracle Cloud Infrastructure"
img: "instance_pool1.jpg"
tags: ["oracle", "oci", "iaas", "cloud", "instance", "pool", "management", "compute", "oracle cloud", "오라클 클라우드"] 
---

Oracle Cloud Infrastructure는 인스턴스 구성 및 인스턴스 풀을 도입하여 컴퓨팅 인스턴스 관리를 강화하였습니다.
클라우드는 Standard OS images, Standard shape configurations 등 유용한 standard를 많이 제공하지만 volume 및 VNIC과 같은 리소스를 프로비저닝하고 첨부하는데 여전히 추가 오버헤드가 있습니다. 그리고 지금까지는 대규모 프로비저닝 및 인스턴스 설정 관리가 어려웠습니다. Instance Configurations feature는 단일 API call로 인스터스 및 필요한 모든 자원의 프로비저닝을 단순화 합니다. 이러한 단순화는 Instance pool로 확장되며, Instance Configurations는 동일한 다수의 컴퓨팅 인스턴스의 논리적 그룹을 생성하고 규모에 맞게 자동으로 시작됩니다.


# What are Instance Configurations?

Instance Configurations는 단일 구성 엔티티로 인스턴스에 연결된 블록 볼륨과 같이 OS 이미지, shape 및 리소스를 포함하여 Oracle Cloud Infrastructure에서 컴퓨팅 인스턴스를 생성하는데 필요한 필수 및 옵션 파라미터 셋을 정의하는 템플릿 입니다. 기존 실행중인 인스턴스에서 Instance Configurations를 만들거나 CLI를 통해 custom Instance Configuration을 구성할 수 있습니다.
Boot 또는 데이터 저장 영역 볼륨이 아직 없는 경우, 인스턴스를 시작할 때 이러한 resource가 자동으로 작성됩니다. 하나의 단일 액션만으로 인스턴스를 시작하고, 스토리지 볼륨을 생성하고, VNIC를 연결한 다음 원하는 수의 AD(Availability Domain) 에 설정된 인스턴스 수를 균등하게 스트라이핑 할수 있습니다. 이는 일반적으로 인스턴스를 시작하기 위해 플랫폼에서 각 개별 리소스를 수동으로 프로비저닝 해야하는 작업 입니다.


## Create an Instance Configuration

**Create Instance Configuration** 버튼으로 기존 실행중인 인스턴스에서 Instance Configuration을 만들수 있습니다. Compartment를 선택하고 configuration에 대한 이름을 지정합니다. 인스턴스의 모든 메타 데이터는 캡처 됩니다.

왼쪽 메뉴에서 **[Compute](https://console.us-ashburn-1.oraclecloud.com/a/compute/instances)**에서 **[Instances](https://console.us-ashburn-1.oraclecloud.com/a/compute/instances)**를 선택한 뒤, **Instance Details**를 클릭합니다.

![]({{site.baseurl}}/assets/img/instance_pool2.jpg)

저장된 configuration를 보려면,  **[Compute](https://console.us-ashburn-1.oraclecloud.com/a/compute/instances)**에서 **[Instance Configuration](https://console.us-ashburn-1.oraclecloud.com/compute/instance-configs)**을 선택합니다.

![]({{site.baseurl}}/assets/img/instance_pool3.jpg)


## How Instance Pools Work

Oracle Cloud Infrastructure는 Instance Pool이라는 논리적 그룹에서 동일한 VM 인스턴스를 관리할 수 있습니다. Pool은 VM 인스턴스의 수평 확장 가능한 pool을 자동으로 프로비저닝 합니다. Instance Pool은 인스턴스 생성 방법에 대한 모든 설정을 포함하는 인스턴스 구성 템플릿을 사용합니다. Instance Pool은 Instance consifuration templete을 기반으로 동일한 인스턴스의 launching을 관리합니다. Pool은 구성된 인스턴스 수를 유지 관리하며 요구에 맞게 확장할 수 있습니다. Instance Pool은 모든 인스턴스가 실행중인 상태인지 확인하기 위해 자체 상태를 지속적으로 모니터링 합니다. 인스턴스가 실패할 경우 Pool이 자동으로 self-heal 되어 Pool을 healthy state로 되돌릴 수 있습니다.


## Easily Create and Launch a New Instance Pool

Instance Pool을 생성하는 것은 30초도 걸리지 않습니다.

1. **[Compute](https://console.us-ashburn-1.oraclecloud.com/a/compute/instances)**에서 **[Instance Pool](https://console.us-ashburn-1.oraclecloud.com/compute/instance-pools)**을 선택하고 **Create Instance Pool** 버튼을 클릭합니다. 그리고 Pool에 사용할 인스터스 수를 입력합니다.

2. 이전에 작성한 Instance Configuration을 선택하세요.

3. AD(Availability Domain)을 선택하세요. (인스턴스는 선택한 AD 전체에 균등하게 분산됩니다.)

4. Primary VNIC과 Subnet을 선택하세요.

![]({{site.baseurl}}/assets/img/instance_pool4.jpg)

Instance Pool을 프로비저닝하면 구성된 인스턴스가 시작됩니다.

![]({{site.baseurl}}/assets/img/instance_pool5.jpg)

Instance Pool이 실행되면 Pool에서 전원 작업을 수행할 수 있습니다. **Edit** 버튼을 사용하여 Pool 크기를 number instance (0-50)로 갱신할 수 있습니다. Pool을 중지하면 풀의 모든 인스턴스가 중지됩니다. **Reboot**는 모든 인스턴스를 다시 시작하고, **Terminate**는 모든 인스턴스와 Pool 자체를 삭제합니다.

![]({{site.baseurl}}/assets/img/instance_pool6.jpg)


## Common Use Cases

### Instance Configurations:

* 인스턴스 복제 및 configuration file 저장

* 표준화된 기본 인스턴스 템플릿 생성

* 단일 configuration file을 사용하여 CLI에서 인스턴스를 쉽게 배포

* 많은 인스턴스와 리소스의 프로비저닝을 자동화


### Instance Pools:

* 일관된 구성으로 인스턴스 워크로드 그룹을 중앙 집중식으로 관리

* Pool의 인스턴스 크기를 늘려 on-demand에 따라 인스턴스 확장

* 단일 instance configuration 변경으로 많은 수의 인스턴스 업데이트

* 고가용성을 유지하고 region 내 AD간 인스턴스 분산

* 더 큰 shape으로 instance configuration을 업데이트하여 pool내에서 VM 크기를 쉽게 확장

* Pool 내에서 자동 self-healing을 활성화하여 Pool 크기와 가용성 유지

* 수백가지 맞춤형 VM 이미지를 대규모로 지원하여 고객 요구사항에 대응

Instance Configuration 및 Instance Pool 모두의 시너지 효과를 통해 Oracle Cloud Infrastructure에서 수백개의 VM을 손쉽게 관리하고 구현하는데 필요한 복잡성을 없앨수 있습니다.
Instance Configuration 및 Instance Pool은 추가비용이 없습니다. 실행된 VM 인스턴스에서 사용된 리소스에 대해서만 과금됩니다.


## 참고

원문은 [Enhanced Compute Instance Management on Oracle Cloud Infrastructure](https://blogs.oracle.com/cloud-infrastructure/enhanced-compute-instance-management-on-oracle-cloud-infrastructure) 입니다.
