---
layout: post
title: "OCI Object Storage - Large file Upload"
date: 2018-09-10 00:00:00 +0800
description: "Object Storage 버킷에 5GB 이상 크기 파일을 CLI로 업로드 해봅니다." # Add post description (optional)
img: oci_cli.png # Add image post (optional)
tags: ["오라클", "클라우드", "oracle", "oci", "iaas", "cloud", "objectstorage", "cli", "오브젝트스토리지"] # add tag
---

OCI Object Storage는 web이나 cloudberry 등에서 upload 시 single file size가 5GB가 넘으면 업로드 되지 않습니다.
5GB 이상의 크기인 Large FIle의 경우, cli로 bulk upload 기능을 이용하여 올릴 수 있습니다.

다음은 7GB 파일을 업로드 하는 예제입니다. (파일 경로는 /home/opc/src 입니다)
먼저 업로드 하려는 Win2012R2.tar.gz 파일의 크기를 확인합니다.

	[opc@cli-test src]$ pwd
	/home/opc/src
	[opc@cli-test src]$
	[opc@cli-test src]$ ls -al
	total 7472924
	drwxrwxr-x.  2 opc opc         35 Aug 29 00:30 .
	drwx------. 11 opc opc       4096 Aug 29 00:30 ..
	-rw-rw-r--.  1 opc opc 7652267479 Aug 29 00:24 Win2012R2.tar.gz
	[opc@cli-test src]$ cd ..
 	[opc@cli-test ~]$


그리고 oci-cli 명령어를 사용해 Win2012R2.tar.gz 파일이 있는 폴더 통째로 업로드 합니다.

	[opc@cli-test ~]$ oci os object bulk-upload --bucket-name CloudBerry --src-dir /home/opc/src
	Uploaded Win2012R2.tar.gz  [####################################]  100%
	{
		"skipped-objects": [],
		"upload-failures": {},
		"uploaded-objects": {
		"Win2012R2.tar.gz": {
			"etag": "7489155231F0C9FCE0538210C00A8AAF",
			"last-modified": "Wed, 29 Aug 2018 00:32:58 GMT",
			"opc-multipart-md5": "m/zVgxVE+NXtJIrPVdVqyg==-58"
		}
		}
	}
	[opc@cli-test ~]$
	

위와 같이 oci-cli로 7GB인 Win2012R2.tar.gz 파일을 성공적으로 업로드 하였습니다.

웹에서도 정상적으로 파일이 업로드 되었음이 확인되었습니다.

![]({{site.baseurl}}/assets/img/largefile_upload.png)



** 해당 포스트는 개인적으로 시간을 할애하여 작성하였습니다. 내용에 오류가 있을 수 있으며, 글의 내용은 개인적인 의견 입니다.
