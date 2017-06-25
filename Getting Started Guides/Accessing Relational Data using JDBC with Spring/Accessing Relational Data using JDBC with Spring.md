# Accessing Relational Data using JDBC with Spring
이 가이드는 Spring을 사용하여 관계형 데이터에 액세스하는 과정을 안내합니다.

## 당신이 만들게 될 것
Spring JdbcTemplate을 사용하여 관계형 데이터베이스에 저장된 데이터에 액세스하는 응용 프로그램을 빌드합니다.

## 당신이 준비해야 할 것
1. 약 15분의 시간
2. 텍스트 에디터 또는 IDE
3. JDK 1.8 혹은 그 이상
4. Gradle 2.3 이상 혹은 Maven 3.0 이상

## 이 가이드를 완료하는 방법
대부분의 Spring Getting Started 가이드와 마찬가지로, 처음부터 시작하여 각 단계를 완료하거나 이미 익숙한 기본 설정 단계를 건너 뛸 수 있습니다. 어쨌든, 당신은 working code로 끝납니다.

##### 처음부터 시작하려면 '이클립스를 사용해서 프로젝트 만들기'로 이동하십시오.

##### 기본 사항을 건너 뛰려면 다음을 수행하십시오.
이 가이드의 소스 저장소를 다운로드하고 압축을 해제하거나 Git을 사용하여 복제합니다. git clone https://github.com/spring-guides/gs-relational-data-access.git
gs-relational-data-access/initial 디렉토리로 이동하십시오.
'Customer 객체 만들기'로 이동하십시오
작업이 끝나면 gs-relational-data-access/complete의 코드와 결과를 비교할 수 있습니다.

### 이클립스를 사용해서 프로젝트 만들기
1. New - Other... 메뉴 선택

![create_mavenproject_1](http://i.imgur.com/d6ARhTU.jpg)

2. Spring Starter Project 선택 후 Next 클릭

![create_spring_project](http://i.imgur.com/8r1Jt00.jpg)

3. Name, Artifact, Package를 아래와 같이 변경 후 Next 클릭

![1](http://i.imgur.com/qOK0JYC.jpg)

4. Finish 클릭

![2](http://i.imgur.com/aiYXtXK.jpg)

5. pom.xml을 다음과 같이 작성
~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>dataaccess</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>gs-relational-data-access</name>
	<description>relational data access project for Spring Boot</description>

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
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
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

## Customer 객체 만들기
아래에서 다루는 간단한 데이터 액세스 로직은 성과 이름을 관리합니다. 응용 프로그램 수준에서이 데이터를 나타내기 위해 Customer 클래스를 만듭니다.
1. src/main/java 하위에 hello 패키지 생성

![3](http://i.imgur.com/Moj2t8P.jpg)

2. hello 패키지 하위에 Customer.java 클래스파일 생성

![4](http://i.imgur.com/pverZjJ.jpg)

3. Customer.java 파일에 다음과 같이 소스 작성
~~~java
package hello;

public class Customer {
    private long id;
    private String firstName, lastName;

    public Customer(long id, String firstName, String lastName) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return String.format(
                "Customer[id=%d, firstName='%s', lastName='%s']",
                id, firstName, lastName);
    }

    // getters & setters omitted for brevity
}
~~~

## 데이터 저장 및 검색
Spring은 JdbcTemplate이라는 템플릿 클래스를 제공하여 SQL 관계형 데이터베이스와 JDBC로 작업하기가 쉽습니다. 대부분의 JDBC 코드는 리소스 확보, 연결 관리, 예외 처리 및 이루고자 하는 코드와는 전혀 관련이 없는 일반적인 오류 검사에 빠져 있습니다. JdbcTemplate이 모든 것을 처리합니다. 당신이해야 할 일은 당면 과제에 집중하는 것뿐입니다.
1. hello 패키지 하위에 Application.java 클래스파일 생성

![5](http://i.imgur.com/RdrOUQc.jpg)

2. Application.java 파일에 다음과 같이 소스코드 작성
~~~java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.jdbc.core.JdbcTemplate;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

@SpringBootApplication
public class Application implements CommandLineRunner {

    private static final Logger log = LoggerFactory.getLogger(Application.class);

    public static void main(String args[]) {
        SpringApplication.run(Application.class, args);
    }

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Override
    public void run(String... strings) throws Exception {

        log.info("Creating tables");

        jdbcTemplate.execute("DROP TABLE customers IF EXISTS");
        jdbcTemplate.execute("CREATE TABLE customers(" +
                "id SERIAL, first_name VARCHAR(255), last_name VARCHAR(255))");

        // Split up the array of whole names into an array of first/last names
        List<Object[]> splitUpNames = Arrays.asList("John Woo", "Jeff Dean", "Josh Bloch", "Josh Long").stream()
                .map(name -> name.split(" "))
                .collect(Collectors.toList());

        // Use a Java 8 stream to print out each tuple of the list
        splitUpNames.forEach(name -> log.info(String.format("Inserting customer record for %s %s", name[0], name[1])));

        // Uses JdbcTemplate's batchUpdate operation to bulk load data
        jdbcTemplate.batchUpdate("INSERT INTO customers(first_name, last_name) VALUES (?,?)", splitUpNames);

        log.info("Querying for customer records where first_name = 'Josh':");
        jdbcTemplate.query(
                "SELECT id, first_name, last_name FROM customers WHERE first_name = ?", new Object[] { "Josh" },
                (rs, rowNum) -> new Customer(rs.getLong("id"), rs.getString("first_name"), rs.getString("last_name"))
        ).forEach(customer -> log.info(customer.toString()));
    }
}
~~~
@SpringBootApplication은 다음을 모두 더한 편리한 어노테이션입니다.

* @Configuration은 클래스를 응용 프로그램 컨텍스트에 대한 Bean 정의의 소스로 태그 지정합니다.
* @EnableAutoConfiguration은 Spring Boot에게 클래스 패스 설정, 다른 bean 및 다양한 속성 설정을 기반으로 bean 추가를 시작하도록 지시합니다.
* 일반적으로 @EnableWebMvc를 Spring MVC app에 추가 할 것이지만 Spring Boot는 classpath에서 spring-webmvc를 인식할 때 자동으로 추가합니다. 이것은 응용 프로그램에 웹 응용 프로그램으로 플래그를 지정하고 DispatcherServlet 설정과 같은 주요 동작을 활성화합니다.
* @ComponentScan은 Spring에게 hello 패키지의 다른 구성 요소, 구성 및 서비스를 찾아서 컨트롤러를 찾을 수있게합니다.

main() 메소드는 Spring Boot의 SpringApplication.run() 메소드를 사용하여 애플리케이션을 시작합니다. 한 줄의 XML도 없다는 것을 눈치 챘습니까? web.xml 파일도 없습니다. 이 웹 응용 프로그램은 100% 순수 자바이며 내부 구조 구성 또는 인프라 구성에 대해 다룰 필요가 없습니다.

스프링 부트는 메모리 내 관계형 데이터베이스 엔진인 H2를 지원하며 자동으로 연결을 생성합니다. Spring-jdbc를 사용하기 때문에, Spring Boot는 자동으로 JdbcTemplate을 생성합니다. @Autowired JdbcTemplate 필드는 자동으로 로드하여 사용할 수 있도록 합니다.

이 Application 클래스는 스프링 부트의 CommandLineRunner를 구현합니다. 즉, 응용 프로그램 컨텍스트가 로드된 후 run() 메서드를 실행합니다.

먼저, JdbcTemplate의 execute 메소드를 사용하여 DDL을 실행합니다.

둘째, 문자열 목록을 가져와서 Java8 스트림을 사용하여 Java 배열의 firstname/lastname 쌍으로 분할합니다.

그런 다음 JdbcTemplate의 batchUpdate 메소드를 사용하여 새로 생성된 테이블에 일부 레코드를 INSERT 합니다. 메서드 호출의 첫 번째 인수는 쿼리 문자열이고 마지막 인수 (Object 배열)는 "?"문자가있는 쿼리에 대체할 변수를 포함합니다.

단일 삽입 문장의 경우 JdbcTemplate의 insert 메소드가 좋습니다. 그러나 다중 삽입의 경우 batchUpdate를 사용하는 것이 좋습니다.

argument로 '?'을 사용하는 것은 변수를 바인드하도록 JDBC에게 지시하여 SQL 인젝션 공격을 피하기 위함입니다.

마지막으로 쿼리 메서드를 사용하여 테이블에서 조건과 일치하는 레코드를 검색합니다. 다시 "?"인수를 사용하여 쿼리 매개 변수를 만들고 호출할 때 실제 값을 전달합니다. 마지막 인수는 각 결과 행을 새 Customer 객체로 변환하는 데 사용되는 Java8 람다입니다.

Java8 람다는 Spring의 RowMapper와 같은 단일 메소드 인터페이스에 잘 매핑됩니다. Java7 이전 버전을 사용하는 경우 익명 인터페이스 구현을 쉽게 플러그인 할 수 있으며, 람다 표현식의 본문과 동일한 메소드 본문을 가질 수 있으며 Spring의 불편함 없이 작동합니다.

## 실행 가능한 JAR 파일로 빌드하기
Gradle 또는 Maven을 사용하여 명령 줄에서 응용 프로그램을 실행할 수 있습니다. 또는 모든 필요한 종속성, 클래스 및 자원을 포함하는 단일 실행 가능 JAR 파일을 빌드하고 실행할 수 있습니다. 따라서 개발 수명주기, 다양한 환경에 걸쳐 응용 프로그램으로 서비스를 쉽게 버전관리 및 배포할 수 있습니다.

### 이클립스에서 Maven 빌드하기
1. 프로젝트 우클릭 - Run As - Maven build... 클릭

![6](http://i.imgur.com/qwQBiup.jpg)

2. Goals에 package 입력 후 Run 버튼 클릭

![7](http://i.imgur.com/MMC50an.jpg)

3. 프로젝트의 target 폴더 하위에 jar 파일이 생성된 것을 확인합니다.

![Uploading 8.jpg… (ph1c8qigo)]()

4. 생성된 jar 파일을 적당한 디렉토리에 복사한 후 디렉토리에서 shift 키를 누른 채로 마우스 우클릭을 하여 '여기서 명령 창 열기 메뉴' 클릭

![9](http://i.imgur.com/bPbTsBu.jpg)

5. 'java -jar jar파일명' 명령어로 실행

![10](http://i.imgur.com/9BOG0h6.jpg)

6. 서비스 실행 로그 확인

![11](http://i.imgur.com/9ByB0CE.jpg)

## 요약
축하드립니다! 당신은 방금 Spring을 사용하여 간단한 JDBC 클라이언트를 개발했습니다.
Spring Boot는 메모리 풀 대신 외부 데이터베이스에 연결하는 것과 같이 연결 풀을 구성하고 사용자 정의하는 많은 기능을 가지고 있습니다. 자세한 내용은 가이드라인을 참조하십시오.
