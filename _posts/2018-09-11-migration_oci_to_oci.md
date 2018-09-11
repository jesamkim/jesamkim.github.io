---
layout: post
title: "Migration from OCI to OCI"
date: 2018-09-11 00:00:00 +0300
description: "다른 Region으로 OCI Compute Instance를 Migration" # Add post description (optional)
img: migration_from_oci_to_oci00.png # Add image post (optional)
tags: [oracle, oci, iaas, cloud, customimage, objectstorage, migration] # add tag
---

OCI에서 Compute Instance의 Migration 상황을 다음의 2가지 경우라고 가정해 봅시다.
	1. 다른 OCI Tenancy로 이전할 경우
	2. 다른 OCI Region으로 이전할 경우 

위와 같은 상황에서는 다음과 같이 Migration 할 수 있습니다.
(Step1인 Instance의 Boot Volume에 대해서만 Migration 해봅니다)
<br><br>
여기서는 Region A 에서 Region B로 Linux Instance를 Migration 하겠습니다.
Region A에서는 진행하기 전 Object Storage에 **Public Bucket**을 생성해 둡니다. 
<br><br><br>

## Region A

**MENU > Compute > Instances**
* 이전하려는 Compute Instance에서 **Create Custom Image** 클릭

![]({{site.baseurl}}/assets/img/migration_from_oci_to_oci01.png)
<br>
![]({{site.baseurl}}/assets/img/migration_from_oci_to_oci02.png)
<br><br>

**MENU > Compute > Custom Images**
* 생성된 Custom Image에서 **Export Custom Image** 클릭

![]({{site.baseurl}}/assets/img/migration_from_oci_to_oci03.png)
<br>
![]({{site.baseurl}}/assets/img/migration_from_oci_to_oci04.png)
<br><br>

이미지는 QCOW2 format으로 export 됩니다. (exporting 에서 시간이 조금 걸립니다)



**MENU > Object Storage > Object Storage**
* 위에서 Custom Image는 Object Storage의 Bucket에서 확인됩니다.
* 여기서 이미지 파일의 URL Path를 복사해 둡니다.
	   
![]({{site.baseurl}}/assets/img/migration_from_oci_to_oci05.png)
<br><br><br>


## Region B

**MENU > Compute > Custom Images**
* Custom Images에서 **Import Image** 클릭

	- Region A의 Object Storage URL 사용
	- IMAGE TYPE : **QCOW2**
	- LAUNCH MODE : **NATIVE MODE**


![]({{site.baseurl}}/assets/img/migration_from_oci_to_oci06.png)
<br><br>


Import가 완료된 Custom Image에서 **Create Instance** 클릭
	
![]({{site.baseurl}}/assets/img/migration_from_oci_to_oci07.png)
<br>
![]({{site.baseurl}}/assets/img/migration_from_oci_to_oci08.png)
<br><br>

정상적으로 Region B에 새로운 인스턴스를 생성하였습니다.
<br><br>
만약 추가의 Block Volume 이 있다면, 먼저 동일 사이즈의 Block Volume을 생성 후 Attach 합니다.<br>
그리고 sftp 등의 방법을 통해 file system을 복사할 수 있습니다.
