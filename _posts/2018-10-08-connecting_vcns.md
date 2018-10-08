---
layout: post
title: "[번역] Multiple VNIC로 서로 다른 VCN 연결하기: Part 1"
date: 2018-10-08 10:00:00 +0900
description: "여러 VNIC를 이용하여 서로 다른 VCN으로 통신하기" # Add post description (optional)
img: connect_vcns_using_vnics.png   # Add image post (optional)
tags: [oracle, oci, iaas, cloud, interconnect, vcn, vnic, terraform, network] # add tag
---

Oracle Cloud Infrastructure에서는 둘 이상의 VCN(Virtual Cloud Network)을 결하는 데 [LPG(Local Peering Gateway)](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/localVCNpeering.htm)가 사용됩니다. LPG를 사용하면 프로바이더 VCN이 고객 VCN과 연결되어 공유 리소스에 대한 Private Access를 허용할 수 있는 서비스 프로바이더 모델을 사용할 수 있습니다. 그러나 [VCN 당 10개의 LPG만 연결](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/localVCNpeering.htm)할 수 있습니다. Oracle Cloud Infrastructure를 사용하는 IT 관리 회사의 예를 살펴보겠습니다. 이들은 하나의 VCN을 중앙 IT 팀이 제어하는 허브 VCN으로 프로비저닝 합니다. 각 클라이언트에 대해 Spoke VCN을 프로비저닝 합니다. 허브 VCN이 10개 이상의 클라이언트 VCN에 연결하여 관리해야 하는 경우 브릿지 인스턴스를 사용하면 제한 문제가 해결될 수 있습니다.

이 포스트에서는 브릿지 인스턴스에서 보조 VNIC를 사용하여 여러 VCN을 연겨하는 솔루션에 대해 설명합니다. 동일한 개념을 확장하여 동일한 리전 및 동일한 테넌시에 2개 이상의 VCN을 연결할 수 있습니다.


# Use Case

이 포스트에서는 비 중복 서브넷인 10.0.0.0/16 및 10.1.0.0/16 을 사용하여 두 개의 VCN (VCN-1 및 VCN-2)을 연결하는 예를 사용 합니다. 두 개의 VCN에 서브넷이 겹치면 동일한 VCN 내의 겹치는 끝점이 라우팅에 우선하기 때문에 트래픽은 동일한 VCN으로 제한합니다.

# VCN-1 어떻게 VCN-2에 연결될까?

정답은 **브릿지 인스턴스의 보조 VNIC** 입니다. 모든 인스턴스에는 인스턴스가 시작되는 VCN의 서브넷에 연결된 기본 VNIC가 있습니다. 보조 VNIC를 이 인스턴스에 연결하고 각 VNIC를 다른 VCN 서브넷에 연결할 수 있습니다. 이는 Oracle Cloud Infrastructure가 서비스 공급자 모델의 구성 요소로 제공하는 고유한 기능 입니다.

브릿지 인스턴스라고하는 공용 인스턴스가 VCN-1의 관리 서브넷에서 시작되면 VCN-1의 관리 서브넷에서 브리지 인스턴스에 VNIC가 연결됩니다. 이것이 Default (Primary) VNIC 입니다.

브릿지 인스턴스를 VCN-2에 연결하려는 attached VNIC (secondary VNIC라고 함)을 만드세요. 다음 그림과 같이 브릿지 인스턴스의 새 VNIC가 VCN-2의 관리 서브넷에서 오는지 확인합니다.

![]({{site.baseurl}}/assets/img/connect_vcns_using_vnics_1.png)

이제 브릿지 인스턴스는 기본 VNIC를 통해 VCN-1의 관리 서브넷 (MgmtSubnet1)에 연결되고 보조 VNIC을 통해 VCN-2의 관리 서브넷 (MgmtSebnet2)에 연결됩니다.

이 구성이 완료된 후에는 기본 route table과 security list를 설정하여 브릿지 인스턴스에서 IP forwarding을 사용할 수 있도록 설정해야 합니다. 이 단계에서는 트래픽을 VCN-1에서 VCN-2로, VCN-2에서 VCN-1로 브릿지 인스턴스를 통해 전달할 수 있습니다.

## Network

각각의 VCN은 Management 서브넷과 Private 서브넷을 가지고 있습니다.

|                             | VCN-1       | VCN-2      |
|:------------------------:|:-------------:|:------------:|
| Network Subnet       | 10.0.0.0/16 | 10.1.0.0/16 |
| Management Subnet | 10.0.0.0/24 | 10.1.0.0/24 |
| Private Subnet         | 10.0.1.0/24 | 10.1.1.0/24 |

## Instances

VCN-1관 VCN-2 모두 해당 Private 서브넷에서 각각 private 인스턴스가 있습니다. VCN-1에는 브릿지 인스턴스라고 하는 공용 인스턴스가 추가로 있으며, 이는 두 VCN을 연결하는 데 도움이 됩니다. 브릿지 인스턴스는 VCN 중 하나에서 생성할 수 있습니다.

|                             | VCN-1              | VCN-2             |
|:------------------------:|:-------------------:|:------------------:|
| Management Subnet | Bridge Instances |                       |
| Private Subnet         | PrivateInstance1  | PrivateInstance2 |


# 어떻게 구현할까요?
이 설정을 배포하기 위한 두 가지 옵션이 있습니다.

## Terraform을 이용한 Deploy

모든 작업을 수행하는 [Terraform 템플릿](https://github.com/oracle/terraform-examples/tree/master/examples/oci/connect_vcns_using_multiple_vnics)을 만들었습니다. Terraform은 위의 설정을 배포하고 필요한 Linux 명령을 자동으로 실행합니다. Terraform 실행이 끝나면 인스턴스에 로그인 정보가 표시됩니다.

사용자는 Terraform을 배포하고 인스턴스에 로그인 한 다음, 한 VCN에서 다른 VCN으로 ping 트래픽을 시작하기만 하면 됩니다.


## Oracle Cloud Infrastructure Console 및 Linux 명령을 사용하여 수동으로 배포

이 절에서는 [Oracle Cloud Infrastructure Console](http://console.us-phoenix-1.oraclecloud.com/) 및 Linux 명령을 사용하여 설정을 배포하는 단계를 설명합니다.


### Console Steps

1. **Create Virtual Cloud Network Only** 옵션과 중복되지 않는 CIDR 블록 (서브넷 용)을 사용하여 두 개의 VCN을 만듭니다. 자세한 내용은 [To create a cloud network in the Networking documentation](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/managingVCNs.htm#console)을 참조하세요.
![]({{site.baseurl}}/assets/img/connect_vcns_using_vnics_2.png)

2. 각 VCN의 각 서브넷에는 연결된 route table과 security list가 필요합니다. 이를 위해 4 개의 route table과 4 개의 security list를 작성하세요. 자세한 내용은 [To create a route table](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/managingroutetables.htm#console) 및 [To create a new security list](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/securitylists.htm#console) 를 참조하세요.

3. 각 VCN에 대해 하나의 public 서브넷과 private 서브넷을 만들어 각각에 대한 security list와 route table을 지정합니다. VCN 서브넷 페이지는 다음과 같습니다. 자세한 내용은 [To create a subnet](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/managingVCNs.htm#console) 을 참조하세요.
![]({{site.baseurl}}/assets/img/connect_vcns_using_vnics_3.png)

4. PrivateInstance1을 만들고 VCN-1의 Private 서브넷 (PrivateSubnet1)에 연결합니다. 마찬가지로 PrivateInstance2를 만들어 VCN-2의 Private 서브넷 (PrivateSubnet2)에 연결합니다.
![]({{site.baseurl}}/assets/img/connect_vcns_using_vnics_4.png)

5. BridgeInstance라는 인스턴스를 하나 더 만들고 VCN-1 및 MgmtSubnet1에 연결합니다.
![]({{site.baseurl}}/assets/img/connect_vcns_using_vnics_5.png)

6. BridgeInstance를 열고, **attached VNICs**를 클릭한 다음, **Create VNIC** 를 클릭합니다.

7. 이 secondary VNIC를 VCN-2 및 MgmtSubnet2에 연결합니다.
![]({{site.baseurl}}/assets/img/connect_vcns_using_vnics_6.png)

8. BridgeInstance의 기본 VNIC 및 보조 VNIC의 IP 주소를 기록합니다.
![]({{site.baseurl}}/assets/img/connect_vcns_using_vnics_7.png)

9. VCN-1을 열고, PrivateRouteTable-1을 연 다음, BridgeInstance의 기본 VNIC IP 주소에 route rule을 추가하세요.
![]({{site.baseurl}}/assets/img/connect_vcns_using_vnics_7.png)

10. VCN-2를 열고 PrivateRouteTable-2를 연 다음 BridgeInstance의 보조 VNIC IP ㅜ소에 route rule을 추가하세요.
![]({{site.baseurl}}/assets/img/connect_vcns_using_vnics_8.png)

11. 다음 예제와 같이 VCN-1을 열고 MgmtSecurityList를 열고 ingress rule을 지정하세요. VCN-2 (10.1.0.0/16)와의 상호 통신에 필요한 additional rule이 강조 표시됩니다. VCN-2에 유사한 rule을 복제하세요.
![]({{site.baseurl}}/assets/img/connect_vcns_using_vnics_9.png)

12. VCN-1에서 다음 예제와 같이 PrivateSecurityList를 열고 ingress 및 egress rule을 지정하세요. VCN-2 에 유사한 rule을 복제하세요.
![]({{site.baseurl}}/assets/img/connect_vcns_using_vnics_10.png)

13. 콘솔 관련 구성을 모두 완료한 후 [Oracle Linux 인스턴스에 로그인](https://docs.oracle.com/en/cloud/iaas/compute-iaas-cloud/stcsg/accessing-oracle-linux-instance-using-ssh.html#GUID-D947E2CC-0D4C-43F4-B2A9-A517037D6C11)하여 다음 단계를 수행하세요.


### Configuring the Bridge Instance

1. 브릿지 인스턴스에 로그인하고 [보조 VNIC를 시작](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/managingVNICs.htm#Linux:)합니다.<br> **NOTE** : 계속하기 전에 보조 VNIC을 활성화 하는 단계를 수행해야 합니다.

2. 보조 VNIC (ens4)에 올바른 IP 주소가 나타나는지 확인하세요.

3. 브릿지 인스턴스에서 다음 명령을 실행하여 IP forwarding을 활성화 합니다.

    sysctl -w net.ipv4.ip_forward = 1

4. Port forwarding을 사용하려면 다음 방화벽 명령을 실행하세요.

    firewall-cmd --direct --add-rule ipv4 filter FORWARD 0 -i ens3 -j ACCEPT
    
    firewall-cmd --direct --add-rule ipv4 filter FORWARD 0 -i ens4 -j ACCEPT
    
    /bin/systemctl restart firewalld

5. 다음 명령을 사용하여 [인스턴스 메타 데이터를 확인](https://docs.cloud.oracle.com/iaas/Content/Compute/Tasks/gettingmetadata.htm)하여 VCN-2의 MgmtSubnet2에 대한 virtualRouteIp에 유의하세요.

    [opc@bridgeinstance ~]$ curl http://169.254.169.254/opc/v1/vnics/
    [ {
      "vnicId" : "ocid1.vnic.oc1.phx.abyhqljssar7g5nfkw5wfa52df6ysz5zh7qgjnz4u5mute7bsjagjixdngya",
      "privateIp" : "10.0.0.2",
      "vlanTag" : 1681,
      "macAddr" : "00:00:17:01:E0:3B",
      "virtualRouterIp" : "10.0.0.1",
      "subnetCidrBlock" : "10.0.0.0/24"
    }, {
      "vnicId" : "ocid1.vnic.oc1.phx.abyhqljsj6uqkigzjkfyauhalgeqgrytxfmhqipgrmddxgan4343tuurwk3q",
      "privateIp" : "10.1.0.2",
      "vlanTag" : 1683,
      "macAddr" : "00:00:17:01:2B:46",
      "virtualRouterIp" : "10.1.0.1",    <<<<<<<<
      "subnetCidrBlock" : "10.1.0.0/24"
    } ][opc@bridgeinstance ~]$

6. VCN-2의 가상 라우터 IP로 라우팅되는 트래픽에 대한 VCN-2 MgmtSubnet2의 IP route rule을 추가합니다.

    ip route add <VCN-2-Network> dev <secondary-vnic> via <MgmtSubnet2.Virtual_router_ip>
    
    Ex: ip route add 10.1.0.0/16 dev ens4 via 10.1.0.1

### Verification

1. BridgeInstace를 통해 PrivateInstance1에 로그인하세요.

2. PrivateInstance1에서 PrivateInstance2로 ping 합니다. 브릿지 인스턴스에서 보조 VNIC를 사용하여 교차 VCN 통신을 사용하도록 설정하였습니다.


# Extension

이제 최대 52개의 VCN을 서로 연결하여 허브 및 스포크 모델을 가지는 솔루션을 확장할 수 있습니다. 브릿지 인스턴스는 [최대 52개의 VNIC](https://docs.cloud.oracle.com/iaas/Content/Compute/References/computeshapes.htm)를 가질 수 있으므로 브릿지 인스턴스를 사용하여 최대 52개의 VCN을 연결할 수 있습니다.


# Next

이 시리즈의 Part2 에서는 브릿지 인스턴스에 HA (High Availability)를 제공하는 방법을 살펴볼 것입니다. 브릿지 인스턴스가 down 되면 백업 브릿지 인스턴스로 take over 됩니다.


## 참고
원문 포스트는 [Connect VCNs by Using Multiple VNICs: Part1](https://blogs.oracle.com/cloud-infrastructure/connecting-vcns-by-using-multiple-vnics%3a-part-1) 입니다.
