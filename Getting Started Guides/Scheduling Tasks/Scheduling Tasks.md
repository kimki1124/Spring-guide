# Scheduling Tasks

이 가이드는 Spring을 사용하여 스케줄을 예약하는 단계를 안내합니다.

## 당신이 만들 것
Spring의 @Scheduled 어노테이션을 사용하여 5초마다 현재 시간을 출력하는 응용 프로그램을 빌드합니다.

## 당신에게 필요한 것
* 약 15분의 시간
* 텍스트 에디터 또는 IDE
* JDK 1.8 또는 그 이상버전
* Gradle 2.3버전 이상 또는 Maven 3.0버전 이상

## 이 가이드를 완료하는 방법
대부분의 Spring Getting Started 가이드와 마찬가지로, 처음부터 시작하여 각 단계를 완료하거나 이미 익숙한 기본 설정 단계를 건너 뛸 수 있습니다. 어쨌든, 당신은 working code로 끝납니다.

##### 처음부터 시작하려면 'Maven으로 빌드'메뉴로 이동하십시오.

##### 기본 사항을 건너 뛰려면 다음을 수행하십시오.
* 이 가이드의 소스 저장소를 다운로드하고 압축을 해제하거나 Git을 사용하여 복제하십시오. git clone https://github.com/spring-guides/gs-scheduling-tasks.git
* gs-scheduling-tasks/initial 디렉토리로 이동하십시오
* '스케줄링된 작업 만들기'메뉴로 이동하십시오

작업이 끝나면 gs-scheduling-tasks / complete의 코드와 결과를 비교할 수 있습니다.

### 이클립스를 사용하여 프로젝트 만들기

1. New - Other... 메뉴 선택

![1](http://i.imgur.com/4CsujJC.jpg)

2. Spring Starter Project 선택 후 Next 클릭

![2](http://i.imgur.com/ep1dKkS.jpg)

3. Name, Artifact, Package를 변경 후 Next 클릭

![3](http://i.imgur.com/X3ABNIT.jpg)

4. Finish 클릭

![4](http://i.imgur.com/1DzjDoO.jpg)

5. pom.xml을 다음과 같이 작성
~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>SchedulingTask</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>SchedulingTask</name>
	<description>SchedulingTask project for Spring Boot</description>

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

## 스케줄링된 작업 만들기
이제 프로젝트를 설정 했으므로 스케줄링 작업을 만들 수 있습니다.
1. src/main/java 하위에 hello 패키지 생성

![5](http://i.imgur.com/IaPCrKf.jpg)

2. hello 패키지 하위에 ScheduledTasks.java 클래스파일 생성

![6](http://i.imgur.com/FhVP3CC.jpg)

3. ScheduledTasks.java 파일에 다음과 같이 소스 작성
~~~java
package hello;

import java.text.SimpleDateFormat;
import java.util.Date;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class ScheduledTasks {

    private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);

    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        log.info("The time is now {}", dateFormat.format(new Date()));
    }
}
~~~
Scheduled 어노테이션은 특정 메소드가 실행되는시기를 정의합니다. 참고 :이 예제에서는 각 호출의 시작 시간에서 측정 된 메서드 호출 간격을 지정하는 fixedRate를 사용합니다. fixedDelay와 같은 다른 옵션이 있습니다.이 옵션은 작업 완료시 측정 된 호출 간격을 지정합니다. 보다 정교한 작업 스케줄링을 위해 @Scheduled(cron = "..") 표현식을 사용할 수도 있습니다.

## 스케줄링 사용
예약된 작업을 웹 응용 프로그램 및 WAR 파일에 포함 할 수 있지만 아래에 설명된 간단한 방법은 독립 실행 형 응용 프로그램을 만듭니다. Java main() 메소드로 구동되는 실행 가능한 단일 JAR 파일로 모든 것을 패키지화합니다.

1. hello 패키지 하위에 Application.java 클래스파일 생성

![7](http://i.imgur.com/J9AFCo0.jpg)

2. Application.java 파일에 다음과 같이 소스 작성
~~~java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
public class Application {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class);
    }
}
~~~
@SpringBootApplication은 다음을 모두 더한 편리한 어노테이션입니다.

* @Configuration은 클래스를 응용 프로그램 컨텍스트에 대한 Bean 정의의 소스로 태그 지정합니다.
* @EnableAutoConfiguration은 Spring Boot에게 클래스 패스 설정, 다른 bean 및 다양한 속성 설정을 기반으로 bean 추가를 시작하도록 지시합니다.
* 일반적으로 @EnableWebMvc를 Spring MVC app에 추가 할 것이지만 Spring Boot는 classpath에서 spring-webmvc를 인식할 때 자동으로 추가합니다. 이것은 응용 프로그램에 웹 응용 프로그램으로 플래그를 지정하고 DispatcherServlet 설정과 같은 주요 동작을 활성화합니다.
* @ComponentScan은 Spring에게 hello 패키지의 다른 구성 요소, 구성 및 서비스를 찾아서 컨트롤러를 찾을 수있게합니다.

main() 메소드는 Spring Boot의 SpringApplication.run() 메소드를 사용하여 애플리케이션을 시작합니다. 한 줄의 XML도 없다는 것을 눈치 챘습니까? web.xml 파일도 없습니다. 이 웹 응용 프로그램은 100% 순수 자바이며 내부 구조 구성 또는 인프라 구성에 대해 다룰 필요가 없습니다.

@EnableScheduling은 백그라운드 태스크 실행 프로그램이 작성되도록 합니다. 그것이 없으면 아무 것도 스케줄되지 않습니다.

## 실행 가능한 JAR 파일로 빌드하기
Gradle 또는 Maven을 사용하여 명령 줄에서 응용 프로그램을 실행할 수 있습니다. 또는 모든 필요한 종속성, 클래스 및 자원을 포함하는 단일 실행 가능 JAR 파일을 빌드하고 실행할 수 있습니다. 따라서 개발 수명주기, 다양한 환경에 걸쳐 응용 프로그램으로 서비스를 쉽게 버전관리 및 배포할 수 있습니다.

### 이클립스에서 Maven 빌드하기
1. 프로젝트 우클릭 - Run As - Maven build... 클릭

![8](http://i.imgur.com/Bp7WAF6.jpg)

2. Goals에 package 입력 후 Run 버튼 클릭

![9](http://i.imgur.com/SK58neq.jpg)

3. 프로젝트의 target 폴더 하위에 jar 파일이 생성된 것을 확인합니다.

![10](http://i.imgur.com/Qu1c3a2.jpg)

4. 생성된 jar 파일을 적당한 디렉토리에 복사한 후 디렉토리에서 shift 키를 누른 채로 마우스 우클릭을 하여 '여기서 명령 창 열기 메뉴' 클릭

![11](http://i.imgur.com/r6itSvd.jpg)

5. 'java -jar jar파일명' 명령어로 실행

![12](http://i.imgur.com/ob0EwDL.jpg)

6. 서비스 실행 로그 확인

![13](http://i.imgur.com/evCzBaQ.jpg)

로깅 출력이 표시되고 로그에서 백그라운드 스레드에 있음을 알 수 있습니다. 5초마다 예약된 작업이 실행됩니다.

## 요약
축하합니다! 예약된 작업으로 응용 프로그램을 만들었습니다. 실제 코드는 빌드 파일보다 짧습니다! 이 기술은 모든 유형의 응용 프로그램에서 작동합니다.
