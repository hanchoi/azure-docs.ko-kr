---
title: API 끝점 크기 조정 | Microsoft Docs
description: Azure 기계 학습에서 웹 서비스 끝점 크기 조정
services: machine-learning
documentationcenter: ''
author: hiteshmadan
manager: padou
editor: ''

ms.service: machine-learning
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 05/25/2016
ms.author: himad

---
# API 끝점 크기 조정
Azure 기계 학습의 웹 서비스 끝점에는 끝점이 사용되는 속도와 일치하도록 선택할 수 있는 제한 수준이 있습니다.

끝점에서 제한 양을 제어하려면 Azure 클래식 포털에서 슬라이더를 사용하여 최대 동시 호출 수를 20-200으로 설정합니다.

동기 API는 일반적으로 낮은 대기 시간을 원하는 경우에 사용됩니다. 여기서 대기 시간은 API가 하나의 요청을 완료하는 데 걸리는 시간을 의미하며 네트워크 지연을 고려하지 않습니다. 대기 시간이 50ms인 API가 있다고 가정합니다. 높음 제한 수준 및 최대 동시 호출 = 20으로 사용 가능한 용량을 완전히 사용하려면 이 API를 초당 20 * 1000 / 50 = 400회 호출해야 합니다. 더욱 확장하여 최대 동시 호출 수를 200으로 설정하면 대기 시간이 50ms일 경우 API를 초당 4000회 호출할 수 있습니다.

최대 동시 호출 수 200에서 지원하는 것보다 높은 부하로 API를 호출하려는 경우 동일한 웹 서비스에서 여러 끝점을 만들고 모든 끝점에 부하를 무작위로 분산해야 합니다.

해당하는 높은 속도로 API를 적중하지 않을 경우 매우 높은 동시성 개수를 사용하는 것은 바람직하지 않습니다. 높은 부하로 구성된 API에 상대적으로 낮은 부하를 배치할 경우 가끔 시간 초과 및/또는 대기 시간 급증이 나타날 수 있습니다.

제한 설정 조정은 동기 API(RRS)의 동작에만 영향을 줍니다. 동기 API에서 503 서비스를 사용할 수 없음 응답이 자주 표시되는 경우 이러한 설정을 조정해야 합니다.

관리 UI를 통해 사용자 지정 동시성 개수를 제공하여 20개의 기본 동시성 개수를 초과하는 끝점의 크기를 조정할 수 있습니다.

1. manage.windowsazure.com 열기
2. 기계 학습 탭으로 이동합니다.
3. 작업 영역을 클릭합니다.
4. 끝점이 있는 웹 서비스로 이동합니다. ![웹 서비스로 이동합니다.](./media/machine-learning-scaling-endpoints/figure-1.png)
5. 끝점을 클릭한 다음 구성 탭을 클릭합니다. ![끝점 구성으로 이동합니다.](./media/machine-learning-scaling-webservice/machlearn-2.png)
6. 슬라이더를 변경하여 동시성 수준을 높이고 저장을 클릭합니다.

<!---HONumber=AcomDC_0622_2016-->