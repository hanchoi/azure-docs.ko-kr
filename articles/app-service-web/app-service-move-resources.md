---
title: 웹앱 리소스를 다른 리소스 그룹으로 이동
description: 하나의 리소스 그룹에서 다른 리소스 그룹으로 웹앱 및 앱 서비스를 이동할 수 있는 시나리오를 설명합니다.
services: app-service
documentationcenter: ''
author: ZainRizvi
manager: wpickett
editor: ''

ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/04/2016
ms.author: zarizvi

---
# 지원되는 이동 구성
[ARM 이동 리소스 Api](../resource-group-move-resources.md)를 사용하여 Azure 웹앱 리소스를 이동할 수 있습니다.

Azure 웹앱은 현재 다음 이동 시나리오를 지원합니다.

* 리소스 그룹(웹 앱, 앱 서비스 계획 및 인증서)의 전체 내용을 다른 리소스 그룹으로 이동 
  * 참고: 대상 리소스 그룹은 이 시나리오에서 모든 Microsoft.Web 리소스를 포함할 수 없습니다.
* 여전히 자신의 현재 앱 서비스 계획(앱 서비스 계획은 이전 리소스 그룹에 유지)에서 호스트하는 동안 다른 리소스 그룹으로 개별 웹앱을 이동

<!---HONumber=AcomDC_0107_2016-->