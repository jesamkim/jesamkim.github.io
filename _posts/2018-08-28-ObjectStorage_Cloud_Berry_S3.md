---
layout: post
title: "OCI Object Storage를 CloudBerry Explorer for Amazon S3로 접속하기"
date: 2018-08-28 00:00:00 +0300
description: "Cloudberry Explorer에 OCI의 Object Storage를 연동시키는 방법 입니다." # Add post description (optional)
img: oraclelogo1.png # Add image post (optional)
tags: [oracle, oci, iaas, cloud, objectstorage, s3] # add tag
---

> OCI (Oracle Cloud Infrastructure)의 IaaS 서비스 중 Object Storage는 S3 호환 API가 제공되므로, CloudBerry Explorer for Amazon S3 사용자는 손쉽게 OCI Object Storage를 연결하실 수 있습니다.
> [CloudBerry Explorer for Amazon S3](https://www.cloudberrylab.com/download-thanks.aspx?prod=cbes3free)에 대한 설치 등의 과정은 본 글에서는 다루지 않습니다.


## OCI Region

본 테스트는 OCI Region 중 us-ashburn-1에서 진행하였습니다. Oracle Cloud Trial 계정을 사용하시는 분들은 북미 지역을 선택하시면 Ashburn region 사용 가능합니다.


## OCI Object Storage 버킷 생성

Bucket이란 Object를 저장하는 논리 컨테이너를 의미합니다.
OCI의 경우 버킷 이름은 전역적으로 고유할 필요가 없습니다.
단, 동일한 네임스페이스 내에서의 버킷 이름은 고유해야 합니다.

OCI의 Web Console로 로그인 합니다.
왼쪽 메뉴에서 Object Storage > Object Storage 를 선택합니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api01.png)

Object Storage 버킷을 생성할 COMPARTMENT를 지정합니다. 
그리고 “Create Bucket” 버튼을 클릭합니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api02.png)

예로 CloudBerry 라는 이름의 버킷을 작성하겠습니다.
Storage Tier는 STANDARD 또는 ARCHIEVE 중 하나를 선택합니다.

	- STANDARD : 일반적인 Object Storage의 타입 입니다.
	- ARCHIEVE : 보관용으로 사용하는 데이터를 Archieve 하며, 비용이 절약 됩니다. 
		     단, Object를 다시 읽어 올때 First time to read의 경우 4시간 정도 소요됩니다.
	
![]({{site.baseurl}}/assets/img/cloudberry_s3api03.png)

다음과 같이 CloudBerry 버킷이 생성되었습니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api04.png)


## S3 호환 API 인증 설정

AWS S3 호환 API를 사용하기 위해 사용자의 액세스 키와 비밀 키가 필요합니다.

메뉴에서 Identity > Users 선택

![]({{site.baseurl}}/assets/img/cloudberry_s3api05.png)

실제로 버킷에 액세스하는 사용자를 선택하고 사용자 이름을 클릭 합니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api06.png)

왼쪽 Resources 에서 Amazon S3 Compatibility API Keys 를 선택합니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api07.png)

Amazon S3 Compatibility API Keys 화면에서 “Generate Secret Key” 버튼을 클릭합니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api08.png)

여기서는 CloudBerry로 키이름을 지정하였습니다. 
이름 입력 후, “Generate Secret Key” 버튼을 클릭합니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api09.png)

와 같이 생성된 Secret Key를 메모장에 Copy & Paste 합니다.
그리고 Access Key에 대한 OCID도 메모장에 Copy & Paste 합니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api10.png)


## CloudBerry Explorer 액세스 등록

CloudBerry Explorer를 실행합니다.
여기서는 CloudBerry Explorer for Amazon S3 — Build 5.6.0.4를 사용하였습니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api11.png)

File > Add New Account 를 선택합니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api12.png)

Select Cloud Storage 창에서 “S3 Compatible” 를 더블클릭 합니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api13.png)

아래와 같이 접속 정보를 입력합니다.

	- Display name : 원하는 이름을 입력합니다.
	- Service point : 테넌트이름.compat.objectstorage.리전이름.oraclecloud.com 형식으로 입력합니다.
			  ex) jaykim000.compat.objectstorage.us-ashburn-1.oraclecloud.com
	- Access Key : OCI 콘솔의 Amazon S3 Compatibility API Keys에서 만든 키의 OCID (앞에서 메모장에 복사하였음)
	- Secret Key : 키 생성 단계에서 Generate Key 한 값을 지정 (앞에서 메모장에 복사하였음)
	- Signature version : 여기서 “4”로 지정
	
![]({{site.baseurl}}/assets/img/cloudberry_s3api14.png)

위와 같이 입력 후 “Test Connection” 버튼을 누릅니다. Connection success 가 나오면 성공 입니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api15.png)

이제 CloudBerry Explorer 오른쪽 패널(혹은 왼쪽)에서 위에서 액세스 설정한 OCI Object Storage 버킷을 선택 및 액세스할 수 있습니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api16.png)

![]({{site.baseurl}}/assets/img/cloudberry_s3api17.png)

OCI Object Storage 의 CloudBerry 버킷에 파일 업로드 테스트도 정상적으로 수행 됩니다.

![]({{site.baseurl}}/assets/img/cloudberry_s3api18.png)

참고로 OCI-C (OCI Classic)의 Obejct Storage는 CloudBerry Explorer for OpenStack Storage에서 연결할 수 있습니다.
OCI의 Object Storage는 AWS S3 호환 API로서 CloudBerry Explorer for Amazon S3 에 연결할 수 있습니다.


### 참조자료
[Oracle Public Document](https://docs.cloud.oracle.com/iaas/Content/Object/Tasks/s3compatibleapi.htm?tocpath=Services%7CObject%20Storage%7C_____8)
