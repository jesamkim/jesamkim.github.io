---
layout: post
title: "OCI에서 Windows AD Domain Controller 생성하기"
date: 2019-01-23 14:00:00 +0900
description: "Creating a Windows Active Directory Domain Controller in Oracle Cloud Infrastructure"
img: "windows_ad01.png"
tags: ["oracle", "oci", "iaas", "cloud", "windows", "ad", "active", "directory", "domain", "controller", "oracle cloud", "오라클 클라우드"] 
---

새로운 Windows AD(Active Directory) 환경을 생성할 수 있는 몇 가지 상황이 있으며, 이 포스트에서는 Oracle Cloud Infrastructure를 사용하여 새로운 AD 도메인 컨트롤러를 구축하는 방법에 대해 설명하며, Microsoft PowerShell과 Oracle Cloudbase-init 스크립트를 사용하면 프로세스를 자동화하고 Windows AD를 구축하는데 따라는 어려움을 줄일 수 있습니다.
Oracle Cloud Infrastructure Console에서 호스트를 생성할 때 스크립트를 **Advanced** option의 **User data** 색션에 배치할 수 있습니다.
<br><br>
이 포스트에서는 AD 도메인 컨트롤러의 자동화에 대해서만 다루며 VCN(Virtual Cloud Network) 및 네트워크 환경에 대해서는 설명하지 않습니다. Creating Windows Adtive Directory Domain Servers in  Oracle Cloud Infrastructure white paper에서 더 많은 것을 배울 수 있습니다.
다음 다이어그램은 VCN 및 서브넷을 구축하는 방법에 대한 아키텍처를 보여줍니다.

![]({{site.baseurl}}/assets/img/windows_ad01.png)


# Automating the Deployment of the AD Domain Controller

도메인을 계획할 때 첫 번째 작업은 forest를 구조화할 방법을 결정하는 것입니다. 수많은 sub domain과 trust dependencies가 있는 AD forest 구축은 복잡해질 수 있으므로 여기서는 tree가 있는 간단한 forest를 사용하여 간단하게 진행할 것입니다.

forest에 대한 더 많은 정보는 [Microsoft documentation](https://docs.microsoft.com/en-us/windows/desktop/ad/active-directory-domain-services)을 참조하세요.


## Scripting the Host Deployment

PowerShell 코드를 살펴보겠습니다. 먼저 cloudbase-init 스크립트의 헤더가 올바른지 확인하세요. cloudbase-init 및 PowerShell이 올바른 모드에서 수행되려면 ps1_sysnative 모드에서 실행해야 합니다.

~~~
    #ps1_sysnative
~~~

그런 다음 로컬 관리자 암호를 설정하고 관리자(Administrator) 계정을 활성화 하세요. (Oracle Cloud Infrastructure Windows 이미지에서는 기본적으로 비활성화 됨) 계정이 복제되고 복제본이 도메인 관리자가 되므로 암호가 안전하고 특수문자, 숫자 및 대문자와 소문자 혼합 표준을 충족하는지 확인하세요. 아래의 암호는 임시 암호이므로 나중에 프로세스에서 암호를 변경할 것입니다.

~~~
    $password="P@ssw0rd123!!"
    # Set the Administrator Password and activate the Domain Admin Account
    net user Administrator $password /logonpasswordchg:no /active:yes
~~~

관리자 계정을 활성화 한 후 AD 도메인 서비스의 필수 구성 요소를 설치합니다. 관련된 Windows feature 및 role의 첫째는 이전 버전 호환성이 있는 .NET Framework 3.0 입니다. 다음은 AD의 핵심 관리 기능인 AD Domain Services 및 Remote Active Directory Service 입니다. 마지막으로 DNS 서비스를 설치하면 AD 도메인 내의 대부분의 통신 문제가 완화됩니다.

~~~
    Install-WindowsFeature NET-Framework-Core
    Install-WindowsFeature AD-Domain-Services
    Install-WindowsFeature RSAT-ADDS
    Install-WindowsFeature RSAT-DNS-Server
~~~

Prerequisites를 모두 설치 후에, 새로운 호스트를 재부팅 합니다. 그러나 호스트를 다시 부팅하기 전에 AD forest 작성을 완료할 RUNONCE 스크립트를 만드세요. 이 작업의 경우, adds 모듈을 사용하여 RUNONCE 스크립트가 될 텍스트 파일을 작성하세요. 이 스크립트는 재부팅 후 로컬 관리자가 다음에 로그인할 때 실행되며 PowerShell 모듈 ADDSDeployment를 import 한 다음 forest의 이름을 지정하고 호스트를 도메인 컨트롤러 수준으로 올리는 Install-ADDSForest 명령을 실행합니다. 이 작업이 완료되면 호스트가 자동으로 재부팅 됩니다.

~~~
   # Create text block for the new script that will run once on reboot
    $addsmodule02 = @" 
    #ps1_sysnative
    Try {
    Start-Transcript -Path C:\DomainJoin\stage2.txt
    `$password = "P@ssw0rd123!!"
    `$FullDomainName = "cesa.corp"
    `$ShortDomainName = "CESA"
    `$encrypted = ConvertTo-SecureString `$password -AsPlainText -Force
    Import-Module ADDSDeployment
    Install-ADDSForest ``
    -CreateDnsDelegation:`$false ``
    -DatabasePath "C:\Windows\NTDS" ``
    -DomainMode "WinThreshold" ``
    -DomainName `$FullDomainName ``
    -DomainNetbiosName `$ShortDomainName ``
    -ForestMode "WinThreshold" ``
    -InstallDns:`$true ``
    -LogPath "C:\Windows\NTDS" ``
    -NoRebootOnCompletion:`$false ``
    -SysvolPath "C:\Windows\SYSVOL" ``
    -SafeModeAdministratorPassword `$encrypted ``
    -Force:`$true
    } Catch {
    Write-Host $_
    } Finally {
    Stop-Transcript
    }
    "@
    Add-Content -Path "C:\DomainJoin\ADDCmodule2.ps1" -Value $addsmodule02
~~~

그런 다음, 관리자(Administrator)가 다음 번 로그인 할 때 RUNONCE key를 추가하세요.

~~~
    # Adding the RunOnce job
    #
    $RunOnceKey = "HKLM:\Software\Microsoft\Windows\CurrentVersion\RunOnce"
    set-itemproperty $RunOnceKey "NextRun" ('C:\Windows\System32\WindowsPowerShell\v1.0\Powershell.exe -executionPolicy Unrestricted -File ' + "C:\DomainJoin\ADDCmodule2.ps1")
~~~


## Rebooting

호스트에 설치하는 모든 prerequisite가 완료되면, 호스트를 다시 부팅할 준비가 된 것입니다. 호스트를 재부팅하면 설치가 보다 명확해지고 new AD forest를 설치할 때 발생할 수 있는 오류의 수가 줄어듭니다.

~~~
   # Last step is to reboot the local host
    Restart-Computer -ComputerName "localhost" -Force 
~~~

다시 부팅한 후 다음 스크린샷과 같이 RUNONCE 스크립트를 사용하여 forest를 설치합니다. forest가 설치되면 호스트가 자동으로 다시 부팅됩니다.

![]({{site.baseurl}}/assets/img/windows_ad02.png)


## Final Check

Forest를 점검하여 모든 것이 올바른지 확인하세요. 도메인이 올바르게 설치되었는지 확인하는데 필요한 모든 기본 정보를 얻으려면 Get-ADForest 명령을 사용하세요.

![]({{site.baseurl}}/assets/img/windows_ad03.png)

다음번 로그인 시 Set-ADAccountPassword 명령으로 관리자 암호를 변경하여 도메인의 보안을 확인하세요.

![]({{site.baseurl}}/assets/img/windows_ad04.png)


## Final Script

다음은 전체 스크립트 입니다. 앞에서 설명한 모든 부분 외에도 스크립트에서 로깅을 설정하여 작업을 추적하고 문제를 해결할 수 있습니다.

~~~
  #ps1_sysnative
    Try {
    #
    # Start the logging in the C:\DomainJoin directory
    #
    Start-Transcript -Path "C:\DomainJoin\stage1.txt"
    # Global Variables
    $password="P@ssw0rd123!!"
    # Set the Administrator Password and activate the Domain Admin Account
    #
    net user Administrator $password /logonpasswordchg:no /active:yes
    # Install the Windows features necessary for Active Directory
    # Features
    # - .NET Core
    # - Active Directory Domain Services
    # - Remote Active Directory Services
    # - DNS Services
    #
    Install-WindowsFeature NET-Framework-Core
    Install-WindowsFeature AD-Domain-Services
    Install-WindowsFeature RSAT-ADDS
    Install-WindowsFeature RSAT-DNS-Server
    # Create text block for the new script that will be ran once on reboot
    #
    $addsmodule02 = @"
    #ps1_sysnative
    Try {
    Start-Transcript -Path C:\DomainJoin\stage2.txt
    `$password = "P@ssw0rd123!!"
    `$FullDomainName = "cesa.corp"
    `$ShortDomainName = "CESA"
    `$encrypted = ConvertTo-SecureString `$password -AsPlainText -Force
    Import-Module ADDSDeployment
    Install-ADDSForest ``
    -CreateDnsDelegation:`$false ``
    -DatabasePath "C:\Windows\NTDS" ``
    -DomainMode "WinThreshold" ``
    -DomainName `$FullDomainName ``
    -DomainNetbiosName `$ShortDomainName ``
    -ForestMode "WinThreshold" ``
    -InstallDns:`$true ``
    -LogPath "C:\Windows\NTDS" ``
    -NoRebootOnCompletion:`$false ``
    -SysvolPath "C:\Windows\SYSVOL" ``
    -SafeModeAdministratorPassword `$encrypted ``
    -Force:`$true
    } Catch {
    Write-Host $_
    } Finally {
    Stop-Transcript
    }
    "@
    Add-Content -Path "C:\DomainJoin\ADDCmodule2.ps1" -Value $addsmodule02
    # Adding the run once job
    #
    $RunOnceKey = "HKLM:\Software\Microsoft\Windows\CurrentVersion\RunOnce"
    set-itemproperty $RunOnceKey "NextRun" ('C:\Windows\System32\WindowsPowerShell\v1.0\Powershell.exe -executionPolicy Unrestricted -File ' + "C:\DomainJoin\ADDCmodule2.ps1")
    # End the logging
    #
    } Catch {
    Write-Host $_
    } Finally {
    Stop-Transcript
    }
    # Last step is to reboot the local host
    Restart-Computer -ComputerName "localhost" -Force  
~~~


## Summary

이 내용은 첫 번째 AD 도메인 컨트롤러를 설치하기 위한 간단한 스크립트 입니다. 프로세스를 완료하기 위해 두 번 재부팅해야하며 모든 Windows features를 설치하는데 약 20-25분이 소요됩니다. **["Creating Your Windows Active Directory Domain Servers in Oracle Cloud Infrastructure"](https://cloud.oracle.com/iaas/whitepapers/creating_active_directory_domain_services_in_oci.pdf)** 백서에서 Oracle Cloud Infrastructure에서 탄력있는 AD 환경을 구축하기 위한 추가 단계를 확인할 수 있습니다.


<br>
## 참고

이 포스트의 원문은 [Creating a Windows Active Directory Domain Controller in Oracle Cloud Infrastructure](https://blogs.oracle.com/cloud-infrastructure/creating-a-windows-active-directory-domain-controller-in-oracle-cloud-infrastructure) 입니다.
