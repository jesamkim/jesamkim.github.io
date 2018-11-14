---
layout: post
title: "향상된 성능을 위한 반가상화 모드"
date: 2018-11-14 14:00:00 +0900
description: "Bring Your Own Custom Image in Paravirtualized Mode for Improved Performance"
img: "featured_oracle_cloud_2.jpg"
tags: ["oracle", "oci", "iaas", "cloud", "반가상화", "custom", "image", "paravitualization", "oracle cloud", "오라클 클라우드"] 
---

Oracle Cloud Infrastructure에서는 반가상화 하드웨어 모드(Paravirtualized hardware mode)를 사용하여 다양한 새로운 운영체제 및 레거시 운영체제를 가져올 수 있습니다. 반가상화 디바이스를 사용하는 VM은 에뮬레이션된 모드에서 실행하는 것에 비해 훨씬 빠른 성능을 제공하며, 최소 6배 더 빠른 Disk I/O 성능을 제공합니다.

기존 가상화 환경에서 VM을 내보낸 후(export), Oracle Cloud Infrastructure로 직접 가져오는 것으로 시작합니다.
이미지를 QCOW2 또는 VMDK 형식으로 가져올(import) 수 있습니다.

아래 스크린샷은 반가상화 모드로 이미지를 가져오도록 선택할 수 있는 이미지 가져오기 대화상자를 보여줍니다.
이미지를 성공적으로 가져오면 이 이미지로 새 VM 인스턴스를 시작할 수 있습니다.

![]({{site.baseurl}}/assets/img/pv_import___image_import.png)

반가상화 모드는 kvm virtio 드라이버가 포함된 Linux OS를 구동하는 X5 및 X7 VM에서 사용할 수 있습니다. Linux Kernel versions 3.4 이상에는 기본적으로 드라이버가 포함되어 있습니다. 여기에는 Oracle Linux 6.9 이상, Redhat Enterprise Linux 7.0 이상, CentOS 7.0 이상 및 Ubuntu 14.04 이상이 포함됩니다. 에뮬레이션 모드를 사용하여 이전 Linux OS 및 Windows OS 이미지를 가져오는 것이 좋습니다.

이미지가 반가상화 드라이버를 지원하는 경우, 기존 에뮬레이션 모드 인스턴스를 반가상화된 인스턴스로 쉽게 변환할 수 있습니다. 인스턴스의 custom image를 생성한 후 object storage로 내보낸 후 반가상화 모드로 다시 가져올 수 있습니다.

반가상화 모드에서 Bring your own custom image는 추가비용 없이 모든 region에서 제공됩니다. 이제 성능 향상과 함께 기존 워크로드를 Oracle Cloud Infrastructure로 사져올 수 있는 또다른 방법입니다!

기존 OS images 가져오기:

* **[NEW] Bring your own custom image to Oracle Cloud Infrastructure using paravirtualized mode:** VMDK 또는 QCOW2 형식을 사용하여 기존 Linux OS 이미지를 가져온 후 반가상화 모드 VM에서 실행하여 성능 향상. 자세한 내용은 [Bring Your Own Custom Image for Paravirtualized Mode Virtual Machines](https://docs.cloud.oracle.com/iaas/Content/Compute/References/customimagesparavirtualizedmode.htm)를 참조하세요.

* **Bring your own custom image to Oracle Cloud Infrastructure VMs using emulation mode:** VMDK 또는 QCOW2 형식을 사용하여 기존 Linux OS 이미지를 가져오고 에뮬레이션 모드 VM에서 실행합니다. 자세한 내용은 [Bring Your Own Custom Image for Emulation Mode Virtual Machines](https://docs.cloud.oracle.com/iaas/Content/Compute/References/customimagesemulationmode.htm)를 참조하세요.

* 기존 하이퍼바이저, 관리 툴 및 프로세스를 사용하여 전체 가상화된 워크로드를 Oracle Cloud Infrastructure로 이동합니다. OS 이미지 또는 Ubuntu 6.x, Redhat Enterprise Linux 3.x 또는 CentOS 5.4와 같은 레거시 OS에서는 Oracle Cloud Infrastructure 베어메탈 인스턴스에서 KVM을 사용할 수 있습니다. 자세한 내용은 [Bring Your Own KVM](https://cloud.oracle.com/iaas/whitepapers/installing_kvm_multi_vnics.pdf)와 [Bring Your Own Nested KVM on VM Shapes](https://blogs.oracle.com/cloud-infrastructure/nested-kvm-virtualization-on-oracle-iaas)를 참조하세요.

* **Bring Your Own Oracle VM:** 자세한 내용은 [Oracle VM on Oracle Cloud Infrastructure](https://cloud.oracle.com/iaas/whitepapers/ovm-on-oci.pdf)를 참조하세요.


혹은 새로운 OS images 만들기 :

* **Oracle Cloud Infrastructure Published OS images:** 오라클은 Oracle Linux, Microsoft Windows, Ubuntu 및 CentOS에 대해 미리 작성된 이미지를 제공합니다. 자세한 내용은 [Oracle-provided images](https://docs.us-phoenix-1.oraclecloud.com/Content/Compute/References/images.htm)의 complete list를 참조하세요.

* **Red Hat Enterprise Linux 7.4 to Oracle Cloud Infrastructure bare metal and VM instances:** [Terraform provider](https://github.com/nelsonse/terraform-provider-oci/tree/master/docs/solutions/rhel74_image)에서 사용할 수 있는 Terraform templete을 사용하여 베어 메탈 및 VM 인스턴스에 대한 Red Hat Enterprise Linux 7.4 이미지를 생성할 수도 있습니다.


Custom images를 포함한 운영중인 워크로드를 Oracle Cloud Infrastructure로 가져오는 방법에 대한 자세한 내용은 [Bring Your Own Image](https://docs.cloud.oracle.com/iaas/Content/Compute/References/bringyourownimage.htm)를 참조하세요.


## 참고

원문은 [Bring Your Own Custom Image in Paravirtualized Mode for Improved Performance](https://blogs.oracle.com/cloud-infrastructure/bring-your-own-custom-image-in-paravirtualized-mode-for-improved-performance) 입니다.
