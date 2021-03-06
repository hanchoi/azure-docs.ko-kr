> [!div class="op_single_selector"]
> * [Node.JS](../articles/iot-hub/iot-hub-node-node-twin-getstarted.md)
> * [C#](../articles/iot-hub/iot-hub-csharp-node-twin-getstarted.md)
> 
> 

## <a name="introduction"></a>소개
장치 쌍은 장치의 상태 정보(메타데이터, 상태 및 조건)를 저장하는 JSON 문서입니다. IoT Hub는 IoT Hub에 연결하는 각 장치에 대해 하나의 장치 쌍을 유지합니다.

장치 쌍의 용도:

* 백 엔드의 장치 메타데이터를 저장합니다.
* 장치 앱의 사용 가능한 기능 및 상태(예: 사용된 연결 방법)와 같은 현재 상태 정보를 보고합니다.
* 장치 앱과 백 엔드 간에 장기 실행 워크플로 (예: 펌웨어 및 구성 업데이트)의 상태를 동기화합니다.
* 장치 메타데이터, 구성 또는 상태를 쿼리합니다.

> [!NOTE]
> 장치 쌍은 장치 구성 및 상태를 동기화하고 쿼리하기 위해 설계되었습니다. 타임 스탬프가 지정된 이벤트(예: 시간 기반 데이터 센서의 원격 분석 스트림)의 시퀀스에 대해서는 [장치-클라우드 메시지][lnk-d2c] 그리고 사용자 제어 앱에서 팬을 켜는 것과 같은 장치 대화형 제어에 대해서는 [직접 메서드][lnk-methods]를 사용합니다.
> 
> 

장치 쌍은 IoT hub에 저장되고 다음을 포함합니다.

* *태그*, 백 엔드만 액세스할 수 있는 장치 메타데이터.
* *desired 속성*, 백 엔드에서 수정할 수 있고 장치 앱에서 관찰할 수 있는 JSON 개체.
* *reported 속성*, 장치 앱에서 수정할 수 있고 백 엔드에서 읽을 수 있는 JSON 개체. 태그 및 속성은 배열을 포함할 수 없지만, 개체는 중첩될 수 있습니다.

![][img-twin]

또한 앱 백 엔드는 위의 모든 데이터를 기반으로 하는 장치 쌍을 쿼리할 수 있습니다.
장치 쌍에 대한 자세한 내용은 [쌍 장치 이해][lnk-twins]를 그리고 쿼리에 대한 참조는 [IoT Hub 쿼리 언어][lnk-query]를 참조하세요.

> [!NOTE]
> 현재 장치 쌍은 MQTT 프로토콜을 사용하여 IoT Hub에 연결하는 장치에서만 액세스할 수 있습니다. 기존 장치 앱이 MQTT를 사용하도록 변환하는 방법에 관한 설명은 [MQTT 지원][lnk-devguide-mqtt] 문서를 참조하세요.
> 
> 

이 자습서에서는 다음을 수행하는 방법에 대해 설명합니다.

* *태그*를 장치 쌍에 추가하는 백 엔드 앱과 연결 채널을 *reported 속성*으로 장치 쌍에 보고하는 시뮬레이션된 장치를 만듭니다.
* 이전에 만든 태그 및 속성에 필터를 사용하여 백 엔드 앱에서 장치를 쿼리합니다.

<!-- images -->
[img-twin]: media/iot-hub-selector-twin-get-started/twin.png

<!-- links -->
[lnk-query]: ../articles/iot-hub/iot-hub-devguide-query-language.md
[lnk-twins]: ../articles/iot-hub/iot-hub-devguide-device-twins.md
[lnk-d2c]: ../articles/iot-hub/iot-hub-devguide-messaging.md#device-to-cloud-messages
[lnk-methods]: ../articles/iot-hub/iot-hub-devguide-direct-methods.md
[lnk-devguide-mqtt]: ../articles/iot-hub/iot-hub-mqtt-support.md

<!--HONumber=Nov16_HO3-->


