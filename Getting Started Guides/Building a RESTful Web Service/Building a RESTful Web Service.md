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

2. Maven Project 선택 후 Next 클릭

![create_mavenproject_2](http://i.imgur.com/91iRfaD.jpg)

3. 아래 그림과 같은 상태로 놓고 Next 클릭

![create_mavenproject_3](http://i.imgur.com/wLnU34q.jpg)

4. Artiface Id가 maven-archetype-quickstart인 archetype 선택 후 Next 클릭

![create_mavenproject_4](http://i.imgur.com/rZN0HT9.jpg)

5. Group Id, Artifact Id, Version을 아래의 그림과 같이 입력 후 Finish 클릭

![create_mavenproject_5](http://i.imgur.com/zR8Wnd2.jpg)

6. 최초에 이클립스에서 Maven Project를 생성하면 JDK가 1.5로 설정되어있을텐데, 우리는 1.8을 사용해야하므로 프로젝트의 JDK 버전을 변경해줘야 한다.
프로젝트 우클릭 - Build Path - Configure Build Path... 클릭

![create_mavenproject_6](http://i.imgur.com/Lfu1hyd.jpg)

7. JRE System Library 더블클릭

![create_mavenproject_7](http://i.imgur.com/538Wlez.jpg)

8. Alternate JRE 선택 후 설치한 JDK 1.8을 선택하고 Finish

![create_mavenproject_8](http://i.imgur.com/pe36mRT.jpg)

9. 좌측 트리메뉴에서 Java Compiler 메뉴 선택

![create_mavenproject_9](http://i.imgur.com/XAAmdty.jpg)

10. Compiler compliance level을 1.8로 변경 후 OK 버튼 클릭

![create_mavenproject_10](http://i.imgur.com/EUhY452.jpg)

### 프로젝트 생성 후 pom.xml 편집
pom.xml을 다음과 같이 작성한다

~~~XML
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.springframework</groupId>
  <artifactId>gs-rest-webservice</artifactId>
  <version>0.1.0</version>
  <packaging>jar</packaging>

  <name>gs-rest-webservice</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>
  </properties>

  <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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
