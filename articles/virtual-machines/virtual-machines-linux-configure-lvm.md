---
title: Linux를 실행하는 가상 컴퓨터에 LVM 구성 | Microsoft Docs
description: Azure에서 Linux에 LVM을 구성하는 방법에 대해 알아봅니다.
services: virtual-machines-linux
documentationcenter: na
author: szarkos
manager: timlt
editor: tysonn
tag: azure-service-management,azure-resource-manager

ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 08/24/2016
ms.author: szark

---
# Azure에서 Linux VM에 LVM 구성
이 문서에서는 Azure 가상 컴퓨터의 LVM(논리 볼륨 관리자)을 구성하는 방법을 설명합니다. 가상 컴퓨터에 연결된 모든 디스크에 LVM을 구성할 수 있지만 기본적으로 대부분의 클라우드 이미지는 OS 디스크에 LVM을 구성하지 않습니다. 이는 OS 디스크가 동일한 배포와 형식의 다른 VM에 연결되어 있는 경우(예: 복구 시나리오 중), 중복 볼륨 그룹에 대한 문제를 방지하기 위함입니다. 따라서 데이터 디스크에만 LVM을 사용하는 것이 좋습니다.

## 선형 및 스트라이프 논리 볼륨 비교
LVM을 사용하여 단일 저장소 볼륨에 여러 실제 디스크를 결합할 수 있습니다. 기본적으로 LVM은 일반적으로 선형 논리 볼륨을 만듭니다. 즉, 실제 저장소가 함께 연결되어 있습니다. 이 경우 일반적으로 읽기/쓰기 작업은 단일 디스크로만 전송됩니다. 반면 읽기 및 쓰기가 볼륨 그룹에 포함된 여러 디스크에 분산되는 스트라이프 논리 볼륨을 만들 수도 있습니다(예: RAID0과 유사함). 성능상의 이유로 논리 볼륨을 스트라이프하여 읽기 및 쓰기가 연결된 모든 데이터 디스크를 사용하는 것이 좋습니다.

이 문서에는 여러 개의 데이터 디스크를 단일 볼륨 그룹으로 결합한 다음 스트라이프 논리 볼륨을 만드는 방법을 설명합니다. 아래 단계는 대부분의 배포로 작업하도록 다소 일반화되어 있습니다. 대부분의 경우 Azure의 LVM을 관리하기 위한 유틸리티 및 워크플로는 다른 환경과 근본적으로 다릅니다. 늘 그렇듯이 특정 배포로 LVM을 사용하는 설명서 및 모범 사례의 경우 Linux 공급업체에도 문의하시기 바랍니다.

## 데이터 디스크 연결
하나의 디스크가 LVM을 사용하는 경우 일반적으로 두 개 이상의 빈 데이터 디스크로 시작합니다. IO 요구 사항에 따라 표준 저장소에 저장된 디스크(디스크당 최대 500IO/ps) 또는 프리미엄 저장소에 저장된 디스크(디스크당 최대 5000IO/ps)를 연결할 수 있습니다. Linux 가상 컴퓨터에 데이터 디스크를 프로비전 및 연결하는 방법은 이 문서에서 자세히 다루지 않습니다. Azure에서 빈 데이터 디스크를 Linux 가상 컴퓨터에 연결하는 방법에 대한 자세한 내용은 Microsoft Azure 문서 [디스크 연결](virtual-machines-linux-add-disk.md)을 참조하세요.

## LVM 유틸리티 설치
* **Ubuntu**
  
        # sudo apt-get update
        # sudo apt-get install lvm2
* **RHEL, CentOS 및 Oracle Linux**
  
        # sudo yum install lvm2
* **SLES 12 및 openSUSE**
  
        # sudo zypper install lvm2
* **SLES 11**
  
        # sudo zypper install lvm2
  
    SLES11에서 /etc/sysconfig/lvm도 편집하고 `LVM_ACTIVATED_ON_DISCOVERED`을 "enable"로 설정해야 합니다.
  
        LVM_ACTIVATED_ON_DISCOVERED="enable" 

## LVM 구성
이 지침에서는 `/dev/sdc`, `/dev/sdd` 및 `/dev/sde`로 참조하는 세 개의 데이터 디스크가 연결되었다고 가정합니다. 이러한 세 디스크는 VM에서 항상 동일한 경로 이름이 아닙니다. '`sudo fdisk -l`' 또는 유사한 명령을 실행하여 사용 가능한 디스크를 나열할 수 있습니다.

1. 실제 볼륨을 준비합니다.
   
        # sudo pvcreate /dev/sd[cde]
          Physical volume "/dev/sdc" successfully created
          Physical volume "/dev/sdd" successfully created
          Physical volume "/dev/sde" successfully created
2. 볼륨 그룹을 만듭니다. 이 예제에서는 볼륨 그룹 "data-vg01"을 호출합니다.
   
       # sudo vgcreate data-vg01 /dev/sd[cde]
         Volume group "data-vg01" successfully created
3. 논리 볼륨을 만듭니다. 아래 명령으로 전체 볼륨 그룹에 걸쳐 "data-lv01"이라는 단일 논리 볼륨을 만들지만, 볼륨 그룹에 여러 논리 볼륨을 만들 수도 있습니다.
   
        # sudo lvcreate --extents 100%FREE --stripes 3 --name data-lv01 data-vg01
          Logical volume "data-lv01" created.
4. 논리 볼륨 포맷
   
        # sudo mkfs -t ext4 /dev/data-vg01/data-lv01
   
   > [!NOTE]
   > SLES11에서는 ext4가 아닌 "-t ext3"을 사용합니다. SLES11은 ext4 파일 시스템에 읽기 전용 액세스만 지원합니다.
   > 
   > 

## /etc/fstab에 새 파일 시스템 추가
**주의:** /etc/fstab 파일을 부적절하게 편집하면 부팅할 수 없는 시스템이 발생할 수 있습니다. 확실하지 않은 경우 배포 설명서에서 이 파일을 제대로 편집하는 방법에 대한 자세한 내용을 확인하세요. 또한 편집하기 전에 /etc/fstab 파일의 백업을 만드는 것이 좋습니다.

1. 새 파일 시스템용으로 원하는 탑재 지점을 만듭니다. 예를 들어 다음과 같습니다.
   
        # sudo mkdir /data
2. 논리 볼륨 경로 찾기
   
        # lvdisplay
        --- Logical volume ---
        LV Path                /dev/data-vg01/data-lv01
        ....
3. 텍스트 편집기에서 /etc/fstab을 열고 예를 들어 다음과 같이 새 파일 시스템에 항목을 추가합니다.
   
        /dev/data-vg01/data-lv01  /data  ext4  defaults  0  2
   
    그런 다음, /etc/fstab를 저장하고 닫습니다.
4. /etc/fstab 항목이 올바른지 테스트합니다.
   
        # sudo mount -a
   
    이 명령 결과 오류 메시지가 발생하는 경우 /etc/fstab 파일에서 구문을 확인하세요.
   
    그런 다음, `mount` 명령을 실행하여 파일 시스템이 탑재되었는지 확인합니다.
   
        # mount
        ......
        /dev/mapper/data--vg01-data--lv01 on /data type ext4 (rw)
5. (선택 사항) /etc/fstab의 Failsafe 부팅 매개 변수
   
    많은 배포에는 /etc/fstab 파일에 추가할 수 있는 `nobootwait` 또는 `nofail` 탑재 매개 변수가 포함되어 있습니다. 이 매개 변수는 특정 파일 시스템 탑재 시 오류를 허용하며 Linux 시스템이 제대로 RAID 파일 시스템을 탑재할 수 없는 경우에도 계속 부팅되도록 합니다. 이러한 매개 변수에 대한 자세한 내용은 배포 설명서를 참조하십시오.
   
    예제(Ubuntu):
   
        /dev/data-vg01/data-lv01  /data  ext4  defaults,nobootwait  0  2

<!---HONumber=AcomDC_0831_2016-->