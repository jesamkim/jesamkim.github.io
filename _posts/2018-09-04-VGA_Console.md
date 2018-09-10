---
layout: post
title: "VGA Console을 통한 OCI Compute 인스턴스 연결"
date: 2018-09-04 00:00:00 +0300
description: "OCI Compute 인스턴스를 Public IP를 통하지 않고 접속하는 방법 입니다." # Add post description (optional)
img: vga_console.png # Add image post (optional)
tags: [oracle, oci, iaas, cloud, vgaconsole, vnc] # add tag
---

OCI Compute 에서는 인스턴스에 대한 VGA Console 접속을 지원 합니다. VGA Console 을 통해 인스턴스의 Public IP 로 접속 없이 인스턴스에 접속할 수 있습니다. 
 
> 일반적인 전산실 환경을 예로 들면, 운영팀은 전산실에 있는 리눅스 서버에 IP로 접속을 할 수 있습니다. 하지만 해당 리눅스에 부팅 문제 등으로 IP를 통한 접속이 힘든 경우가 있는데, 이때는 물리서버에 직접 키보드, 모니터를 연결하여 troubleshooting을 하게 됩니다. 이러한 경우 처럼 물리적인 서버에 키보드, 마우스를 직접 연결하여 인스턴스를 볼 수 있게 하는 것이 OCI의 VGA Console 기능 입니다.

VGA Console의 경우 일반적으로 다음의 시나리오 상에서 사용할 수 있습니다.
- OCI Compute instance에서 Public IP 접속이 안되는 상황에서 troubleshooting을 할 경우
- 잘못된 Firewall rule로 인해 외부 및 내부 연결이 막혔을 경우
- RDP를 사용하지 않고 Windows instance에 접속하는 경우
	
Client PC platform은 Windows를 기준으로 설명하겠습니다.


## 사전 준비 사항

	1. OCI Compute 에서 인스턴스 생성
	2. VNC Viewer 설치
	3. VGA Console 인증을 위한 Key pair (Pubic / Private)
	   (인스턴스 생성시 사용한 Key pair를 다시 사용해도 됩니다)


## Console Connections 만들기

Compute >> Instances >> Instance Details >> Resources의 Console Connections 클릭
Create Console Connection 선택 -> Public Key 파일을 등록 합니다.

- 그림1: VGA Console 생성 창
![]({{site.baseurl}}/assets/img/vga_console01.png) 


## Console Connections 접속

Compute >> Instances >> Instance Details >> Resources의 Console Connections 클릭
Connect with VNC를 선택합니다.

- 그림2: VNC를 통한 Console 접속 선택
![]({{site.baseurl}}/assets/img/vga_console02.png)

Platform을 WINDOWS로 선택합니다.  
(만약 Linux 혹은 Mac 을 사용하시는 환경에서는 Platform을 Linux/MAC OS를 선택하시면 됩니다)
그리고 CONNECTION STRING 부분을 복사하여 메모장에 붙여넣습니다. 

- 그림3: Windows 플랫폼에서의 VNC Connection String 창
![]({{site.baseurl}}/assets/img/vga_console03.png)

메모장에 붙여넣은 CONNECTION STRING에서 Private Key 부분을 본인의 환경에 맞게 수정해야 합니다.
이탤릭체의 부분(Private Key)을 Private Key가 있는 절대경로로 수정합니다. 
(본 환경에서는 D:\privateKey.ppk 가 절대 경로 입니다)

	Start-Job { Echo N | plink.exe -i d:\privateKey.ppk -N -ssh -P 443 -l
	ocid1.instanceconsoleconnection.oc1.iad.abuwcljt5kejuuy7piveobg4ng5zimaajwdhj4hggzmwarg5uhrcfk26m6ib -L
	5905:ocid1.instance.oc1.iad.abuwcljtbkooty5o6fsqvs5hvtvewrkoojicpqravrfezo7gkgiertt3upxa:5905 instance-console.us-ashburn-
	1.oraclecloud.com }; sleep 5; plink.exe -i d:\privateKey.ppk -N -L 5900:localhost:5900 -P 5905 localhost -l 
	ocid1.instanceconsoleconnection.oc1.iad.abuwcljt5kejuuy7piveobg4ng5zimaajwdhj4hggzmwarg5uhrcfk26m6ib

Power Shell을 열고 위의 String 붙여넣기 하면, OCI Instance 와 SSH 터널링이 됩니다.
만약 Power Shell 창을 닫게 되면 터널링이 끊어집니다. 따라서 Power Shell 창은 닫지 마시기 바랍니다.

- 그림4: Power Shell에서 Connection String 실행
![]({{site.baseurl}}/assets/img/vga_console04.png)

VNC Viewer를 열고, localhost:5900 으로 접속하면 터널링된 Instance로 VGA Connection이 완료됩니다.

- 그림5: VNC Viewr로 localhost:5900 으로 리눅스 인스턴스 접속
![]({{site.baseurl}}/assets/img/vga_console05.png)


### 참조자료
[OCI Public Document - Instance Console Connections](https://docs.cloud.oracle.com/iaas/Content/Compute/References/serialconsole.htm?tocpath=Services%7CCompute%7C_____14)
[OCI Blogs - Announcing VGA Console support for Oracle Cloud Infrastructure Virtual Machines](https://blogs.oracle.com/cloud-infrastructure/announcing-vga-console-support-for-oracle-cloud-infrastructure-virtual-machines)
	
