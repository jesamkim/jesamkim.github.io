---
layout: post
title: "OCI-CLI 설치 (Oracle Linux 7)"
date: 2018-09-06 00:00:00 +0800
description: "OCI CLI를 Oracle Linux 7에 설치해 봅니다." # Add post description (optional)
img: oci_cli.png # Add image post (optional)
tags: [oracle, oci, iaas, cloud, cli] # add tag
---

## Prerequisites

- OS : Oracle Linux 7 (테스트 환경은 OL 7.4 과 7.5 에서 진행하였습니다)


## To Installation the CLI

OCI CLI는 간단하게 자동 설치할 수 있습니다.

	$ bash -c “$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)”
	
만약 manually 설치를 진행하는 경우는 아래와 같이 합니다.

	$ sudo yum -y install gcc libffi-devel python-devel openssl-devel
	$ sudo easy_install pip
	
	https://github.com/oracle/oci-cli/releases/download/v2.4.18/oci-cli-2.4.18.zip 다운로드 
	그리고 압축 풀고 해당 디렉토리 이동
	
	$ sudo pip install oci_cli-2.4.18-py2.py3-none-any.whl
	$ sudo pip install oci-cli
	

## OCI CLI setup

	$ oci setup config
	
위와 같이 실행하면 ~/.oci 디렉토리에 config 파일이 생성됩니다.
그리고 아래와 같은 정보를 요구합니다.

	1. User OCID
	2. Tenancy OCID
	3. Region (예: us-ashburn-1)
	4. CLI를 위한 key pair 생성 여부 
	   여기서 yes를 선택하면 자동으로 key pair 가 생성되며, public key는 OCI -> IAM -> User 에서 등록 해야 합니다. 
	   생성된 public key 파일 명은 oci_api_key_public.pem
	   

## Verifying OCI CLI

테스트 환경의 Object Storage 정보를 조회하는 예제를 보여드리겠습니다.

	- identity domain : gse00014914
	- Object Storage Bucket Name : STD_BK
	
Object Storage에서 이미 생성된 Bucket의 정보를 가져오는 경우에 대한 CLI 입니다.

	$ oci os bucket get -ns gse00014941 –bucket-name STD_BK
	
	{
	“data”: {
	“compartment-id”: “ocid1.compartment.oc1..aaaaaaaad2te2fvep7m6llxc67zzirimb2npecwndy36pt7v4xzly7v6adza”,
	“created-by”: “ocid1.saml2idp.oc1..aaaaaaaavztcratyyxajhg6j5kzgpwhgsnsvpukv3fyrdhrz3mjdhlpkjbeq/cloud.admin”,
	“defined-tags”: {},
	“etag”: “61169ca6-0f6b-4e17-a9c5-6c776baaa3da”,
	“freeform-tags”: {},
	“metadata”: {},
	“name”: “STD_BK”,
	“namespace”: “gse00014941”,
	“public-access-type”: “ObjectRead”,
	“storage-tier”: “Standard”,
	“time-created”: “2018-03-22T12:58:58.541000+00:00”
	},
	“etag”: “61169ca6-0f6b-4e17-a9c5-6c776baaa3da”
	}

정상적으롤 STD_BK 버킷의 정보를 읽어오는 것을 확인하실 수 있습니다.

OCI CLI에 대한 다양한 명령어는 [Oracle Cloud Infrastructure CLI Command Reference](https://docs.cloud.oracle.com/iaas/tools/oci-cli/latest/oci_cli_docs/index.html) 에서 확인하실 수 있습니다.
