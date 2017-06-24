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
먼저 기본 빌드 스크립트를 설정합니다. Spring을 사용하여 응용 프로그램을 빌드할 때 원하는 빌드 시스템을 사용할 수 있지만 Maven을 사용하여 작업해야하는 코드가 여기에 포함됩니다. Maven에 익숙하지 않은 경우 'Maven으로 Java 프로젝트 빌드'를 참조하십시오.

### 이클립스를 사용해서 프로젝트 만들기

1. New - Other... 메뉴 선택

![create_mavenproject_1](http://i.imgur.com/d6ARhTU.jpg)

2. Spring Starter Project 선택 후 Next 클릭

![create_spring_project](http://i.imgur.com/8r1Jt00.jpg)

3. Name, Artifact, Package를 아래와 같이 변경 후 Next 클릭

![create_spring_project_2](http://i.imgur.com/OLLsmbF.jpg)

4. Finish 클릭

![create_spring_project_3](http://i.imgur.com/MYGuNZv.jpg)

5. pom.xml을 다음과 같이 작성
~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>restwebservice</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>rest-webservice</name>
	<description>rest webservice project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
            <groupId>com.jayway.jsonpath</groupId>
            <artifactId>json-path</artifactId>
            <scope>test</scope>
        </dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
        <repository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </pluginRepository>
    </pluginRepositories>


</project>
~~~

Spring Boot Maven 플러그인은 많은 편리한 기능을 제공합니다

* 클래스 패스에있는 모든 jar 파일들을 수집하고 단독으로 실행 가능한 "über-jar"로 빌드합니다. 이것은 서비스를 실행하고 전송하는 것이 더 편리합니다.
* public static void main () 메서드를 검색하여 실행 가능 클래스로 플래그를 지정합니다.
* 스프링 부트 의존성에 맞게 버전 번호를 설정하는 빌트인 의존성 분석기를 제공합니다. 원하는 버전으로 오버라이드 할 수 있지만, 기본적으로 Boot에서 선택한 버전 세트가 사용됩니다.

## 리소스 표현 클래스 만들기
이제 프로젝트를 설정하고 시스템을 구축 했으므로 웹 서비스를 만들 수 있습니다.

서비스 상호 작용에 대해 생각함으로써 프로세스를 시작하십시오.

이 서비스는 '/greeting'에 대한 GET 요청을 처리하며 선택적으로 쿼리 문자열에 name 매개 변수를 사용합니다. GET 요청은 인사말을 나타내는 본문에 JSON이있는 200 OK 응답을 반환해야합니다. 다음과 같이 보일 것입니다.
~~~JSON
{
    "id": 1,
    "content": "Hello, World!"
}
~~~
id 필드는 인사말의 고유 식별자이며 content는 인사말의 텍스트 표현입니다.

인사 표현을 모델링하려면 리소스 표현 클래스를 작성하십시오. id 및 content 데이터에 대한 필드, 생성자 및 접근자를 가진 POJO를 제공하십시오.

1. src/main/java 하위에 hello 패키지 생성

![create_spring_project_4](http://i.imgur.com/tFD1Tab.jpg)

2. hello 패키지 하위에 Greeting.java 클래스파일 생성

![create_spring_project_5](http://i.imgur.com/M169dIK.jpg)

3. Greeting.java 파일에 다음과 같이 소스 작성

~~~java
package hello;

public class Greeting {

    private final long id;
    private final String content;

    public Greeting(long id, String content) {
        this.id = id;
        this.content = content;
    }

    public long getId() {
        return id;
    }

    public String getContent() {
        return content;
    }
}
~~~
이후의 단계에서 볼 수 있듯이 Spring은 Jackson JSON 라이브러리를 사용하여 Greeting 유형의 인스턴스를 JSON으로 자동 마샬링합니다.

그런 다음 이러한 인사말을 제공 할 리소스 컨트롤러를 만듭니다.

## 리소스 컨트롤러 만들기
RESTful 웹 서비스를 구현하기위한 Spring의 접근 방식에서 HTTP 요청은 컨트롤러에 의해 처리됩니다. 이러한 구성 요소는 @RestController 어노테이션으로 쉽게 식별할 수 있으며 GreetingController는 Greeting 클래스의 새 인스턴스를 리턴하여 '/greeting'에 대한 GET 요청을 처리합니다.

1. hello 패키지 하위에 GreetingController.java 클래스파일 생성

![create_spring_project_6](http://i.imgur.com/D4OMsXP.jpg)

2. GreetingController.java 파일에 다음과 같이 소스 작성
~~~java
package hello;

import java.util.concurrent.atomic.AtomicLong;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @RequestMapping("/greeting")
    public Greeting greeting(@RequestParam(value="name", defaultValue="World") String name) {
        return new Greeting(counter.incrementAndGet(),
                            String.format(template, name));
    }
}
~~~
이 컨트롤러는 현재는 간결하고 간단하지만 계속 진행됩니다. 단계별로 해봅시다.

@RequestMapping 어노테이션은 '/greeting'에 대한 HTTP 요청이 greeting() 메소드에 매핑되도록합니다.

@RequestMapping은 기본적으로 모든 HTTP 작업을 매핑하기 때문에 위의 예에서는 GET, PUT, POST 등을 지정하지 않습니다. @RequestMapping(method=GET)을 사용하여 매핑의 범위를 좁힙니다.

@RequestParam은 쿼리 문자열 매개 변수 이름의 값을 greeting() 메서드의 name 매개 변수에 바인딩합니다. 이 쿼리 문자열 매개 변수는 선택적으로 표시됩니다 (기본적으로 required=true). 요청에 문자열 매개 변수가 없으면 "World"라는 defaultValue가 사용됩니다.

메소드 본문의 구현은 카운터의 다음 값을 기반으로 id 및 content 속성을 사용하여 새 Greeting 객체를 만들고 반환하며 인사말 템플릿을 사용하여 지정된 이름의 형식을 지정합니다.

위의 전통적인 MVC 컨트롤러와 RESTful 웹 서비스 컨트롤러의 주요 차이점은 HTTP 응답 본문을 만드는 방법입니다. 이 RESTful 웹 서비스 컨트롤러는 단순히 뷰 기술을 사용하여 서버 쪽 인사말 데이터를 HTML로 렌더링하는 대신 Greeting 개체를 채우고 반환합니다. 객체 데이터는 JSON으로 HTTP 응답에 직접 기록됩니다.

이 코드는 모든 메소드가 뷰 대신 도메인 객체를 반환하는 컨트롤러로 클래스를 표시하는 Spring 4의 새로운 @RestController 어노테이션을 사용합니다. @RestController 어노테이션은 @Controller와 @ResponseBody 어노테이션의 역할을 한꺼번에 담당합니다.

Greeting 객체는 JSON으로 변환되어야합니다. Spring의 HTTP 메시지 변환기 지원 덕분에 이 변환을 수동으로 수행 할 필요가 없습니다. Jackson 2가 클래스 경로에 있기 때문에 Spring의 MappingJackson2HttpMessageConverter가 자동으로 선택되어 Greeting 인스턴스를 JSON으로 변환합니다.

## 어플리케이션을 실행 가능하게 만들기
이 서비스를 외부 응용 프로그램 서버에 배포하기위한 기존 WAR 파일로 패키지화 할 수 있지만 아래에 설명 된 간단한 방법은 독립 실행형 응용 프로그램을 만듭니다. 좋은 오래된 Java main() 메소드로 구동되는 실행 가능한 단일 JAR 파일로 모든 것을 패키지화합니다. 그동안 외부 인스턴스에 배포하는 대신 Tomcat 서블릿 컨테이너를 HTTP 런타임으로 포함시키기위한 Spring의 지원을 사용합니다.

1. hello 패키지 하위에 Application.java 클래스파일 생성

![create_application_1](http://i.imgur.com/4kglhQR.jpg)

2. Application.java 파일에 다음과 같이 소스코드 작성
~~~java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
~~~
@SpringBootApplication은 다음을 모두 더한 편리한 어노테이션입니다.

* @Configuration은 클래스를 응용 프로그램 컨텍스트에 대한 Bean 정의의 소스로 태그 지정합니다.
* @EnableAutoConfiguration은 Spring Boot에게 클래스 패스 설정, 다른 bean 및 다양한 속성 설정을 기반으로 bean 추가를 시작하도록 지시합니다.
* 일반적으로 @EnableWebMvc를 Spring MVC app에 추가 할 것이지만 Spring Boot는 classpath에서 spring-webmvc를 인식할 때 자동으로 추가합니다. 이것은 응용 프로그램에 웹 응용 프로그램으로 플래그를 지정하고 DispatcherServlet 설정과 같은 주요 동작을 활성화합니다.
* @ComponentScan은 Spring에게 hello 패키지의 다른 구성 요소, 구성 및 서비스를 찾아서 컨트롤러를 찾을 수있게합니다.

main() 메소드는 Spring Boot의 SpringApplication.run() 메소드를 사용하여 애플리케이션을 시작합니다. 한 줄의 XML도 없다는 것을 눈치 챘습니까? web.xml 파일도 없습니다. 이 웹 응용 프로그램은 100% 순수 자바이며 내부 구조 구성 또는 인프라 구성에 대해 다룰 필요가 없습니다.

## 실행 가능한 JAR 파일로 빌드하기
Gradle 또는 Maven을 사용하여 명령 줄에서 응용 프로그램을 실행할 수 있습니다. 또는 모든 필요한 종속성, 클래스 및 자원을 포함하는 단일 실행 가능 JAR 파일을 빌드하고 실행할 수 있습니다. 따라서 개발 수명주기, 다양한 환경에 걸쳐 응용 프로그램으로 서비스를 쉽게 버전관리 및 배포할 수 있습니다.

### 이클립스에서 Maven 빌드하기
1. 프로젝트 우클릭 - Run As - Maven build... 클릭

![maven_build_1](http://i.imgur.com/2M4FeE3.jpg)

2. Goals에 package 입력 후 Run 버튼 클릭

![maven_build_2](http://i.imgur.com/7Kftm8I.jpg)

3. 프로젝트의 target 폴더 하위에 gs-rest-webservice-0.1.0.jar 파일이 생성된 것을 확인합니다.

![maven_build_3](http://i.imgur.com/ZgxeTRh.jpg)

4. 생성된 jar 파일을 적당한 디렉토리에 복사한 후 디렉토리에서 shift 키를 누른 채로 마우스 우클릭을 하여 '여기서 명령 창 열기 메뉴' 클릭

![maven_build_4](http://i.imgur.com/5yN4WCO.jpg)

5. 'java -jar gs-rest-webservice-0.1.0.jar' 명령어로 실행

![maven_build_5](http://i.imgur.com/lJveNkV.jpg)

6. 서비스 실행 로그 확인

![maven_build_6](http://i.imgur.com/PdkGgu3.jpg)

## 서비스 테스트
이제 서비스가 시작되었으므로 브라우저에 http://localhost:8080/greeting 주소를 입력하십시오. 그러면 다음과 같은 메시지를 확인할 수 있습니다.
~~~
{"id":1,"content":"Hello, World!"}
~~~

![result](http://i.imgur.com/CPuKNzy.jpg)

http://localhost:8080/greeting?name=User와 함께 이름 쿼리 문자열 매개 변수를 제공하십시오. content 속성의 값이 "Hello, World!"에서 어떻게 변경되는지 주목하십시오. "Hello, User!"

![result2](http://i.imgur.com/ycDLQ1v.jpg)

이 변화는 GreetingController의 @RequestParam 배열이 예상대로 작동 함을 보여줍니다. name 매개 변수에는 기본값 "World"가 지정되었지만 항상 쿼리 문자열을 통해 명시적으로 재정의 될 수 있습니다.

또한 id 속성이 1에서 2로 변경된 것을 확인할 수 있습니다. 이는 여러 요청에서 동일한 GreetingController 인스턴스에 대해 작업하고 카운터 필드가 각 호출에서 예상대로 증가하고 있음을 증명합니다.

## 요약
축하합니다! 방금 스프링에 RESTful 웹 서비스를 개발했습니다.
