---
title: 자바 8 Date
date: 2017-01-10 20:55:33
categories: [tech]
tags:
  - java
  - date
  - archives-2017
url : /2017/01/10/java8-date/
featuredImage: /images/java8-date/java_date.png
images :
  - /images/java8-date/java_date.png
---
이제까지 내 기억으로는 Date 관련 클래스를 아래처럼 점차 바꿔써온걸로 기억이 난다.
`java.util.Date` > `java.util.Calendar` > `org.joda.time`
그런데 java 8 버전에서 기존에 있었던 문제들을 개선해서 나왔다고 한다. ([네이버 HellowWorld 포스팅 참고](http://d2.naver.com/helloworld/645609)) `JSR-310` 이라는 표준명세로.
<!-- more -->
지금부터는 JAVA 8 에서 제공하는 API로 날짜 연산을 어떻게 하는지에 대해 알아보고자 한다. (물론 수많은 날짜 연산 방법을이 있지만 자주 쓰이는 부분들 위주로 정리해보자.)

- Date > String (format)

```java
LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));
```

- String > Date (format)

```java
LocalDateTime.parse("2017-01-01 12:30:00", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```

- 날짜/시간 증감

```java
LocalDateTime localDateTime = LocalDateTime.of(2017, 1, 1, 10, 0, 0);
localDateTime.plusDays(1);  // 일
localDateTime.plusMonths(1);  // 월
localDateTime.plusHours(1); // 시간
localDateTime.plusWeeks(1); // 주
localDateTime.minusYears(1);  // 년
localDateTime.minusMinutes(1);  // 분
```

더 다양한 내용들은 아래 URL 에서 확인이 가능하다.
https://docs.oracle.com/javase/tutorial/datetime/iso/overview.html
