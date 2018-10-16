---
layout: post
title: "Object Storage Lifecycle Management 소개"
date: 2018-10-16 14:00:00 +0900
description: "OCI Object Storage Lifecycle Management."
img: "lcp_create_thumbnail.jpg"
tags: ["오라클", "클라우드", "oracle", "oci", "iaas", "cloud", "object", "storage", "lifecycle", "management", "오브젝트", "스토리지", "라이프사이클"] # add tag
---

![]({{site.baseurl}}/assets/img/lcp_create_main.jpg)

모든 저장된 데이터가 동일한 중요도를 가지거나 미션 크리티컬한 것은 아닙니다. 이는 Oracle Cloud Infrastructure 스토리지 포트폴리오에 [Archive Storage](https://blogs.oracle.com/cloud-infrastructure/archiving-your-data-to-the-cloud-just-got-a-whole-lot-easier)를 도입했을 때의 운영상의 가정이었습니다. 아카이브 스토리지는 데이터를 Hot 또는 Cold로 분류한 다음 해당 분류를 기준으로 Standard Bucket 또는 Archive Bucket에 저장할 수 있는 유연성을 제공합니다. 이에 대한 benefit은 비용 절감일 것입니다. 아카이브 스토리지에 데이터를 저장하는 것은 standard 오브젝트 스토리지에 데이터를 저장하는 것보다 90배 더 저렴합니다.

즉, 데이터가 항상 Hot 또는 Cold 범주에 딱 맞게 혹은 영구적으로 분류되는 것은 아닙니다. Lifecycle이 Hot 상태로 시작되는 데이터(자주/빠르게 액세스 해야 함)는 시간이 지남에 따라 수요가 감소할 수 있습니다. (이때는 Archive Cold 스토리지에 적합) 스토리지 비용을 줄이기 위해 데이터를 삭제해야 하는 경우도 있습니다. Lifecycle 전체에 걸쳐 적극적으로 데이터 배치를 관리하면 전체 스토리지 비용을 크게 절감할 수 있습니다. 그러나 효과적인 데이터 관리 도구가 없이 데이터 lifecycle을 관리하면 상당한 운영 오버헤드가 발생할 수 있습니다.

이러한 스토리지 관리 문제를 완화하기 위해 Oracle Cloud Infrastructure의 Object Lifecycle Management 기능이 출시되었습니다.

Object lifecycle management를 사용하면 버킷에 저장된 개체가 시간에 따라 자동으로 관리되는 방식을 제어할 수 있도록 버킷에 대한 수명 주기 정책을 정의할 수 있습니다. 개체의 수명 주기 관리를 제어하는 각 버킷에 대해 최대 1,000개의 개별 규칙을 생성할 수 있습니다. Object lifecycle management는 개체를 아카이브하는 규칙 및 개체를 삭제하는 규칙이라는 두 가지 유형의 규칙을 제공합니다. 첫 번째의 경우 Object Storage는 개체의 스토리지 계층을 일(day) 단위로 standard object storage에서 archive storage로 변경합니다. 지정된 데이터가 지정된 일수가 경과한 후 삭제된다는 점을 제외하고 개체를 삭제하는 규칙은 동일한 방식으로 작동합니다. 버킷에 저장된 모든 개체에 적용되는 규칙을 정의하거나 지정된 개체 이름 접두사 패턴을 포함하는 개체 하위 집합에서만 작동하는 규칙을 정의할 수 있습니다.

Lifecycle policy에서 규칙을 혼합하고 일치시켜 특정 수명 주기 관리 동작을 수행할 수 있습니다. 예를 들어 데이터 생성 후 30일 후에 "ABC"라는 이름 접두사가 포함된 개체를 standard object storage에서 archive storage로 자동으로 마이그레이션 한 다음 동일한 개체 그룹을 생성 후 120일 후에 삭제할 수 있습니다. Archive storage 버킷의 데이터의 경우 삭제 규칙만 정의할 수 있습니다.


## Sample Lifecycle policy
~~~
    [
        {
            "name": “Archive ABC”,
            "action": "ARCHIVE",
            "objectNameFilter": {
                "inclusionPrefixes": [
                    “ABC”
                ]
            },
            "timeAmount": 30,
            “timeUnit”: “DAYS”,
            "isEnabled": true
        },
        {
            "name": “DELETE_ABC”,
            "action": "DELETE",
            "objectNameFilter": {
                "inclusionPrefixes": [
                    “ABC”
                ]
            },
            "timeAmount": 120,
            “timeUnit”: “DAYS”,
            “isEnabled": true
        }
    ]
~~~

규칙을 생성한 후 수정할 수 있습니다. 규칙에 대한 모든 변경 사항은 즉시 적용됩니다. 규칙은 런타임 충돌에 대해 평가되고 객체를 삭제하는 규칙은 항상 동일한 객체를 보관하는 규칙보다 우선합니다. CLI 또는 SDK 또는 API를 사용하여 규칙을 수정하거나 기존 수명 주기 정책에 추가하려면 이전에 정의한 모든 버킷 및 편집되지 않은 추가 규칙을 포함하여 버킷의 전체 수명 주기 정책을 다시 작성해야 합니다. (정책에서 이전에 정의한 규칙이 다시 생성되지 않으면 해당 규칙을 덮어씁니다.)

또한 개별 규칙을 쉽게 추가, 편집 및 제거할 수 있는 Oracle Cloud Infrastructure Console을 사용하여 lifecycle policy를 편집할 수 있습니다.

Oracle Cloud Infrastructure Console을 사용하여 수명 주기 정책을 생성하려면 콘솔에 로그인하고 Lifecycle policy를 정의할 Object Storage bucket을 선택하세요.

왼쪽 아래에 표시된 리소스 목록에서 **Lifecycle Policy Rules**를 클릭하세요.

![]({{site.baseurl}}/assets/img/lcp_create.jpg)

**Create Rule**을 클릭하고 **Create Lifecycle Rule** 대화상자에서 규칙 이름, 작업 유형 및 수명 주기 규칙이 활성화되기 전의 데이터 기간을 지정하세요. 선택적으로, 버킷에 저장된 개체의 하위 집합에만 규칙을 적용하려는 경우에도 개체 접두사를 지정할 수 있습니다. **Create**를 누르면 버킷에 수명 주기 정책 역할이 생성됩니다.

![]({{site.baseurl}}/assets/img/setrule.jpg)

Object Lifecycle Management에 대한 자세한 내용은 storage [FAQ](https://cloud.oracle.com/storage/object-storage/faq) 및 Object Lifecycle Management [documentation](https://docs.cloud.oracle.com/iaas/Content/Object/Tasks/usinglifecyclepolicies.htm)을 참조하세요.


## 참조
원문은 [Introducing Object Storage Lifecycle Management Policies](https://blogs.oracle.com/cloud-infrastructure/introducing-object-storage-lifecycle-management-policies) 입니다.
