---
layout: post
title: "Tagging을 이용한 Cost Tracking"
date: 2018-11-06 14:00:00 +0900
description: "Tracking Costs with Oracle Cloud Infrastructure Tagging"
img: "tracking_cost00.png"
tags: ["oracle", "oci", "iaas", "cloud", "tagging", "cost", "tracking", "tag", "oracle cloud", "오라클 클라우드"] 
---
![]({{site.baseurl}}/assets/img/tracking_cost01.png)

Oracle Cloud Infrastructe를 사용함에 있어 부서별 혹은 Cost Center에 따른 비용 산정은 중요합니다.<br>
Oracle Cloud Infrastrucre를 사용하면 **My Services Dashboard**를 사용하여 서비스 또는 Compartment 수준에서 비용을 tracking 할 수 있지만, multiple compartment 또는 compartment를 공유하는 프로젝트의 비용 또한 tracking 할 수 있는 유연성도 필요합니다.

이를 고려하여 cost tracking tag를 사용하여 사용자, 프로젝트, 부서 또는 청구 목적으로 선택한 기타 메타 데이버별로 태그를 지정할 수 있습니다. **Cost Tracking Tag**는 My Services Dashboard의 온라은 보고서에 표시되는 정의된 태크의 한 유형 입니다. 이 기능은 제어가 손쉬운 스키마 기반 태그 지정 방식을 기반으로 합니다. 다른 클라우드는 자유 형식의 태그를 지원하지만 Oracle Cloud Infrastructure는 정의된(defined) 태그를 제공하여 보다 효과적으로 제어할 수 있습니다. 정의된 태그는 tagging을 제어할 수 있으며, 스팸을 방지하는데 도움이되는 스키마를 지원합니다. 적잘한 사용 및 청구관리를 보장 합니다.


# Creating Cost Tracking Tags

Cost Tracking 태그를 생성하는 방법과 Cost를 귀속시킬 수 있는 태그가 어떻게 구성되는지 살펴보겠습니다.<br>
먼저 Oracle Cloud Infrastructure Console 에서 **Tag Namespaces**를 살펴보겠습니다.

**OCI Console > MENU > Governance > Tag Namespaces**

![]({{site.baseurl}}/assets/img/tracking_cost02.png)

**Create Namespace Definition**을 클릭하여 새로운 Namespace Definition을 생성합니다.
그리고 생성한 Namespace Definition(여기서 이름은 CostTracking으로 생성하였음)을 클릭합니다. 

**Number of Cost-tracking Tags**를 확인해보세요. 이 값은 태그 네임스페이스의 모든 cost tracking tag definition을 보여줍니다. 최대 10개의 cost tracking definition을 가질수 있습니다.

이제 나만의 cost tracking tag를 설정하는 방법을 살펴보겠습니다. 여기서는 예시로 cost를 4가지 방법으로 추적할 것이므로, 다음의 4가지 definition으로 cost tracking tag를 설정하였습니다.

* **CostCenter**는 비용이 발생되는 내부 부서 입니다.
* **Project**는 고객을 단일 제품 오퍼링에 함께 그룹화 합니다.
* **Customer**는 사용료가 청구되는 고객 입니다.
* **Customer_Job**은 Compute 인스턴스에서 실행중인 actual job 입니다.

![]({{site.baseurl}}/assets/img/tracking_cost03.png)

위와 같은 Tag Key Definition은 위에서 생성한 CostTracking이라는 Namespace Definition을 클릭하여 **Create Tag Key Definition**을 클릭하여 만들 수 있습니다. Cost-Tracking을 위한 것이라면 생성시에 **COST-TRACKING**에 체크해주어야 합니다.

이 태그 중 3개는 이미 **Cost-Tracking**을 **Yes**로 설정하여 Oracle의 청구 시스템으로 전송되었음을 나타냅니다.
**Customer_Job**의 **Cost-Tracking**이 **No**로 설정되었으므로, 이 정의된 태그를 cost-tracking tag로 변환해야 합니다.
이를 위해 **Customer_Job** tag key definition을 열고, **Cost-tracking: No** 옆에 있는 연필 아이콘을 클릭하여, **Cost-tracking** 체크박스를 선택합니다.

![]({{site.baseurl}}/assets/img/tracking_cost04.png)

![]({{site.baseurl}}/assets/img/tracking_cost05.png)

이러한 tag key definition이 cost tracking으로 설정되었으므로, 태그는 대시보드에 있는 My Services로 전송되는 사용 데이터에 포함됩니다. 태그가 cost tracking으로 마크되면 My Services로 처리되고 온라인 성명서에 포함되기까지 2~4시간이 걸릴 수 있습니다.

Compute Instance, Object Storage Bucket 및 Block Storage 등을 생성할 때, Tag 부분에서 위에서 정의한 Tag Key Definition 및 그에 대한 적절한 값들을 입력합니다.


# Viewing Cost Tracking Tags in My Services

이제 My Services 대시보드에서 이러한 태그를 볼 수 있습니다.

**OCI Console > MENU > Administration > My Services Dashboard**

Account Management를 클릭하고, filter by tags 칸을 클릭하여 cost-tracking 기반 filter를 선택하세요. 다음 스크린샷과 같이 정의된 cost tracking tag를 기준으로 cost를 필터링하고 특정 cost center(예: Finance department)의 cost를 결정할 수 있습니다. 이 예제에서는 **Finance:CostCenter=w1234** 라는 태그를 사용하여 실행하던 데이터베이스와 관련된 비용을 보여줍니다.

![]({{site.baseurl}}/assets/img/tracking_cost06.png)

My Services Dashboard에서 이 정보를 확인할 수 있을 뿐만 아니라 결과를 CSV 파일로 다운로드 할 수 있으므로 Excel 또는 기타 도구로 분석하는데 이상적 입니다.

API를 사용하여 태그로 cost tracking 데이터를 수집하는 프로세스를 자동화하려는 경우에도 이를 수행할 수 있습니다. [API documentation](https://docs.oracle.com/en/cloud/get-started/subscriptions-cloud/meter/op-api-v1-usagecost-accountid-tagged-get.html)을 사용하여 시작할 수 있습니다. 다음은 명시적인 예시 입니다.

    https://itra.oraclecloud.com/metering/api/v1/usagecost/cacct-{your caact}/tagged?
    startTime=2018-09-01T00:00:00.000&endTime=2018-10-04T00:00:00.000&computeTypeEnabled=Y&tags=CostTracking%3ACostCenter%3Dw1234
    &timeZone=America/Los_Angeles&usageType=DAILY&rollupLevel=RESOURCE

URL은 특정 태그에 대해 필터링 중임을 나타내는 **/tagged**가 포함됩니다. 태그 필드는 URL로 인코딩 되어야 합니다. 즉, 콜론(**:**)을 **%3A**로 변환하고 동일한 기호(**=**)를 **%3D**로 변환해야 합니다.
이 예에서는**CostTracking:CostCenter=w1234**를 사용했습니다. URL 인코딩은 **CostTracking%3ACostCenter%3Dw1234** 입니다.

다음 예제에서는 **Finance:CostCenter=w1234**라는 태그를 사용하여 실행중인 데이터베이스와 관련된 비용을 보여줍니다.

~~~
    {
    "accountId": "cacct-your caact",
    "canonicalLink": "/metering/api/v1/usagecost/cacct-caact /tagged?timeZone=America%2FLos_Angeles&startTime=2018-09-01T00%3A00%3A00.000&endTime=2018-10-04T00%3A00%3A00.000&computeTypeEnabled=Y&tags=Finance%3ACostCenter%3Dw1234&usageType=DAILY&rollupLevel=RESOURCE",
    "items": [
        {
            "costs": [
                {
                    "computedAmount": 19.44,
                    "computedQuantity": 48.0,
                    "overagesFlag": "N"
                }
            ],
            "currency": "USD",
            "endTimeUtc": "2018-09-19T17:00:00.000",
            "gsiProductId": "B88331",
            "resourceDisplayName": "Database Standard Added CPUs",
            "resourceName": "PIC_DATABASE_STANDARD_ADDITIONAL_CAPACITY",
            "startTimeUtc": "2018-09-18T17:00:00.000",
            "tag": "Finance:CostCenter=w1234"
        },
        ...
~~~


## 참조

해당 내용에 대한 원문은 [Tracking Costs with Oracle Cloud Infrastructure Tagging](https://blogs.oracle.com/cloud-infrastructure/tracking-costs-with-oracle-cloud-infrastructure-tagging) 입니다.
