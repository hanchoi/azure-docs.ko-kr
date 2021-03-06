---
title: 다른 서브넷으로 VM 또는 역할 인스턴스를 이동하는 방법
description: VM 및 역할 인스턴스를 다른 서브넷으로 이동하는 방법을 알아봅니다.
services: virtual-network
documentationcenter: na
author: jimdial
manager: carmonm
editor: tysonn

ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/22/2016
ms.author: jdial

---
# 다른 서브넷으로 VM 또는 역할 인스턴스를 이동하는 방법
PowerShell을 사용하여 동일한 가상 네트워크(VNet)에서 서브넷 간에 VM을 이동할 수 있습니다. PowerShell을 사용하지 않고 CSCFG를 편집하여 역할 인스턴스를 이동할 수 있습니다.

> [!NOTE]
> 이 문서에는 Azure 클래식 배포에만 상대적인 정보가 포함됩니다.
> 
> 

다른 서브넷으로 VM을 이동하는 이유 서브넷 마이그레이션은 기존 서브넷이 너무 작고 해당 서브넷에서 실행 중인 기존 VM으로 인해 확장할 수 없는 경우에 유용합니다. 이 경우 새로운, 더 큰 서브넷을 만들고 새 서브넷으로 VM을 마이그레이션한 다음 마이그레이션이 완료된 후 이전의 빈 서브넷을 삭제할 수 있습니다.

## 다른 서브넷으로 VM을 이동하는 방법
VM을 이동하려면 아래 예를 템플릿으로 사용하여 Set-AzureSubnet PowerShell cmdlet을 실행합니다. 아래 예제에서는 현재 서브넷에서 Subnet-2로 TestVM을 이동합니다. 사용 중인 환경에 맞게 예를 편집해야 합니다. 절차의 일부로 Update-AzureVM cmdlet을 실행할 때마다 VM이 업데이트 프로세스의 일환으로 다시 시작됩니다.

    Get-AzureVM –ServiceName TestVMCloud –Name TestVM `
    | Set-AzureSubnet –SubnetNames Subnet-2 `
    | Update-AzureVM

VM에 대한 고정 내부 개인 IP를 지정한 경우 먼저 해당 설정을 제거해야 새 서브넷으로 VM을 이동할 수 있습니다. 이 경우 다음을 사용합니다.

    Get-AzureVM -ServiceName TestVMCloud -Name TestVM `
    | Remove-AzureStaticVNetIP `
    | Update-AzureVM
    Get-AzureVM -ServiceName TestVMCloud -Name TestVM `
    | Set-AzureSubnet -SubnetNames Subnet-2 `
    | Update-AzureVM

## 다른 서브넷으로 역할 인스턴스를 이동하려면
역할 인스턴스를 이동하려면 CSCFG 파일을 편집합니다. 아래 예제에서는 가상 네트워크 *VNETName*의 "Role0"을 현재 서브넷에서 *Subnet-2*로 이동합니다. 역할 인스턴스가 이미 배포되었기 때문에 서브넷 이름 = Subnet-2를 변경하기만 합니다. 사용 중인 환경에 맞게 예를 편집해야 합니다.

    <NetworkConfiguration>
        <VirtualNetworkSite name="VNETName" />
        <AddressAssignments>
           <InstanceAddress roleName="Role0">
                <Subnets><Subnet name="Subnet-2" /></Subnets>
           </InstanceAddress>
        </AddressAssignments>
    </NetworkConfiguration> 

<!---HONumber=AcomDC_0810_2016-->