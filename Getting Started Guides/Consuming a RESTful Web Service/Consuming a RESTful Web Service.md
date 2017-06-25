# Consuming a RESTful Web Service

이 가이드는 RESTful 웹 서비스를 사용하는 응용 프로그램을 만드는 과정을 안내합니다.

## 당신이 만들게 될 것
Spring의 RestTemplate을 사용하여 http://gturnquist-quoters.cfapps.io/api/random에서 임의의 Spring Boot 인용을 검색하는 응용 프로그램을 빌드 할 것입니다.

## 당신이 준비해야 할 것
1. 약 15분의 시간
2. 텍스트 에디터 또는 IDE
3. JDK 1.8 혹은 그 이상
4. Gradle 2.3 이상 혹은 Maven 3.0 이상

## 이 가이드를 완료하는 방법
대부분의 Spring Getting Started 가이드와 마찬가지로, 처음부터 시작하여 각 단계를 완료하거나 이미 익숙한 기본 설정 단계를 건너 뛸 수 있습니다. 어쨌든, 당신은 working code로 끝납니다.

##### 처음부터 시작하려면 '이클립스를 사용해서 프로젝트 만들기'로 이동하십시오.

##### 기본 사항을 건너 뛰려면 다음을 수행하십시오.
이 가이드의 소스 저장소를 다운로드하고 압축을 해제하거나 Git을 사용하여 복제합니다. git clone https://github.com/spring-guides/gs-consuming-rest.git
gs-consuming-rest/initial 디렉토리로 이동하십시오.
'REST 리소스 가져오기'로 이동하십시오
작업이 끝나면 gs-consuming-rest/complete의 코드와 결과를 비교할 수 있습니다.

### 이클립스를 사용해서 프로젝트 만들기
1. New - Other... 메뉴 선택

![create_mavenproject_1](http://i.imgur.com/d6ARhTU.jpg)

2. Spring Starter Project 선택 후 Next 클릭

![create_spring_project](http://i.imgur.com/8r1Jt00.jpg)

3. Name, Artifact, Package를 아래와 같이 변경 후 Next 클릭

![1](http://i.imgur.com/WUH4JVs.jpg)

4. Finish 클릭

![2](http://i.imgur.com/5f2ICUE.jpg)

5. pom.xml을 다음과 같이 작성
~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>consumingrest</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>gs-consuming-rest</name>
	<description>consuming rest project for Spring Boot</description>

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
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
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


</project>
~~~
Spring Boot Maven 플러그인은 많은 편리한 기능을 제공합니다

* 클래스 패스에있는 모든 jar 파일들을 수집하고 단독으로 실행 가능한 "über-jar"로 빌드합니다. 이것은 서비스를 실행하고 전송하는 것이 더 편리합니다.
* public static void main () 메서드를 검색하여 실행 가능 클래스로 플래그를 지정합니다.
* 스프링 부트 의존성에 맞게 버전 번호를 설정하는 빌트인 의존성 분석기를 제공합니다. 원하는 버전으로 오버라이드 할 수 있지만, 기본적으로 Boot에서 선택한 버전 세트가 사용됩니다.

## REST 리소스를 가져오기
프로젝트 설정이 완료되면 RESTful 서비스를 사용하는 간단한 애플리케이션을 만들 수 있습니다.

RESTful 서비스가 http://gturnquist-quoters.cfapps.io/api/random에 떠있습니다. 이 서비스는 Spring Boot에 관한 따옴표를 무작위로 가져 와서 JSON 문서로 반환하는 서비스입니다.

웹 브라우저나 cURL을 통해 URL을 요청하면 다음과 같은 JSON 문서를 받게됩니다.
~~~json
{
   type: "success",
   value: {
      id: 10,
      quote: "Really loving Spring Boot, makes stand alone Spring apps easy."
   }
}
~~~
브라우저를 통해 또는 cURL을 통해 가져올 때 충분히 쉽지만 굉장히 유용하지는 않습니다.

REST 웹 서비스를 사용하는 보다 유용한 방법은 프로그래밍 방식입니다. 이 작업을 돕기 위해 Spring은 RestTemplate이라는 편리한 템플릿 클래스를 제공합니다. RestTemplate은 대부분의 RESTful 서비스와 단지 한 줄로 요청과 응답을 주고 받을 수 있습니다. 또한 이 데이터를 커스텀 도메인에 바인딩 할 수도 있습니다.

먼저 필요한 데이터를 담을 도메인 클래스를 만듭니다.
1. src/main/java 하위에 hello 패키지 생성

![3](http://i.imgur.com/u1skjQO.jpg)

2. hello 패키지 하위에 Quote.java 클래스파일 생성

![4](http://i.imgur.com/t2JmCXy.jpg)

3. Quote.java 파일에 다음과 같이 소스 작성
~~~java
package hello;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Quote {

    private String type;
    private Value value;

    public Quote() {
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public Value getValue() {
        return value;
    }

    public void setValue(Value value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Quote{" +
                "type='" + type + '\'' +
                ", value=" + value +
                '}';
    }
}
~~~
보시다시피 이 속성은 몇 가지 속성과 일치하는 getter 메서드가 있는 간단한 Java 클래스입니다. Jackson JSON 프로세싱 라이브러리의 @JsonIgnoreProperties 어노테이션이 달려있어 이 유형에 바인딩되지 않은 모든 속성은 무시된다는 것을 나타냅니다.

커스텀 데이터 타입에 데이터를 직접 바인딩하려면 API에서 반환된 JSON 문서의 키와 완전히 동일한 변수 이름을 지정해야합니다. JSON 문서의 변수 이름과 키가 일치하지 않는 경우 @JsonProperty 어노테이션을 사용하여 JSON 문서의 정확한 키를 지정해야합니다.

내부 quotation 자체를 삽입하려면 추가 클래스가 필요합니다.
1. hello 패키지 하위에 Value.java 클래스파일 생성

![5](http://i.imgur.com/pVaFijo.jpg)

2. Value.java 파일에 다음과 같이 소스 작성
~~~java
package hello;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Value {

    private Long id;
    private String quote;

    public Value() {
    }

    public Long getId() {
        return this.id;
    }

    public String getQuote() {
        return this.quote;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setQuote(String quote) {
        this.quote = quote;
    }

    @Override
    public String toString() {
        return "Value{" +
                "id=" + id +
                ", quote='" + quote + '\'' +
                '}';
    }
}
~~~
동일한 어노테이션을 사용하지만 다른 데이터 필드로 매핑됩니다.

## 애플리케이션을 실행 가능하게 만들기
이 서비스를 외부 응용 프로그램 서버에 배포하기위한 기존 WAR 파일로 패키지화 할 수 있지만 아래에 설명 된 간단한 방법은 독립 실행형 응용 프로그램을 만듭니다. 단순한 Java main() 메소드로 구동되는 실행 가능한 단일 JAR 파일로 모든 것을 패키지화합니다. 그동안 외부 인스턴스에 배포하는 대신 Tomcat 서블릿 컨테이너를 HTTP 런타임으로 포함시키는 Spring의 지원을 사용합니다.

이제 Spring Boot quotation 서비스에서 데이터를 가져 오기 위해 RestTemplate을 사용하는 Application 클래스를 작성할 수 있습니다.
1. hello 패키지 하위에 Application.java 클래스파일 생성

![6](http://i.imgur.com/fqHrVkH.jpg)

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
Jackson JSON 프로세싱 라이브러리가 클래스 경로에 있기 때문에 RestTemplate은 메시지 변환기를 통해 수신 JSON 데이터를 Quote 객체로 변환하는 데 RestTemplate을 사용합니다. 거기에서 Quote 객체의 내용이 콘솔에 기록됩니다.

여기서는 RestTemplate을 사용하여 HTTP GET 요청만 했습니다. 그러나 RestTemplate은 POST, PUT 및 DELETE와 같은 다른 HTTP 요청도 지원합니다.

## 스프링 부트로 애플리케이션 라이프 사이클 관리하기
지금까지는 응용 프로그램에서 Spring Boot를 사용하지 않았지만 그렇게 할 때 몇 가지 장점이 있습니다. 장점 중 하나는 Spring Boot가 RestTemplate에서 메시지 변환기를 관리하게 하여 사용자 정의를 선언적으로 쉽게 추가 할 수있게 하려는 것입니다. 이를 위해 우리는 메인 클래스에서 @SpringBootApplication을 사용하고 Spring 부트 애플리케이션과 같이 main 메소드를 시작하도록 변환한다. 마지막으로 RestTemplate을 CommandLineRunner 콜백으로 이동하여 시작시 스프링 부트에 의해 실행됩니다.
1. Application.java 파일에 다음과 같이 소스코드 작성
~~~java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class Application {

	private static final Logger log = LoggerFactory.getLogger(Application.class);

	public static void main(String args[]) {
		SpringApplication.run(Application.class);
	}

	@Bean
	public RestTemplate restTemplate(RestTemplateBuilder builder) {
		return builder.build();
	}

	@Bean
	public CommandLineRunner run(RestTemplate restTemplate) throws Exception {
		return args -> {
			Quote quote = restTemplate.getForObject(
					"http://gturnquist-quoters.cfapps.io/api/random", Quote.class);
			log.info(quote.toString());
		};
	}
}
~~~
RestTemplateBuilder는 Spring에 의해 주입되며 RestTemplate을 생성하기 위해 Spring을 사용한다면 Spring Boot에서 메시지 변환기와 요청 팩토리로 발생하는 모든 자동 설정의 이점을 누릴 수 있습니다. 우리는 RestTemplate을 @Bean으로 추출하여 쉽게 테스트 할 수 있습니다 (이 방법으로 mock 서비스 테스트를 쉽게 할 수 있습니다).

## 실행 가능한 JAR 파일로 빌드하기
Gradle 또는 Maven을 사용하여 명령 줄에서 응용 프로그램을 실행할 수 있습니다. 또는 모든 필요한 종속성, 클래스 및 자원을 포함하는 단일 실행 가능 JAR 파일을 빌드하고 실행할 수 있습니다. 따라서 개발 수명주기, 다양한 환경에 걸쳐 응용 프로그램으로 서비스를 쉽게 버전관리 및 배포할 수 있습니다.

### 이클립스에서 Maven 빌드하기
1. 프로젝트 우클릭 - Run As - Maven build... 클릭

![7](http://i.imgur.com/NISV9Te.jpg)

2. Goals에 package 입력 후 Run 버튼 클릭

![8](http://i.imgur.com/R9PmZla.jpg)

3. 프로젝트의 target 폴더 하위에 jar 파일이 생성된 것을 확인합니다.

![9](http://i.imgur.com/SEIZh6y.jpg)

4. 생성된 jar 파일을 적당한 디렉토리에 복사한 후 디렉토리에서 shift 키를 누른 채로 마우스 우클릭을 하여 '여기서 명령 창 열기 메뉴' 클릭

![10](http://i.imgur.com/kL6GrLg.jpg)

5. 'java -jar jar파일명' 명령어로 실행

![11](http://i.imgur.com/a69NsYY.jpg)

6. 서비스 실행 로그 확인

![12](http://i.imgur.com/SzGvXh6.jpg)

만약 'Could not extract response: no suitable HttpMessageConverter found for response type [class hello.Quote]' 에러를 보게 된다면 클라이언트가 JSON을 보내는 백엔드 서비스에 연결할 수 없는 환경에 있을 가능성이 있습니다. 어쩌면 당신은 기업 프록시 뒤에 있을까요? 표준 시스템 등록 정보 http.proxyHost 및 http.proxyPort를 사용자 환경에 적합한 값으로 설정하십시오.

## 요약
축하드립니다! 당신은 Spring을 사용하여 간단한 REST 클라이언트를 개발했습니다.
