---
layout: post
title: "OCI - Autoscaling 테스트"
date: 2019-03-04 12:00:00 +0900
description: "OCI의 Autoscaling 기능을 실제로 테스트 해봅시다."
img: "resourcemanager01.jpg"
tags: ["oracle", "oci", "iaas", "cloud", "autoscaling", "scaleout", "오토스케일", "오토스케일링", "metric", "demo","oracle cloud", "오라클 클라우드"] 
---

2019년 3월 Oracle Cloud Infrastructure에도 Autoscaling 기능이 GA 되었습니다.
Metric based Monitoring, Instance Configurations, Instance Pools와 연동하여 Instance를 Scale Out 할 수 있습니다.

본 포스트는 OCI의 Autoscaling이 동작하는 것을 보기 위한 간단한 DEMO를 보여줍니다.
먼저 기본적인 VCN(Virtual Cloud Network) 및 Instance를 만드는 방법은 TheKoguryo's 기술블로그 내용을 참조하시기 바랍니다.

- [OCI Compute VM 사용하기](https://thekoguryo.github.io/oci/chapter03/)


여기서는 jaykim-base-instance라는 이름의 VM.Standard2.1 shape의 Ubuntu 서버를 하나 만들었고 이것으로 Instance Config를 만들어 보겠습니다.
(Instance Configuration 및 Instance Pool에 대해서는 이전 저의 포스트 [템플릿을 이용한 손쉬운 Compute Instance 관리](https://jesamkim.github.io/instance_pool/)를 참조하시기 바랍니다.)

![]({{site.baseurl}}/assets/img/autoscaling01.png)

이 인스턴스의 구성으로 Instance Configuration을 생성하겠습니다.
위와 같이 Instance Details 화면의 우측 상단에 있는 **Create Instance Configuration** 버튼을 클릭합니다.

![]({{site.baseurl}}/assets/img/autoscaling02.png)

원하는 Instance Configuration Name을 지정한 후 **Create Instance Configuration** 버튼을 눌러서 완료 합니다.

![]({{site.baseurl}}/assets/img/autoscaling03.png)

위와 같이 Instance Configuration이 잘 생성되었습니다. 이제는 이것으로 Instance Pools를 만들어보겠습니다.
**Create Instance Pool** 버튼을 클릭합니다. Instance Pool의 이름을 지정하고, 최초 프로비저닝 할 Instance 수를 입력합니다. 여기서는 기본으로 1개의 Instance만 프로비저닝 하겠습니다.

![]({{site.baseurl}}/assets/img/autoscaling04.png)

![]({{site.baseurl}}/assets/img/autoscaling05.png)

위와 같이 Instance Pool이 생성되었으며 지정한만큼의 Instance가 생성되었습니다. (1개)

이제는 이 Instance Pool을 기반으로 Autoscaling 설정을 하겠습니다.
Instance Pool Details 화면 우측 상단의 **Action > Create Autoscaling Confguration**을 클릭합니다.

![]({{site.baseurl}}/assets/img/autoscaling06.png)

![]({{site.baseurl}}/assets/img/autoscaling07.png)
![]({{site.baseurl}}/assets/img/autoscaling08.png)

Autoscaling Policy 이름을 지정하고, Performance Metric을 선택합니다. (CPU 혹은 Memory) 여기서는 Cpu Utilization을 선택하였습니다.
그리고 최소 인스턴스 수(1), 최대 인스턴스 수(3), 초기 인스턴스 수(1) 및 Scaling Rule을 지정합니다.
여기서는 CPU 사용률이 70%가 넘으면 인스턴스가 1개 추가되고, CPU 사용률이 30% 이하가 되면 인스턴스를 1개 줄이는 설정 입니다.

![]({{site.baseurl}}/assets/img/autoscaling09.png)

위와 같이 Autoscaling Policy 가 생성되었습니다.


이제 Pool에 있는 프로비저닝 되어 있는 인스턴스에 부하를 주어 Autoscaling (Scale Out)이 되는지 보겠습니다.
Compute > Instance 메뉴로 이동합니다.

![]({{site.baseurl}}/assets/img/autoscaling10.png)

위와 같이 Instance-Pool로 프로비저닝 된 inst-bgwhr-jaykim-instance-pool-20190304 인스턴스 1개가 보입니다.

이 인스턴스에 접속해서 stress tool을 설치합니다.
~~~
    $ sudo apt install gcc
    $ sudo apt install make
    $ wget https://launchpad.net/ubuntu/+archive/primary/+sourcefiles/stress/1.0.4-2/stress_1.0.4.orig.tar.gz
    $ tar -xvf stress_1.0.4.orig.tar.gz
    $ cd stress_1.0.4
    $ ./configure && make && sudo make install
~~~

이제 ssh 접속 창을 2개 띄웁니다. 

창 1: Stress tool로 2-core 사용률 100% 부하 시작

    $ stress -c 2

창 2: top 실행

    $ top

![]({{site.baseurl}}/assets/img/autoscaling11.png)

CPU 부하를 주기 시작한 뒤, Instance Details 페이지의 아래쪽 Metrics를 보면 CPU 사용률이 증가함을 확인할 수 있습니다.

![]({{site.baseurl}}/assets/img/autoscaling12.png)

Autoscaling Policy에 설정된 300초가 지난 뒤, Compute > Instance Pools 에서 autoscaling 되는지 확인합니다.

![]({{site.baseurl}}/assets/img/autoscaling13.png)

Compute > Instance Pools 에서 Status가 Scaling으로 되어 있습니다.

![]({{site.baseurl}}/assets/img/autoscaling14.png)

그리고 Compute > Instances에 보면 자동으로 새로운 Instance (inst-vpbol-jaykim-instance-pool-20190304) 가 프로비저닝 됨을 확인할 수 있습니다.
마지막으로 stress tool을 종료 시킨 후, 300초 뒤에는 아래처럼 처음 pool에서 생성된 Instance(오래된)가 Terminated 됨을 볼 수 있습니다.

![]({{site.baseurl}}/assets/img/autoscaling15.png)



## 참조
- [OCI Documentation - Autoscaling](https://docs.cloud.oracle.com/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm)


** 해당 포스트는 개인적으로 시간을 할애하여 작성하였습니다. 내용에 오류가 있을 수 있으며, 글의 내용은 개인적인 의견 입니다.
