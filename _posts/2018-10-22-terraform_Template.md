---
layout: post
title: "Terraform Template으로 Elastic Search 구축하기"
date: 2018-10-22 14:00:00 +0900
description: "Deploying Elasticsearch on Oracle Cloud Infrastructure Using a Terraform Template"
img: "terraform_template00.png"
tags: ["오라클", "클라우드", "oracle", "oci", "iaas", "cloud", "terraform", "template", "테라폼"] # add tag
---

이제 Terraform 템플릿을 사용하여 Oracle 고성능 클라우드에 분산된 RESTful 검색 및 분석 엔진인 Elastic Search를 구축할 수 있습니다.

이번 발표를 통해 Oracle Cloud Infrastructure는 파트너의 빅 데이터 ISV 에코시스템을 강화합니다. Oracle Cloud Infrastructure에 대한 Elastic Search를 시작하려면 [Terraform automated deployment template](https://github.com/cloud-partners/oci-elasticsearch)를 사용하세요. 템플릿은 인스턴스 및 스토리지를 프로비저닝하고, 소프트웨어를 배포 및 구성하며, 네트워킹과 로드 밸런서를 설정하고, 이룰 모두 시작하는 단계를 수행합니다.

# Deployment  Architecture

다음 다이어그램은 템플릿을 사용한 Elasticsearch 및 Kibana 배포를 보여줍니다.

![]({{site.baseurl}}/assets/img/terraform_template01.png)

**Bastion host:** Bastion host는 인터넷에서 소프트웨어를 업데이트하고 설치하기 위해 Elastic Search 마스터 및 데이터 노드의 NAT 인스턴스로 사용됩니다.

**Elasticsearch, master nodes:** 마스터 노드는 새 인덱스 생성 및 shard rebalancing과 같은 클러스터 관리 작업을 수행합니다. 이 노드들은 데이터를 저장하지 않습니다. 고가용성(HA)를 위해 3개의 가용성 도메인(AD)에 3개의 마스터 노드(더 큰 클러스터 권장)가 배포됩니다.

**Elasticsearch, data nodes:** 고가용성(HA)을 위해 데이터 노드 4개가 도 개의 가용성 도메인(각 가용성 도메인의 노드 2개)에 배포됩니다. Elastic Search는 사용 가능한 메모리 양에 따라 달라지기 때문에 메모리 최적화 컴퓨팅 인스턴스가 권장됩니다. 선택할 Compute Instance 목록을 [여기](https://cloud.oracle.com/compute)에서 사용할 수 있습니다. 각 데이터 노드는 200 GiB 블록 스토리지로 구성됩니다. Oracle Cloud Infrastructure는 VM 외에도 클러스터에 연결된 강력한 베어 메탈 인스턴스를 25 GB 미사용 네트워크 인프라에 제공합니다. 이 구성은 높은 성능 분산 스트리밍 워크로드의 핵심 요구사항인 짧은 대기 시간과 높은 처리량을 보장합니다. Oracle Cloud Infrastructure는 네트워크 처리량 성능 [SLA](https://cloud.oracle.com/iaas/sla) 를 갖춘 유일한 클라우드 입니다.

**Kibana:** 마스터 노드와 마찬가지로 키바나는 비교적 가벼운 리소스 요구 사항을 가지고 있습니다. 대부분의 연산은 Elastic Search로 푸쉬 됩니다. 이 배포에서 키바나는 마스터 노드에서 실행됩니다.

Template 배포를 커스터마이징하려면 다음 작업을 수행할 수 있습니다:

* 마스터 노드 및 데이터 노드 인스턴스의 shape를 선택합니다.

* 데이터 노드 인스턴스의 스토리지 용량 지정합니다.

* 가상 클라우드 네트워크 및 서브넷 및 기타 구성 설정에 대한 CIDR 블록 크기를 변경합니다.

Terraform 템플릿에 대한 자세한 내용은 [Readme.md 파일](https://github.com/cloud-partners/oci-elasticsearch/blob/master/README.md)을 참조하세요.


## 참조

원문은 [Deploying Elasticsearch on Oracle Cloud Infrastructure Using a Terraform Template](https://blogs.oracle.com/cloud-infrastructure/deploying-elasticsearch-on-oracle-cloud-infrastructure-using-a-terraform-template) 입니다.
