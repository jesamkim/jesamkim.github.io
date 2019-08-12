---
layout: post
title: "OCI - Windows 인스턴스 패스워드 Reset 방법"
date: 2019-08-12 12:00:00 +0900
description: "OCI에서 생성한 Windows Server 2012 이상 인스턴스의 패스워드를 리셋하는 방법에 대해 알아봅시다."
img: "win2k_vm_pw_reset_00.jpg"
tags: ["oracle", "oci", "windows", "instance", "vm", "password", "reset", "2012", "2016", "oracle cloud", "오라클 클라우드"] 
---

몇 달만의 OCI 포스팅 입니다!!
업무적으로 다른 영역으로 변경되어 포스팅 주제에 대해 고민이 있었습니다. (실은 지금도 고민중이에요)<br>
틈틈이 유용할 내용이 있으면 업데이트하고, 추후에는 지금 하고 있는 빅데이터 관련 포스팅도 할 예정 입니다.

이번 시간에는 OCI 에서 만든 Windows 인스턴스를 오랜 시간 접속하지 않아, 접속시 패스워드가 잠기는 경우가 있습니다.<br>
혹은 패스워드를 잊어버려서 접속하지 못하는 경우가 있을 수 있습니다.<br>
이럴 때, 다음과 같이 같은 서브넷에 Ubuntu 인스턴스를 하나 만들어서 Windows Server 인스턴스의 패스워드를 초기화 할 수 있습니다.<br>
(단, Windows 인스턴스가 AD에 의해 계정 관리 되지 않는다는 조건 입니다.)

**중요** : Windows 2012 및 2016 Server 에 해당되는 내용 입니다.
<br>
<br>
<br>
<br>
## 1단계 : 부팅 볼륨을 유지하면서(삭제하지 않고), Windows 인스턴스 Terminate

OCI Console 에서 Windows 인스턴스를 Terminate 하는데, 아래와 같이 Permanently delete the attached Boot Volume 에 체크 해제하고 Terminate Instance 버튼을 클릭합니다.

**중요** : 인스턴스의 모든 정보 (compartment, attached 된 block volume, subnet IP 혹은 static IP 정보)를 기록해두세요.

![]({{site.baseurl}}/assets/img/win2k_vm_pw_reset_01.jpg)
<br>
<br>
<br>
<br>
## 2단계 : Ubuntu 인스턴스 생성 후, Windows 인스턴스의 boot volume 연결 

Windows 인스턴스가 있던 subnet에서 Ubuntu 인스턴스를 하나 생성 합니다.<br>
(Image는 Canonical Ubuntu 16.04 선택)<br>
Shape은 기본(VM.Standard2.1)이면 충분합니다.

Ubuntu 인스턴스가 RUNNING 상태가 되면, OCI Console 에서 Windows 인스턴스의 boot volume을 block storage volume 으로 attach 하세요.<br>
(ISCSI 로 연결하며, Access 부분에 READ/WRITE를 선택해야 합니다.)

![]({{site.baseurl}}/assets/img/win2k_vm_pw_reset_02.jpg)
<br>
<br>
<br>
<br>
## 3단계 : Ubuntu 인스턴스에서 ISCSI 로그인 명령 실행

Ubuntu 인스턴스에 ssh 접속한 뒤 iSCSI 접속 명령을 입력할 겁니다.<br>
OCI Console에서 볼륨에 연결된 인스턴스의 세부 정보 페이지에서 ISCSI 명령을 확인할 수 있습니다.<br>
볼륨 행에서 Actions 아이콘을 클릭하고 iSCSI Information을 클릭한 뒤, 아래와 같은 커맨드를 copy 합니다.

예시)
~~~
    $ sudo iscsiadm -m node -o new -T iqn.2015-02.oracle.boot:uefi -p 169.254.2.2:3260
    $ sudo iscsiadm -m node -o update -T iqn.2015-02.oracle.boot:uefi -n node.startup -v automatic
    $ sudo iscsiadm -m node -T iqn.2015-02.oracle.boot:uefi -p 169.254.2.2:3260 -l
~~~

iSCSI로 Windows 인스턴스 볼륨이 정상적으로 붙었으면 lsblk 명령어로 /dev/sdb 가 보일 것 입니다.
<br>
<br>
<br>
<br>
## 4단계 : 파티션 레이아웃 확인

다음 명령을 실행하면 예제와 비슷한 결과를 볼 수 있습니다.

**$ sudo sfdisk -l /dev/sdb**

~~~
    Disk /dev/sdb: 256 GiB, 274877906944 bytes, 536870912 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 4096 bytes
    I/O size (minimum/optimal): 4096 bytes / 1048576 bytes
    Disklabel type: gpt
    Disk identifier: 3D757AFA-F8FA-4592-A8FE-5D6A19E2FA6C

    Device       Start       End   Sectors   Size Type
    /dev/sdb1     2048    616447    614400   300M Windows recovery environment
    /dev/sdb2   616448    821247    204800   100M EFI System
    /dev/sdb3   821248   1083391    262144   128M Microsoft reserved
    /dev/sdb4  1083392 536868863 535785472 255.5G Microsoft basic data
~~~
<br>
<br>
<br>
<br>
## 5단계 : NTFS 볼륨 확인

**$ sudo ntfsfix /dev/sdb4**

~~~
    Mounting volume... OK
    Processing of $MFT and $MFTMirr completed successfully.
    Checking the alternate boot sector... OK
    NTFS volume version is 3.1.
    NTFS partition /dev/sdb4 was processed successfully.
~~~

**$ sudo mkdir -p /media/windows**

**$ sudo mount /dev/sdb4 /media/windows**
<br>
<br>
<br>
<br>
## 6단계 : chntpw 설치

**$ sudo apt-get update**

**$ sudo apt-get install chntpw**
<br>
<br>
<br>
<br>
## 7단계 : chntpw로 Windows Administrator 패스워드 리셋

**$ chntpw /media/windows/Windows/System32/config/SAM -u opc**

~~~
    ubuntu@linux-instance-1:~$ chntpw /media/windows/Windows/System32/config/SAM -u opc
    chntpw version 1.00 140201, (c) Petter N Hagen
    Hive </media/windows/Windows/System32/config/SAM> name (from header): <\SystemRoot\System32\Config\SAM>
    ROOT KEY at offset: 0x001020 * Subkey indexing type is: 686c <lh>
    File size 65536 [10000] bytes, containing 7 pages (+ 1 headerpage)
    Used for data: 329/31960 blocks/bytes, unused: 28/8776 blocks/bytes.

    ================= USER EDIT ====================

    RID     : 1000 [03e8]
    Username: opc
    fullname:
    comment :
    homedir :

    00000220 = Administrators (which has 2 members)

    Account bits: 0x0010 =
    [ ] Disabled        | [ ] Homedir req.    | [ ] Passwd not req. |
    [ ] Temp. duplicate | [X] Normal account  | [ ] NMS account     |
    [ ] Domain trust ac | [ ] Wks trust act.  | [ ] Srv trust act   |
    [ ] Pwd don't expir | [ ] Auto lockout    | [ ] (unknown 0x08)  |
    [ ] (unknown 0x10)  | [ ] (unknown 0x20)  | [ ] (unknown 0x40)  |

    Failed login count: 2, while max tries is: 8
    Total  login count: 2

    - - - - User Edit Menu:
    1 - Clear (blank) user password
    (2 - Unlock and enable user account) [seems unlocked already]
    3 - Promote user (make user an administrator)
    4 - Add user to a group
    5 - Remove user from a group
    q - Quit editing user, back to user select

    Select: [q] >
~~~

**> 1**

~~~
    Password cleared!
    ================= USER EDIT ====================

    RID     : 1000 [03e8]
    Username: opc
    fullname:
    comment :
    homedir :

    00000220 = Administrators (which has 2 members)

    Account bits: 0x0010 =
    [ ] Disabled        | [ ] Homedir req.    | [ ] Passwd not req. |
    [ ] Temp. duplicate | [X] Normal account  | [ ] NMS account     |
    [ ] Domain trust ac | [ ] Wks trust act.  | [ ] Srv trust act   |
    [ ] Pwd don't expir | [ ] Auto lockout    | [ ] (unknown 0x08)  |
    [ ] (unknown 0x10)  | [ ] (unknown 0x20)  | [ ] (unknown 0x40)  |

    Failed login count: 2, while max tries is: 8
    Total  login count: 2
    ** No NT MD4 hash found. This user probably has a BLANK password!
    ** No LANMAN hash found either. Try login with no password!

    - - - - User Edit Menu:
    1 - Clear (blank) user password
    (2 - Unlock and enable user account) [seems unlocked already]
    3 - Promote user (make user an administrator)
    4 - Add user to a group
    5 - Remove user from a group
    q - Quit editing user, back to user select
    Select: [q] >
~~~

**> 2**

~~~
    Unlocked!

    ...

    Select: [q] >
~~~

**> q**

~~~
    Hives that have changed:
    #  Name
    0  </media/windows/Windows/System32/config/SAM>
    Write hive files? (y/n) [n] : 
~~~

**> y**

~~~
     0  </media/windows/Windows/System32/config/SAM> - OK
~~~
<br>
<br>
<br>
<br>
## 8단계 : 빈 암호 로그인이 허용되도록 레지스트리 설정 편집

**$ chntpw -e /media/windows/Windows/System32/config/SYSTEM**

**> ls Select**

~~~
    Node has 0 subkeys and 4 values
      size     type              value name             [value if type DWORD]
         4  4 REG_DWORD          <Current>                  1 [0x1]
         4  4 REG_DWORD          <Default>                  1 [0x1]
         4  4 REG_DWORD          <Failed>                   0 [0x0]
         4  4 REG_DWORD          <LastKnownGood>            2 [0x2]
~~~

현재 필드 값을 기록해두세요.

**> cd ControlSet001\Control\Lsa\\**

위에서 Current가 0x1인지 0x2 인지에 따라 ControlSet001 또는 ControlSet002를 사용하세요.

**> ed LimitBlankPasswordUse**

~~~
    EDIT: <LimitBlankPasswordUse> of type REG_DWORD (4) with length 4 [0x4]
    DWORD: Old value 1 [0x1], enter new value (prepend 0x if hex, empty to keep old value)
~~~

**> 0x0**

**> q**

~~~
    Hives that have changed:
    # Name
    0 
    Write hive files? (y/n) [n] :
~~~

**> y**
<br>
<br>
그리고 iSCSI로 attached 된 Windows 인스턴스 boot 볼륨을 detach 합니다.

**$ sudo umount /media/windows**

**$ sudo iscsiadm -m node -T iqn.2015-02.oracle.boot:uefi -p 169.254.2.2:3260 -u**

**$ sudo iscsiadm -m node -o delete -T iqn.2015-02.oracle.boot:uefi -p 169.254.2.2:3260**
<br>
<br>
<br>
<br>
## 9단계 : Windows 인스턴스 새로 생성

Create Instance 메뉴를 통해 Windows 인스턴스를 다시 만드세요. 이때, Image는 기존의 boot volume을 선택해야 합니다.<br>
(그외 기타 파라미터가 있다면 1단계에서 기록한 내용으로 함께 설정하세요.)

![]({{site.baseurl}}/assets/img/win2k_vm_pw_reset_03.jpg)


이제 새로 만든 Windows 인스턴스에 opc로 접속하면 initial password를 설정하게 됩니다.<br>
패스워드 설정 후, 보안을 위해 빈 비밀번호가 허용되지 않도록 아래의 레지스트리를 수정하세요.

**HKEY_LOCAL_MACHINE\System\Currentcontrolset\Control\Lsa\Limitblankpassworduse 키를 1로 설정**
<br>
<br>
이로서 OCI에서 Windows 인스턴스 (2012, 2016)의 패스워드를 리셋하는 방법에 대해 알아보았습니다.
<br>
<br>
<br>
## 참조
- 본 내용에 대한 원문은 [Tutorial: How to Reset a Forgotten Password for a Windows Instance](https://blogs.oracle.com/cloud-infrastructure/tutorial:-how-to-reset-a-forgotten-password-for-a-windows-instance) 입니다.


** 해당 포스트는 개인적으로 시간을 할애하여 작성하였습니다. 내용에 오류가 있을 수 있으며, 글의 내용은 개인적인 의견 입니다.
