---
title: 데이터 원본 및 검색 인덱스의 차이를 극복하는 Azure 검색 인덱서 필드 매핑
description: 필드 이름 및 데이터 표현의 차이를 처리하도록 Azure 검색 인덱서 필드 매핑 구성
services: search
documentationcenter: ''
author: chaosrealm
manager: pablocas
editor: ''

ms.service: search
ms.devlang: rest-api
ms.workload: search
ms.topic: article
ms.tgt_pltfrm: na
ms.date: 04/30/2016
ms.author: eugenesh

---
# 데이터 원본 및 검색 인덱스의 차이를 극복하는 Azure 검색 인덱서 필드 매핑
Azure 검색 인덱서를 사용할 때 입력 데이터가 대상 인덱스 스키마와 정확히 일치하지 않는 경우에 직면할 수 있습니다. 이러한 경우 **필드 매핑**을 사용하여 데이터를 원하는 모양으로 변환할 수 있습니다.

필드 매핑이 유용한 일부 상황:

* 데이터 원본에 `_id` 필드가 있지만 Azure 검색이 밑줄로 시작하는 필드 이름을 허용하지 않습니다. 필드 매핑을 사용하면 필드 "이름 바꾸기"가 가능합니다. 
* 예를 들어 이러한 필드에 다른 분석기를 적용하려고 하기 때문에 동일한 데이터 원본 데이터를 사용하여 인덱스에 여러 필드를 입력하려고 합니다. 필드 매핑을 사용하여 데이터 원본 필드를 "분기"할 수 있습니다.
* 데이터를 Base64 인코딩 또는 디코딩해야 합니다. 필드 매핑은 Base64 인코딩 및 디코딩에 대한 함수를 포함한 여러 **매핑 함수**를 지원합니다.   

> [!IMPORTANT]
> 필드 매핑 기능은 현재 미리 보기 상태입니다. **2015-02-28-Preview** 버전을 사용하여 REST API로만 제공됩니다. 미리 보기 API는 테스트 및 평가 용도로 제공되며 프로덕션 환경에는 사용되지 않는다는 점을 유념하세요.
> 
> 

## 필드 매핑 설정
[인덱서 만들기](search-api-indexers-2015-02-28-preview.md#create-indexer) API를 사용하여 새 인덱서를 만들 때 필드 매핑을 추가할 수 있습니다. [인덱서 업데이트](search-api-indexers-2015-02-28-preview.md#update-indexer) API를 사용하여 인덱싱 인덱서에서 필드 매핑을 관리할 수 있습니다.

필드 매핑은 3개의 부분으로 구성됩니다.

1. `sourceFieldName`은(는) 데이터 원본의 필드를 나타냅니다. 이 속성은 필수입니다. 
2. 선택 사항 `targetFieldName`은(는) 검색 인덱스의 필드를 나타냅니다. 생략된 경우 데이터 원본과 동일한 이름이 사용됩니다.
3. 선택 사항 `mappingFunction`은(는) 미리 정의된 여러 함수 중 하나를 사용하여 데이터를 변환할 수 있습니다. 함수의 전체 목록은 [아래](#mappingFunctions)입니다.

필드 매핑은 인덱서 정의의 `fieldMappings` 배열에 추가되었습니다.

예를 들어 필드 이름의 차이를 수용하는 방법은 다음과 같습니다.

```JSON

PUT https://[service name].search.windows.net/indexers/myindexer?api-version=[api-version]
Content-Type: application/json
api-key: [admin key]
{
    "dataSourceName" : "mydatasource",
    "targetIndexName" : "myindex",
    "fieldMappings" : [ { "sourceFieldName" : "_id", "targetFieldName" : "id" } ] 
} 
```

인덱서는 여러 필드 매핑을 포함할 수 있습니다. 예를 들어 필드를 "분기"할 수 있는 방법은 다음과 같습니다.

```JSON

"fieldMappings" : [ 
    { "sourceFieldName" : "text", "targetFieldName" : "textStandardEnglishAnalyzer" },
    { "sourceFieldName" : "text", "targetFieldName" : "textSoundexAnalyzer" }, 
] 
```

> [!NOTE]
> Azure 검색은 대/소문자 구분 비교를 사용하여 필드 매핑의 필드 및 함수 이름을 확인합니다. 이는 편리(모든 대/소문자를 올바르게 할 필요가 없음)하지만 데이터 원본 또는 인덱스가 대/소문자만으로 다른 필드를 가질 수 없음을 의미합니다.
> 
> 

<a name="mappingFunctions"></a>

## 필드 매핑 함수
이러한 함수는 현재 지원됩니다.

* [base64Encode](#base64EncodeFunction)
* [base64Decode](#base64DecodeFunction)
* [extractTokenAtPosition](#extractTokenAtPositionFunction)
* [jsonArrayToStringCollection](#jsonArrayToStringCollectionFunction)

<a name="base64EncodeFunction"></a>

### base64Encode
입력된 문자열의 *URL 안전* Base64 인코딩을 수행합니다. 입력이 UTF-8 인코딩되었다고 가정합니다.

#### 샘플 사용 사례
고객은 예를 들어 조회 API를 사용하여 문서를 처리할 수 있어야 하기 때문에 URL 안전 문자만 Azure 검색 문서 키에 나타날 수 있습니다. 데이터가 URL로부터 안전하지 않은 문자를 포함하고 검색 인덱스의 키 필드를 채우는 데 사용하려는 경우 이 함수를 사용합니다.

#### 예제
```JSON

"fieldMappings" : [ 
  { 
    "sourceFieldName" : "Path", 
    "targetFieldName" : "UrlSafePath",
    "mappingFunction" : { "name" : "base64Encode" } 
  }] 
```

<a name="base64DecodeFunction"></a>

### base64Decode
입력 문자열의 Base64 디코딩을 수행합니다. 입력은 *URL 안전* Base64 인코딩된 문자열로 간주됩니다.

#### 샘플 사용 사례
Blob 사용자 지정 메타데이터 값은 ASCII 인코딩되어야 합니다. Base64 인코딩을 사용하여 Blob 사용자 지정 메타데이터에서 임의의 유니코드 문자열을 나타낼 수 있습니다. 그러나 의미 있는 검색을 만들려면 검색 인덱스를 채울 때 이 함수를 사용하여 인코딩된 데이터를 “일반" 문자열로 변환시킬 수 있습니다.

#### 예제
```JSON

"fieldMappings" : [ 
  { 
    "sourceFieldName" : "Base64EncodedMetadata", 
    "targetFieldName" : "SearchableMetadata",
    "mappingFunction" : { "name" : "base64Decode" } 
  }] 
```

<a name="extractTokenAtPositionFunction"></a>

### extractTokenAtPosition
지정된 구분 기호를 사용하여 문자열 필드를 분할하고 결과 분할의 지정된 위치에서 토큰을 선택합니다.

예를 들어 입력이 `Jane Doe`이고 `delimiter`은(는) `" "`(공백)이고 `position`이(가) 0이면 결과는 `Jane`이고 `position`이(가) 1이면 결과는 `Doe`입니다. 위치가 존재하지 않는 토큰을 참조하는 경우 오류가 반환됩니다.

#### 샘플 사용 사례
데이터 원본이 `PersonName` 필드를 포함하고 두 개의 개별 `FirstName` 및 `LastName` 필드로 인덱싱하려고 합니다. 구분 기호로 공백 문자를 사용하여 입력을 분할하는 데 이 함수를 사용할 수 있습니다.

#### 매개 변수
* `delimiter`: 입력 문자열을 분할할 때 구분 기호로 사용할 문자열입니다.
* `position`: 입력 문자열을 분할한 후 선택할 정수 0부터 시작하는 토큰의 위치입니다.    

#### 예제
```JSON 

"fieldMappings" : [ 
  { 
    "sourceFieldName" : "PersonName", 
    "targetFieldName" : "FirstName",
    "mappingFunction" : { "name" : "extractTokenAtPosition", "parameters" : { "delimiter" : " ", "position" : 0 } } 
  }, 
  { 
    "sourceFieldName" : "PersonName", 
    "targetFieldName" : "LastName",
    "mappingFunction" : { "name" : "extractTokenAtPosition", "parameters" : { "delimiter" : " ", "position" : 1 } } 
  }] 
```

<a name="jsonArrayToStringCollectionFunction"></a>

### jsonArrayToStringCollection
JSON 문자열 배열 형식으로 생성된 문자열을 인덱스의 `Collection(Edm.String)` 필드를 채우는 데 사용할 수 있는 문자열 배열로 변환합니다.

예를 들어 입력 문자열이 `["red", "white", "blue"]`이면 `Collection(Edm.String)` 형식의 대상 필드는 세 개의 값 `red`, `white` 및 `blue`(으)로 채워집니다. JSON 문자열 배열로 구문 분석할 수 없는 입력 값의 경우 오류가 반환됩니다.

#### 샘플 사용 사례
Azure SQL 데이터베이스는 Azure 검색의 `Collection(Edm.String)` 필드에 자연스럽게 매핑하는 기본 제공 데이터 형식이 없습니다. 문자열 컬렉션 필드를 채우려면 JSON 문자열 배열로 원본 데이터의 서식을 지정하고 이 함수를 사용합니다.

#### 예제
```JSON

"fieldMappings" : [ 
  { "sourceFieldName" : "tags", "mappingFunction" : { "name" : "jsonArrayToStringCollection" } }
] 
```

## Azure 검색 개선 지원
기능 요청 또는 개선에 대한 아이디어가 있는 경우 [UserVoice 사이트](https://feedback.azure.com/forums/263029-azure-search/)를 통해 연락해 주세요.

<!---HONumber=AcomDC_0504_2016-->