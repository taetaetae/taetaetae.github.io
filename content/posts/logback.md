---
title: logback 설정하기
date: 2017-02-19 15:10:45
categories: [tech]
tags:
  - java
  - logback
  - archives-2017
url : /2017/02/19/logback/
featuredImage: /images/logback/logback.jpg
---
자바 개발자라면 한번쯤은 들어봤고, 한번쯤은 사용했을법한 logger 로 `log4j`가 있을것이다. 하지만 최근들어 `logback`이라는것을 알게되었고, 왜 `logback`을 사용해야 하는 이유라는 글이 있을정도로 여러 측면에서 개선이 된듯 하다. <!-- more -->([링크](https://beyondj2ee.wordpress.com/2012/11/09/logback-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%95%BC-%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0-reasons-to-prefer-logback-over-log4j))
이번에 작성할 글의 목적은 `logback`을 설정하고 어떻게 사용하는지에 대해 작성해 보고자 한다.
※ 공식사이트 : https://logback.qos.ch/

## pom.xml
maven구조라고 가정했을때 `logback Dependency`를 가져오기 위해서는 아래와 같이 pom.xml 에 설정해 주면 된다.
```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.1.7</version> <!--버전은 상황에 따라 -->
</dependency>
```

## 로그레벨
`ERROR`, `WARN`, `INFO`, `DEBUG` or `TRACE`

#### # logback 설정파일
일반적으로 `logback.xml` 이라는 이름으로 만들어 `src/main/resources/`아래에 위치하게 된다. Spring-Boot 환경에서는 `logback-spring.xml` 이라는 이름으로 설정해야 하는데 `logback.xml`로 설정하면 스프링부트가 설정하기 전에 로그백 관련한 설정을 하기 때문에 제어할 수가 없게 된다.
( 공식사이트 메뉴얼 : https://logback.qos.ch/documentation.html )
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />
    <include resource="org/springframework/boot/logging/logback/console-appender.xml" />    

    <!-- 변수 지정 -->
    <property name="LOG_DIR" value="/logs" />
    <property name="LOG_PATH_NAME" value="${LOG_DIR}/data.log" />

    <!-- FILE Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH_NAME}</file>
        <!-- 일자별로 로그파일 적용하기 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH_NAME}.%d{yyyyMMdd}</fileNamePattern>
            <maxHistory>60</maxHistory> <!-- 일자별 백업파일의 보관기간 -->
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%-5p] [%F]%M\(%L\) : %m%n</pattern>
        </encoder>
    </appender>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
      <layout class="ch.qos.logback.classic.PatternLayout">
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%-5p] [%F]%M\(%L\) : %m%n</pattern>
      </layout>
    </appender>

    <!-- TRACE > DEBUG > INFO > WARN > ERROR, 대소문자 구분 안함 -->
    <!-- profile 을 읽어서 appender 을 설정할수 있다.(phase별 파일을 안만들어도 되는 좋은 기능) -->
    <springProfile name="local">
      <root level="DEBUG">
        <appender-ref ref="FILE" />
        <appender-ref ref="STDOUT" />
      </root>
    </springProfile>
    <springProfile name="real">
      <root level="INFO">
        <appender-ref ref="FILE" />
        <appender-ref ref="STDOUT" />
      </root>
    </springProfile>
</configuration>
```


## java 코딩에서의 로깅
실제 사용은 다음과 같이 `LoggerFactory`를 이용해서 사용하거나 `Lombok`어노테이션을 활용하면 심플하게 사용이 가능하다.
- LoggerFactory 사용

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Foo {
    static final Logger logger = LoggerFactory.getLogger(Foo.class);

    public void test() {
        logger.debug("ID : {}", "foo");
    }
}
```
- Lombok 어노테이션 사용

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Foo {

    public void test() {
        log.debug("ID : {}", "foo");
    }
}
```
## 마치며
일반적인 웹 어플리케이션에서는 WAS에서 로깅을 따로 관리하고 있기 때문에 file 로 로깅을 할 필요는 없을것 같다.(일반 jar 형태에서는 파일 로깅이 필요 할수도...)

## 참고사이트
- http://yookeun.github.io/java/2015/11/10/log4jtologback/
- http://java.ihoney.pe.kr/397
- https://logback.qos.ch/
