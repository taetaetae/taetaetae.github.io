---
title: eclipse에서 spring-boot로 web 만들기
date: 2017-02-27 14:37:27
categories: [tech]
tags:
  - spring-boot
  - eclipse
  - archives-2017
url : /2017/02/27/spring-boot-eclipse/
featuredImage: /images/spring-boot-eclipse/spring-boot-logo.jpg

---
Spring 환경에서 웹 어플리케이션을 만들어야 한다면 pom.xml 에 이런저런 설정들을 적어줘야 했다. 하지만 이런 수고(?)를 덜어줄수 있는 방법중에 한가지가 바로 Spring Boot로 만드는 방법인데, 이클립스 환경에서 만드는 법을 정리하고자 한다.
<!-- more -->

## new > Maven Project
빈 Maven Project 를 만드는 방법은 아주 간단하니 생략하고... 만들게 되면 pom.xml 은 아래처럼 아주 깔끔한(?)상태로 만들어지게 된다.
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>boot</artifactId>
  <version>0.0.1-SNAPSHOT</version>
</project>
```
그러면 이 비어있는 pom.xml 에 Spring-Boot 에 필요한 설정들을 추가해주기로 한다.
```xml
<parent> <!--boot의 스타터를 사용하겠다고 명시적으로 설정-->
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.1.RELEASE</version>
    <relativePath />
</parent>

<dependencies>
    <dependency> <!--boot에서 스타터패키지로 제공해주는 것들중에 web 설정 부분 -->
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
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
```
그다음 임의의 java 클래스를 하나 만들고 거기에 아래처럼 설정하면 끝
```java
@SpringBootApplication // @Configuration + @EnableAutoConfiguration + @ComponentScan 들의 종합 어노테이션
public class TestApplication{
    public static void main(String[] args) throws Exception {
        SpringApplication.run(TestApplication.class, args);
    }
}
```
Spring Boot 에서는 내장WAS를 가지고 있기 때문에 main 메소드에서 우클릭후 `run AS → Spring Boot App` 을 선택해주면 8080포트로 띄워지게 된다.

## new > Spring Starter project
(STS가 설치되어있다는 가정하에)이 메뉴를 사용하면 위에서 했던 일련의 설정들을 자동으로 해주게 된다. 간단한 내용이니 next를 해주다 마지막에 `Dependencies` 설정하는 부분에서 Web 을 체크해주고 `Finish 버튼`을 누르면 끝

## Spring Initializr (start.spring.io)
http://start.spring.io/ 에 들어가보면 구지 설명하지 않아도 친절하게 Generate 해주는 페이지가 보인다. 여기서 web 을 `Dependencies`에 추가하고 Generate를 하면 해당 프로젝트가 압축된 상태로 다운이 받아지게 되고 이를 IDE 에서 열어보면 위에서 했던 일련의 과정들이 설정되어 있는것을 확인해볼수가 있다.

## 내장톰켓을 사용안하고 별도 톰켓을 사용해야 하는 경우
Spring boot는 자체적으로 내장 WAS를 가지고 있다. 하지만 관리포인트나 이런저런 이유로 내장톰켓을 사용하지 못하는 환경이라면 다음과 같은 설정을 해주면 된다.
- 일반적으로 빌드가 되면 `jar`로 만들어 질텐데 `war`로 빌드 되도록 수정을 해야한다. (was가 WAR를 물고 떠야하기 때문..)

```xml
<packaging>war</packaging>
```

- dependency 에 tomcat을 추가해준다.

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-tomcat</artifactId>
  <scope>provided</scope>
</dependency>
```

- 아래처럼 main 메소드가 있는 클래스에 `SpringBootServletInitializer`를 상속받게 한 후 `configure`메소드를 오버라이딩 해준다.

```java
@SpringBootApplication
public class TestApplication extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(TestApplication.class);
    }
    public static void main(String[] args) throws Exception {
        SpringApplication.run(TestApplication.class, args);
    }
}
```

- 톰켓에 띄우기 위하여 프로젝트 설정(Project Facets)에서 `Dynamic Web Module`을 체크해준다.

참고 URL
- http://www.donnert.net/86
- http://opennote46.tistory.com/124
