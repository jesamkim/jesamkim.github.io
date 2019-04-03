---
layout: post
title: "OCI에서 openVPN 구축하기"
date: 2019-04-03 12:00:00 +0900
description: "OCI에서 oepnVPN을 구축해 봅니다."
img: "openvpn.jpg"
tags: ["oracle", "oci", "iaas", "cloud", "vpn", "openvpn", "priavte", "network", "oracle cloud", "오라클 클라우드"] 
---

VPN(Virtual Private Network)를 사용하면 Public Internet을 통해 엔터프라이즈 네트웍에 연결할 수 있습니다.<br>
Oracle Cloud Infrastructure에서는 온프레미스 데이터센터와 OCI 데이터센터를 연결하는 방법으로 IPSec VPN과 FastConnect 라는 옵션이 있습니다.<br>
IPSec VPN은 고객의 온프레미스 데이터센터의 CPE와 OCI의 DRG를 연결하여 터널링을 구성하는 방식 입니다.<br><br>

본 포스트에서는 좀더 간단한 환경인 CPE를 사용하지 않고 외부에 있는 사용자(Client)가 VPN에 접속하여 OCI의 IGW를 통해 Network (VCN)에 연결하는 방법에 대해 알아보도록 하겠습니다. 기본적은 VM 및 VCN 생성에 대한 자세한 과정은 생략합니다.<br>

OpenVPN 구축은 다음과 같은 순서로 진행 됩니다.<br><br>
1. OpenVPN을 위한 OCI 구성<br>
2. OpenVPN 서버 설치 및 구성<br>
3. OpenVPN 클라이언트 설치 (PC, MAC, iPhone 등)<br>


## 구성 다이어그램

아래는 본 포스트에서 구축하고자 하는 아키텍처를 보여줍니다.<br>

![]({{site.baseurl}}/assets/img/openvpn01.png)

그림에서 보듯이 VCN(10.0.0.0/16)에는 2개의 서브넷이 있습니다.

* **Public Subnet (10.0.1.0/24)** - IGW를 통해 인터넷에 액세스 할 수 있는 Public Subnet 입니다.
* **Private Subnet (10.0.2.0/24)** - 인터넷에 액세스 할 수 없는 Private Subnet 입니다.


# 1. OpenVPN을 위한 OCI 구성

이번 단계에서는 OpenVPN용 OCI VCN을 생성하는 방법에 대해 간략히 설명합니다.


## VCN 만들기

OpenVPN 서버와 Linux 호스트들을 수용할 수 있도록 AD에 2개의 서브넷이 있는 VCN을 만듭니다. VCN을 만드는 방법 및 Best Practive는 [VCN 개요 및 배포 가이드](https://cloud.oracle.com/opc/iaas/whitepapers/OCI_WhitePaper_VCN_v1.0_LL.pdf)를 참조하세요.

![]({{site.baseurl}}/assets/img/openvpn02.png)

* VCN Name : DataCenter
* VCN CIDR : 10.0.0.0/16


## Public Subnet 구성

Public Subnet의 Route Table인 **Default route table for datacenter**는 모든 트래픽(0.0.0.0/0)에 대해 IGW로의 라우팅 룰이 있습니다.
Public Subnet의 Security List인 **Default Security List**에는 모든 트래픽(0.0.0.0/0)에 대한 egress룰이 있으며, ingress 룰은 다음과 같습니다.

* TCP Port 22 for SSH
* TCP Port 443 for OpenVPN TCP connection
* TCP Port 943 for OpenVPN Web-UI
* UDP Port 1194 for OpenVPN UDP Port

서브넷을 만드는 방법에 대한 상세는 [VCN 및 서브넷](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/managingVCNs.htm)을 참조하세요.

![]({{site.baseurl}}/assets/img/openvpn03.png)


## OpenVPN 서버 VM 인스턴스 시작

OpenVPN 서버 역활을 하는 인스턴스를 생성합니다. 여기서는 **CentOS 7**과 **VMStandard2.1 shape**을 사용하며 VCN의 Public Subnet 상에 런칭합니다. 이 인스턴스에 OpenVPN을 설치하겠습니다.

자세한 내용은 [인스턴스 시작](https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/launchinginstance.htm?tocpath=Getting%20Started%7CTutorial%20-%20Launching%20Your%20First%20Linux%20Instance%7C_____4)을 참조하세요.

![]({{site.baseurl}}/assets/img/openvpn04.png)

인스턴스가 Running 상태가 되면, 이 인스턴스의 VNIC에 대한 forwarding 설정을 합니다.
OCI UI에서 **Compute > Instances > Instance Details > Attached VNICs** 화면에서 아래에 있는 Attached VNICs에 나타나 있는 Primary VNIC의 오른쪽에 있는 "..." 버튼을 클릭하여 **Edit VNIC**을 선택하고 **Skip Source/Destination Check**에 체크표시를 한 뒤, Update VNIC 버튼을 눌러 나옵니다.


## Private Subnet 구성

Private Subnet의 라우팅 테이블인 Private RT에는 라우팅 룰이 있으며, OpenVPN 서버 인스턴스의 Private IP (10.0.1.9)를 통해 모든 트래픽 0.0.0.0/0에 대한 경로로 구성 됩니다.

![]({{site.baseurl}}/assets/img/openvpn05.png)

Security Lists에는 모든 대상에 대한 트래픽을 허용하는 egress rule이 있습니다. ingress rule은 특정 주소 범위 (예: 사내 네트워크 또는 VCN의 다른 Private Subnet) 만 허용 합니다.

![]({{site.baseurl}}/assets/img/openvpn06.png)


# 2. OpenVPN 서버 설치 및 구성

위에서 구동된 OpenVPN 서버 인스턴스에 SSH를 통해 접속하고 OpenVPN 패키지를 설치합니다.
[OpenVPN](https://blogs.oracle.com/cloud-infrastructure/You%20can%20download%20the%20software%20package%20for%20your%20OS%20platform%20from%20OPENVPN%20website%20%20%20%20https:/openvpn.net/index.php/access-server/download-openvpn-as-sw.html) 웹 사이트에서 OS 플랫폼 용 소프트웨어 패키지를 다운로드 할 수 있습니다.

OpenVPN 서버 패키지를 다운로드 합니다.<br>
~~~
    $ wget http://swupdate.openvpn.org/as/openvpn-as-2.5.2-CentOS7.x86_64.rpm
~~~

![]({{site.baseurl}}/assets/img/openvpn07.png)

RPM 명령어로 패키지를 설치 합니다.
~~~
    $ sudo rpm -ivh openvpn-as-2.5.2-CentOS7.x86_64.rpm
~~~

![]({{site.baseurl}}/assets/img/openvpn08.png)

그리고 passwd openvpn 명령어로 openvpn 유저에 대한 패스워드를 설정합니다.
~~~
    $ sudo passwd openvpn
~~~

OpenVPN 인스턴스의 Public IP로 Web UI에 접속합니다. (https://PUBLIC-IP:943/admin)<br>
Admin 계정은 openvpn 으로 접속 합니다.

![]({{site.baseurl}}/assets/img/openvpn09.png)

로그인 후 Configuration > **Network Settings** 을 클릭하고 **Hostname or IP Address:**에는 OpenVPN 인스턴스의 Public IP를 입력합니다.

![]({{site.baseurl}}/assets/img/openvpn10.png)

그런 다음 Configuration > **VPN Settings** 를 클릭하고 **Routing** 섹션에 VCN의 Public Subnet 및 Private Subnet 범위를 추가하세요.

![]({{site.baseurl}}/assets/img/openvpn11.png)

**Routing** 섹션에서 **Should client Internet traffic be routed through the VPN?**을 **Yes**로 설정 합니다.

**DNS Settings** 섹션에서 **Have clients use these DNS servers**에서 Primaru DNS Server와 Secondary DNS server를 구글 DNS로 설정합니다.
* Primary DNS Server : 8.8.8.8
* Secondary DNS Server : 8.8.4.4  (아래 그림이 잘못되었습니다)

![]({{site.baseurl}}/assets/img/openvpn12.png)


## 클라이언트 간 통신

Configuration > **Advanced VPN** 을 클릭하고, **Should clients be able to communicate with each other on the VPN IP Network?**를 **Yes**로 설정 합니다.

![]({{site.baseurl}}/assets/img/openvpn13.png)

변경 사항을 **Save Settings**를 눌러서 저장합니다. 그리고 **Update Running Server**를 클릭하여 OpenVPN 서버에 설정된 내용을 반영합니다.


# 3. OpenVPN 클라이언트 설치

OpenVPN 액세스 서버의 Web(https://PUBLIC-IP:943)에 접속하여 클라이언트 플랫폼에 맞는 OpenVPN 클라이언트를 다운로드 합니다.
(모바일 디바이스 ex. iPhone 및 iPad의 경우, AppStore에서 OpenVPN을 검색하여 설치 합니다.)

![]({{site.baseurl}}/assets/img/openvpn14.png)

설치가 완료되면 OS 작업 표시줄에 OpenVPN 아이콘이 나타납니다. 이 아이콘을 마우스 오른쪽 버튼으로 클릭하면 OpenVPN 연결을 시작할 수 있는 상황에 맞는 메뉴가 나타납니다.

![]({{site.baseurl}}/assets/img/openvpn15.png)

**Connect**를 클릭하면 OpenVPN 사용자명 및 암호를 묻는 창이 나타납니다. 위에서 설정한 openvpn 계정 및 암호로 **Connect** 하여 VPN 터널링을 설정합니다.

![]({{site.baseurl}}/assets/img/openvpn16.png)


## VPN 테스트

VPN 연결이 활성화 된 후, OpenVPN 서버의 Private IP 또는 Private Subnet에 있는 인스턴스의 Private IP로 SSH 접속을 해 봅니다.

![]({{site.baseurl}}/assets/img/openvpn17.png)





## 참조
- 본 내용에 대한 원문은 [Creating a Secure SSL VPN Connection Between Oracle Cloud Infrastructure and a Remote User](https://blogs.oracle.com/cloud-infrastructure/creating-a-secure-ssl-vpn-connection-between-oracle-cloud-infrastructure-and-a-remote-user) 입니다.



** 해당 포스트는 개인적으로 시간을 할애하여 작성하였습니다. 내용에 오류가 있을 수 있으며, 글의 내용은 개인적인 의견 입니다.