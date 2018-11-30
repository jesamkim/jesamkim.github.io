---
layout: post
title: "Cross-Region Block Volume Backup 기능 소개"
date: 2018-11-30 14:00:00 +0900
description: "Cross-Region Block Volume Backups for Business Continuity, Migration, and Expansion"
img: "cross-region-bv-backup01.jpeg"
tags: ["oracle", "oci", "iaas", "cloud", "block", "volume", "backup", "cross", "region", "oracle cloud", "오라클 클라우드"] 
---

Oracle Cloud Infrastructure 에서 OCI Region 간 볼륨 백업을 할 수 있는 기능이 추가되었습니다.

이 새로운 기능은 클라우드에서 포괄적인 애플리케이션 및 데이터 보호 솔루션을 고객에게 제공하기 위한 지속적인 투자의 일환 입니다. Cross-region 백업은 다음과 같은 장점을 가집니다.

* **재해 복구 및 비즈니스 연속성:** 정기적으로 블록 볼륨 백업을 다른 region으로 복사함으로써 region 전체의 재난이 발생하더라도 다른 region에서 애플리케이션 및 데이터를 재구축 할 수 있습니다.

* **마이그레이션 및 확장:** 애플리케이션을 다른 region으로 쉽게 마이그레이션하고 확장할 수 있습니다.

이 기능은 Oracle Cloud Infrastructure 고객에게 추가 비용 없이 제공되며, 원격 복사에 사용되는 블록 및 오브젝트 스토리지의 크기 이상의 비용이 발생하지 않습니다. Oracle Cloud Infrastructure Console, API, CLI, SDK 및 Terraform에서 사용할 수 있습니다.

블록 볼륨 백업을 다른 region으로 복사하는 것은 Oracle Cloud Infrastructure Console에서 간단하게 수행할 수 있습니다.
다음 단계에서는 cross-region 간 백업을 복사하는 방법과 다른 region의 백업에서 볼륨을 복원하는 방법을 설명합니다.

1. 콘솔의 Block Storage 섹션에서 컴파트먼트를 선택하고 block volume backups로 들어가세요.

2. 다른 region에 복사할 백업에 대한 작업 메뉴(...)에서 **Copy to Another Region**을 선택하세요.

![]({{site.baseurl}}/assets/img/cross-region-bv-backup02.png)

3. 백업 및 대상 region에서 이름을 지정하고, **Copy Block Volume Backup**을 누르세요.

![]({{site.baseurl}}/assets/img/cross-region-bv-backup03.png)

4. 백업 복사 설정을 확인하세요.

![]({{site.baseurl}}/assets/img/cross-region-bv-backup04.png)

5. 콘솔에서 대상 region으로 이동하고 해당 region에서 백업을 사용할 수 있는지 확인 합니다.

![]({{site.baseurl}}/assets/img/cross-region-bv-backup05.png)

이제 백업에서 새 볼륨을 생성하여 대상 region의 백업에서 복원할 수 있습니다.

6. 대상 region의 백업에 대한 작업 메뉴(...)에서 **Create Block Volume**을 선택합니다.

![]({{site.baseurl}}/assets/img/cross-region-bv-backup06.png)

7. 복원된 볼륨의 이름을 입력하고 필요한 파라미터를 넣은 다음, **Create Block Volume**을 클릭합니다.

![]({{site.baseurl}}/assets/img/cross-region-bv-backup07.png)

8. 대상 region의 블록 볼륨 섹션에서 복원된 볼륨을 사용할 수 있는지 확인합니다.

![]({{site.baseurl}}/assets/img/cross-region-bv-backup08.png)


더 자세한 내용은 [Oracle Cloud Infrastructure 시작 안내서](https://docs.us-phoenix-1.oraclecloud.com/Content/GSG/Concepts/baremetalintro.htm), [블록 볼륨 서비스 개요](https://cloud.oracle.com/en_US/bare-metal-storage/block-volume/features), [블록 볼륨 설명서](https://docs.cloud.oracle.com/iaas/Content/Block/Concepts/overview.htm) 및 [FAQ](https://cloud.oracle.com/storage/block-volume/faq)를 참조하십시오.


## 참고

이 포스트의 원문은 [Cross-Region Block Volume Backups for Business Continuity, Migration, and Expansion](https://blogs.oracle.com/cloud-infrastructure/cross-region-block-volume-backups-for-business-continuity%2c-migration%2c-and-expansion) 입니다.
