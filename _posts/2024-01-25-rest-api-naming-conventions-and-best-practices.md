---
layout: post
title:  "REST API Naming Conventions and Best Practices"
date:   2024-01-24 22:00:00 +0900
author: hsyoo
categories: REST API
published: true
---
<hr>

오늘은 REST API URI 네이밍 규칙과 모범 사례에 대해서 알아보겠습니다.

해당 포스팅은 [https://restfulapi.net/resource-naming/](https://restfulapi.net/resource-naming/){:target="_blank"}를 기계 번역해서 작성했습니다.


# 리소스란 무엇인가?

REST에서 주요 데이터 표현을 리소스라 합니다.


## 리소스 데이터는 싱글턴 또는 컬렉션으로 존재할 수 있다.

싱글턴은 단일 리소스를 나타내고, 컬렉션은 리소스의 그룹을 나타냅니다.
 
예를 들어 "customers"는 컬렉션 리소스이고 "customer"는 싱글톤 리소스입니다.

```
/customers			//is a collection resource

/customers/{id}		// is a singleton resource
```

## 리소스는 하위 컬렉션을 포함할 수 있다.

예를 들어, 특정 "accounts"의 하위 컬렉션 리소스 "customer"는 URN "/customers/{customerId}/accounts"을 사용하여 식별될 수 있습니다.

마찬가지로 "accounts" 하위 컬렉션 리소스 내의 "account" 싱글턴 리소스는 다음과 같이 식별할 수 있습니다.

```
/customers/{customerId}/accounts/{accountId}
```

## URI(Uniform Resource Identifier)

> URI는 리소스를 식별하는 데 사용되는 고유 식별자  

REST API는 URI를 사용하여 리소스를 처리합니다.
리소스 이름을 잘 지정하면 API가 직관적이고 사용하기 쉬우며 제대로 수행되지 않으면 동일한 API를 사용하고 이해하기 어려울 수 있습니다.


# 모범 사례(Best Practices)

## 리소스 이름은 명사로 구성되어야 한다.

RESTful URI는 동작(동사)을 참조하는 대신 사물(명사)인 리소스를 참조해야 합니다.  
왜냐하면 명사는 동사가 갖지 않는 속성을 갖기 때문입니다.
마찬가지로 리소스에도 속성이 있습니다.

예제

- 시스템 사용자
- 사용자 계정
- 네트워크 장치

```
/device-management/managed-devices 

/device-management/managed-devices/{device-id} 

/user-management/users

/user-management/users/{id}

```

보다 명확하게 하기 위해 리소스 원형을 세가지로 나눌 수 있습니다.

- 문서
- 컬렉션
- 저장소

그런 다음 리소스를 한 원형에 넣고 일관되게 그 명명 규칙을 사용하는 것이 좋습니다.

### 문서(Document)

문서 리소스는 개체 인스턴스 또는 데이터베이스 레코드와 유사한 **단일** 개념입니다.

REST에서는 리소스 컬렉션 내의 단일 리소스로 볼 수 있습니다. 문서의 상태 표현에는 일반적으로 값이 있는 필드와 다른 관련 리소스에 대한 링크가 모두 포함됩니다.

문서 리소스 원형을 표시하려면 " 단수형 " 이름을 사용하세요.

```
/device-management/managed-devices/{device-id}
/user-management/users/{id}
/user-management/users/admin
```


### 컬렉션(Collection)

컬렉션 리소스는 서버에서 관리하는 리소스 디렉터리입니다.

클라이언트는 컬렉션에 추가할 새 리소스를 제안할 수 있습니다. 그러나 새 리소스를 생성할지 여부를 선택하는 것은 컬렉션 리소스에 달려 있습니다.

컬렉션 리소스는 포함할 항목을 선택하고 포함된 각 리소스의 URI도 결정합니다.

컬렉션 리소스 원형을 나타내려면 **"복수형"** 이름을 사용하세요.

```
/device-management/managed-devices
/user-management/users
/user-management/users/{id}/accounts
```

### 저장소(Store)

스토어는 클라이언트가 관리 하는 자원 저장소입니다. 

클라이언트는 API를 이용하여 자원을 넣거나 가져올 수 있고 삭제 할 수 있습니다. 

복수를 사용하여 스토어를 표현합니다.

```
/song-management/users/{id}/playlists
```


## 일관성이 핵심이다

모호성을 최소화하고 가독성과 유지 관리성을 극대화하려면 일관된 리소스 명명 규칙과 URI 형식을 사용하세요. 

일관성을 유지하기 위해 아래 디자인 힌트를 구현할 수 있습니다.


### 슬래시(/)를 사용하여 계층 관계를 나타냅니다.

슬래시(/) 문자는 리소스 간의 계층 관계를 나타내기 위해 URI의 경로 부분에 사용됩니다.

```
/device-management
/device-management/managed-devices
/device-management/managed-devices/{id}
/device-management/managed-devices/{id}/scripts
/device-management/managed-devices/{id}/scripts/{id}
```

### URI에 후행 슬래시(/)를 사용하지 말 것

URI 경로의 마지막 문자인 슬래시(/)는 의미적 값을 추가하지 않으며 혼동을 줄 수 있습니다. URI에서 삭제하는 것이 좋습니다.

```
/device-management/managed-devices/
/device-management/managed-devices         /*This is much better version*/
```


### URI의 가독성을 높이려면 하이픈(-)을 사용

사람들이 URI를 쉽게 검색하고 해석할 수 있도록 하려면 하이픈(-) 문자를 사용하여 긴 경로 세그먼트에서 이름의 가독성을 높이세요.

```
/devicemanagement/manageddevices/
/device-management/managed-devices 	/*This is much better version*/
```


### 밑줄( _ )을 사용하지 말 것

하이픈 대신 밑줄을 사용하여 구분 기호로 사용할 수 있습니다. 

그러나 응용 프로그램의 글꼴에 따라 밑줄(_) 문자가 일부 브라우저나 화면에서 부분적으로 가려지거나 완전히 숨겨질 수 있습니다.

이러한 혼란을 피하려면 밑줄( _ ) 대신 하이픈(-)을 사용하세요.


```
/inventory-management/managed-entities/{id}/install-script-location  //More readable

/inventory-management/managedEntities/{id}/installScriptLocation  //Less readable
```

### URI에 소문자 사용

편리한 경우 URI 경로에서는 소문자를 일관되게 선호해야 합니다.

```
https://api.example.org/my-folder/my-doc       //1
HTTPs://API.EXAMPLE.ORG/my-folder/my-doc     //2
https://api.example.org/My-Folder/my-doc       //3
```

위의 예에서 1과 2는 동일하지만 3은 대문자로 My-Folder를 사용하므로 그렇지 않습니다.

### 파일 확장자를 사용하지 말 것

파일 확장자는 보기에 좋지 않으며 어떠한 이점도 추가하지 않습니다. 

이를 제거하면 URI 길이도 줄어듭니다. 보관할 이유가 없습니다.

API의 미디어 유형을 파일 확장자를 통해 강조하고 싶다면, Content-Type 헤더를 통해 통신된 미디어 유형에 의존하여 본문의 콘텐츠를 처리하는 방법을 결정해야 합니다.

```
/device-management/managed-devices.xml  /*Do not use it*/
/device-management/managed-devices 	/*This is correct URI*/
```

### URI에 CRUD 함수 이름을 사용하지 말 것

CRUD 기능을 나타내기 위해 URI를 사용해서는 안 됩니다. 

URI는 리소스를 식별하는 데에만 사용해야 하며 리소스에 대한 어떤 작업도 고유하게 사용해서는 안 됩니다.

어떤 CRUD 기능이 수행되는지 나타내기 위해 HTTP 요청 방법을 사용해야 합니다.

```
HTTP GET /device-management/managed-devices  			//Get all devices
HTTP POST /device-management/managed-devices  			//Create new Device

HTTP GET /device-management/managed-devices/{id}  		//Get device for given Id
HTTP PUT /device-management/managed-devices/{id}  		//Update device for given Id
HTTP DELETE /device-management/managed-devices/{id}  	//Delete device for given Id
```

### 쿼리 구성 요소를 사용하여 URI 수집 필터링

종종 특정 리소스 속성을 기반으로 정렬, 필터링 또는 제한되는 리소스 컬렉션이 필요한 요구 사항에 직면하게 됩니다.

이 요구 사항을 충족하려면 새 API를 생성하지 마세요. 대신 리소스 컬렉션 API에서 정렬, 필터링 및 페이지 매김 기능을 활성화하고 입력 매개 변수를 쿼리 매개 변수로 전달하세요.

```
/device-management/managed-devices
/device-management/managed-devices?region=USA
/device-management/managed-devices?region=USA&brand=XYZ
/device-management/managed-devices?region=USA&brand=XYZ&sort=installation-date
```

### URI에 동사를 사용하지 말 것

REST URI에 동사를 넣는 것은 올바르지 않습니다. 

REST는 명사를 사용하여 리소스를 나타내며 HTTP 메서드(GET, POST, PUT, DELETE 등)는 해당 리소스에 대한 작업을 수행하는 데 사용되어 효과적으로 동사 역할을 합니다.

URI에 동사를 사용하는 경우 JSON 또는 XML 요청/응답 형식을 갖는 RPC 스타일 메서드 호출을 생성할 가능성이 높습니다. 

REST라고 부르는 것은 올바르지 않습니다.

```
/device-management/managed-devices/{id}/scripts/{id}/execute    //It is RPC, and not REST
```

리소스 정의에 자연스럽게 적용되지 않는 일부 작업을 수행해야 하는 경우 명사/리소스로 간주될 수 있는 사용자 지정 URI를 만들고 이에 대한 작업을 수행할 수 있습니다.

예를 들어 호출하는 대신 /scripts/{id}/execute

현재 실행 중인 모든 스크립트에 대한 리소스를 생성하고 스크립트를 실행하려는 경우 여기에 스크립트를 제출할 수 있습니다.
```
/device-management/managed-devices/{id}/scripts/{id}/execute	//DON't DO THIS!
/device-management/managed-devices/{id}/scripts/{id}/status		//POST request with action=execute
```

# 결론

결론적으로, 자원은 주로 명사(객체)라고 할 수 있으며, 이 원칙에서 벗어나서는 안 됩니다. 언제든지 URI에 동사를 넣으면 RPC 호출이 생성됩니다.


## 번역하면서 느낀 점

해당 문서는 저자가 5년도 전에 작성한 문서지만, 마지막 업데이트가 작년 11월일 정도로 

REST API URI 네이밍 규칙에 대한 고민이 여전히 많이 있음을 알 수 있습니다.(댓글이 여전히 활발함)

다만 저자가 말한 것처럼, 일관성이 가장 중요하다고 생각합니다.

위 내용을 바탕으로 PMI 개발부에서도 REST API URI 네이밍 규칙을 정하고, 이를 지키도록 합시다. 


