# Building a RESTful Web Service

이 가이드는 Spring과 함께 "hello world" RESTful 웹 서비스를 만드는 과정을 안내합니다.

## 당신이 만들게 될 것
1. HTTP GET 요청을 받아들이는 서비스
~~~
http://localhost:8080/greeting
~~~
2. greeting의 JSON 포맷으로 응답
3. 쿼리 문자열의 선택적 name 매개 변수를 사용하여 greeting을 사용자 정의
~~~
http://localhost:8080/greeting?name=User
~~~
4. name 매개 변수 값은 기본값 "World"를 대체하며 응답에 반영
~~~
{"id":1,"content":"Hello, User!"}
~~~

## 당신이 준비해야 할 것
1. 약 15분의 시간
2. 텍스트 에디터 또는 IDE
3. JDK 1.8 혹은 그 이상
4. Gradle 2.3 이상 혹은 Maven 3.0 이상

## 이 가이드를 완료하는 방법
대부분의 Spring Getting Started 가이드와 마찬가지로, 처음부터 시작하여 각 단계를 완료하거나 이미 익숙한 기본 설정 단계를 건너 뛸 수 있습니다. 어쨌든, 당신은 working code로 끝납니다.

##### 처음부터 시작하려면 'Maven으로 빌드'로 이동하십시오.

##### 기본 사항을 건너 뛰려면 다음을 수행하십시오.
이 가이드의 소스 저장소를 다운로드하고 압축을 해제하거나 Git을 사용하여 복제합니다. git clone https://github.com/spring-guides/gs-rest-service.git
gs-rest-service / initial 디렉토리로 이동하십시오.
'리소스 표현 클래스 만들기'로 이동하십시오
작업이 끝나면 gs-rest-service / complete의 코드와 결과를 비교할 수 있습니다.

## Maven으로 빌드
먼저 기본 빌드 스크립트를 설정합니다. Spring을 사용하여 응용 프로그램을 빌드할 때 원하는 빌드 시스템을 사용할 수 있지만 Maven을 사용하여 작업해야하는 코드가 여기에 포함됩니다. Maven에 익숙하지 않은 경우 Maven으로 Java 프로젝트 빌드를 참조하십시오.
