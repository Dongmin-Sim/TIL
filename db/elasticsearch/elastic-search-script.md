---
title: 엘라스틱 서치 스크립트
description: 
author: codongmin
date: 2025-02-01T21:34:00
categories:
  - 자바
tags: 
image:
  path: /assets/img/thumbnail/.png
  lqip: 
  alt: 이미지 설명
---
loc

## 스크립트

Elasticsearch에서 스크립트(Script)는 **도큐먼트에 복잡한 연산, 계산**을 수행할 수 있게 도와주는 기능이다.
스크립트를 사용하는 이유는 정적인 쿼리로 불가능한 동적 쿼리가 필요할 때 주로 사용된다. 예를 들면 특정 조건을 만족하는 도큐먼트의 필드 값을 증감하거나, 


스크립트는 적용 방식에 따라 형태가 나뉜다. 

- Inline script : 요청,쿼리 시에 작성되는 스크립트
- Stored script: 미리 정의된 로직을 Elasticsearch 측에 저장해두고 사용하는 스크립트
	- 주로 복잡한 재사용성이 높은 로직의 경우.
- File script: 특정 디렉토리(config/scripts)에 파일 형태로 저장해서 사용하는 스크립트.
	- 잘 사용되지 않고, 현재는 보안상의 이슈로 권장되지 않는 방법.


|종류|특징|보안|용도|
|---|---|---|---|
|Inline|코드 즉시 삽입|제한적 허용|테스트, 단순 로직|
|Stored|사전 등록 후 ID로 호출|안전|재사용, 운영 환경|
|File|서버에 직접 저장|위험/비권장|구버전 환경|

### 스크립트 언어
스크립트를 사용하기 위한 전용 언어들이 존재. 

주로 사용되는 스크립트 언어는 `painless` 라는 언어이다. 성능과 보안상의 이유 ES에서 가장 권장하는 스크립팅 언어이다. 

이외에도 Expression, Mustache와 같은 스크립팅 언어들도 사용할 수 있다. 

Groovy, JavaScript, Python로도 스크립트 작성이 가능했으나 현재는 보안상 위험을 이유로 구버전만 지원하거나 권장되지 않는다. 

Painless is purpose-built for Elasticsearch, can be used for any purpose in the scripting APIs, and provides the most flexibility. The other languages are less flexible, but can be useful for specific purposes.

|Language|Sandboxed|Required plugin|Purpose|
|---|---|---|---|
|[`painless`](https://www.elastic.co/docs/explore-analyze/scripting/modules-scripting-painless)|![Yes](https://doc-icons.s3.us-east-2.amazonaws.com/icon-yes.png)|Built-in|Purpose-built for Elasticsearch|
|[`expression`](https://www.elastic.co/docs/explore-analyze/scripting/modules-scripting-expression)|![Yes](https://doc-icons.s3.us-east-2.amazonaws.com/icon-yes.png)|Built-in|Fast custom ranking and sorting|
|[`mustache`](https://www.elastic.co/docs/solutions/search/search-templates)|![Yes](https://doc-icons.s3.us-east-2.amazonaws.com/icon-yes.png)|Built-in|Templates|
|[`java`](https://www.elastic.co/docs/explore-analyze/scripting/modules-scripting-engine)|![No](https://doc-icons.s3.us-east-2.amazonaws.com/icon-no.png)|You write it!|Expert API|

사용 위치: `script`, `scripted_metric`, `script_field`, `update by query`, etc.

주로 Document를 업데이트하거나, 사용자 정의함수로 scoring하거나(검색시 우선순위), 커스텀한 기준으로 집계를 할때 사용됨. 

```json
POST /index/_update/1

"script": {
  "source": "ctx._source.price *= 1.1"
}

```


```json
POST /index/_search

# scoring
POST /my_index/_search
{
  "query": {
    "function_score": {
      "query": {
        "match_all": {}
      },
      "script_score": {
        "script": {
          "source": "doc['price'].value * _score"
        }
      }
    }
  }
}

```

`scoring`은 검색 결과 문서별로 **관련성 점수(Relevance Score)** 를 계산하는 과정입니다.
위 예시는 `function_score` 쿼리 내에서 스크립트를 이용해 문서 필드 `price`와 기본 점수 `_score`를 곱해서 최종 점수를 조정하는 방식입니다.
즉, **검색 점수에 가격 가중치를 곱해 랭킹을 조절하는 역할**을 합니다.

```json

# aggreation
POST /my_index/_search
{
  "size": 0,
  "aggs": {
    "custom_avg": {
      "avg": {
        "script": {
          "source": "doc['price'].value * params.vat_rate",
          "params": {
            "vat_rate": 1.2
          }
        }
      }
    }
  }
}

```

이 쿼리는 Elasticsearch에서 **스크립트를 이용한 사용자 정의 평균(Average) 집계**를 수행합니다.
`price` 필드에 `VAT(부가세)` 비율 1.2를 곱한 값의 **평균을 계산**합니다.

### 주의사항

Dynamic Scripting
**실행 시점에 문자열 형태의 스크립트를 전달해서 바로 실행하는 방식**을 말합니다.
즉, 스크립트를 미리 저장하지 않고 쿼리 요청 때마다 새로 작성하여 동적으로 실행하는 것을 의미합니다.

스크립트를 사용할 수 있는 권한을 제한되게 주어야만 함. 왜냐하면 보통 ES에서 Dynamic 스크립팅을 사용하게되면 부하가 많이감. 필요하지 않다면 운영 상황에서는 Dynamic scripIting을 사용하지 않는게 좋음

필요하다면 스크립트를 store 해서 아이디를 중심으로 스크립트를 사용하는 것이 좋음. 

- 임의 코드 실행 가능 → 보안 위협
- 과도한 자원 소모 가능성 → 시스템 부하 유발

잘 사용은 안하고, 업데이트하는 부분만 inline으로 사용

