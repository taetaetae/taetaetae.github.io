---
title: spring-boot에서 mybatis로 mysql 연동하기
date: 2019-04-21 22:47:04
categories:
  - tech
tags: 
  - spring-boot
  - mysql
  - mybatis
url : /2019/04/21/spring-boot-mybatis-mysql-xml/
featuredImage: /images/spring-boot-mybatis-mysql-xml/logo.jpg
images :
  - /images/spring-boot-mybatis-mysql-xml/logo.jpg
---
실무에서 개발을 하다보면 과거 누군가 잘 구성해 놓은 밥상(legacy)에 숟가락만 얹는 느낌으로 `로직 구현`만 할때가 있다. 그러다보면 각종 레이어가 어떻게 구성(설정)되어있는지도 모르고 <!-- more --> 간혹 설정에서 문제가 발생하면 "아 내가 이것도 모르고 이제까지 개발을 해왔나" 하는 자괴감이 들며 몇시간을 삽질하는 경우가 있다. 그게 지금의 필자인것 같다. (눙물...)

{{< image src="/images/spring-boot-mybatis-mysql-xml/mung.jpg" caption="출처 : http://blog.naver.com/PostView.nhn?blogId=ondo_h&logNo=221437452142" width="30%" >}}

사이드 프로젝트 초기셋팅을 하며 호기롭게 spring boot 최신버전에서 db를 연동하려 했는데 막상 완전 바닥부터 해본 경험이 적다보니 (spring boot 2 버전에서는 더욱더...) 어디서부터 뭘 설정을 해야할지... 그리고 `이럴때 보는` 도큐먼트를 봐도 잘 이해가 안되어 삽질을 해가며 당황하기 일쑤였다.
이번 포스팅에서는 아래와 같은 구성을 하는데 목표를 두고자 한다.
- Spring Boot 2 프로젝트를 처음 만들고 
- mybatis 를 사용해서
- mysql 을 연동하는것 (AWS 의 RDS를 사용, 추후 RDS사용법에 대해 블로깅 예정)

위와 같은 상황을 처음 접하는 분들께 도움이 되었으면 하는 바램으로 짧게나마 필자의 삽질기를 여행해보자.

## Spring boot 2 프로젝트 만들기
필자는 IntelliJ를 사용하고 있어서 새로 프로젝트를 만들려고 할때 클릭 몇번만으로 dependency 설정까지 다 해주기 때문에 편하고 좋았다. 혹 이클립스나 다른 IDE를 사용하고 있다면 https://start.spring.io/ 을 참고하면 도움이 될것같다. 여기서도 클릭 몇번으로 IntelliJ 에서 해주는 것처럼 내가 사용할 모듈을 선택하고 generate 를 누르면 프로젝트가 생성되어 다운로드 받아진다. (참 좋은 세상...)
우선 File → New → Project 를 눌러서 아래 창을 열어보자. 그리고 뭔가 다 해줄것 같은 (개발도 해주면 안되나...) `Spring Initializr`을 선택후 아래와 같은 설정을 적어준 뒤 다음을 눌러준다.

{{< image src="/images/spring-boot-mybatis-mysql-xml/1.jpg" caption="" width="80%" >}}

사용할 모듈을 선택해주자. 필자는 이것저것(?)을 도와주는 `lombok`과 `Mybatis`, `MySQL`을 선택하고 프로젝트를 생성하였다. 그러면 이쁜(?) pom.xml 과 함께 당장 개발을 시작할 수 있는 환경이 제공된다.
```xml
<dependencies>
	<dependency>
		<groupId>org.mybatis.spring.boot</groupId>
		<artifactId>mybatis-spring-boot-starter</artifactId>
		<version>2.0.1</version>
	</dependency>
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
	</dependency>	
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<optional>true</optional>
	</dependency>
</dependencies>
```

{{< image src="/images/spring-boot-mybatis-mysql-xml/2.jpg" caption="" width="80%" >}}

우선 여기까지 잘 되었는제 확인해보기 위해 Controller 에 현재시간을 출력하는걸 만들어 보고
```java
@RestController
public class ApiController {

	@GetMapping(path = "/helloWorld")
	public String helloWorld() {
		return LocalDateTime.now().format(DateTimeFormatter.ISO_LOCAL_DATE_TIME);
	}
}
```
톰켓을 실행해보면 정상적으로 접속과 출력이 되는것을 확인할 수 있다.

{{< image src="/images/spring-boot-mybatis-mysql-xml/3.jpg" caption="" width="80%" >}}


## MySQL 연동하기
필자가 허둥지둥 했던점 중 하나는 MyBatis와 MySQL을 동시에 연동하려고 하다보니 문제가 발생해도 어디서의 문제인지를 제대로 파악하지 못하고 삽질했다는 점이다. 여기서 정확히 짚고 넘어가면 우선 데이터를 연결해주는 ORM인 MyBatis를 셋팅해준 다음 MySQL을 연동해주는 식으로 분리해서 설정을 하면 햇갈리지 않고 (돌아가지 않고) 보다 빠르게 설정이 가능할것 같다. (여기서 순서는 중요하지 않고 별도로 설정해야 한다는 관점이 중요한것 같다.)
우선 `src/main/resources`폴더에 있는 `application.properties` 에 다음처럼 작성해주자.
```
spring.datasource.hikari.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.hikari.jdbc-url=jdbc:mysql://{url}:{port}/{db}
spring.datasource.hikari.username={id}
spring.datasource.hikari.password={password}
```
위의 jdbc-url 항목에서 AWS에서 제공하는 RDS를 사용하는 경우 RDS에서 제공해주는 엔드포인트와 포트를 적어주면 된다. (추후 AWS - RDS에 대해 블로깅 예정이다.)
Spring Boot 2.0 이후부터 기본적으로 사용되는 커넥션 풀이 HikariCP로 변경되었다고 한다. ([링크](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Release-Notes#hikaricp)) 커넥션 풀 종류중 성능이 좋다고 하는데 [링크](https://github.com/brettwooldridge/HikariCP)를 가보면 다른 커넥션 풀 라이브러리와 성능을 비교한 벤치마크 결과를 확인할 수 있다.
위처럼 `spring.datasource.hikari` 가 prefix로 붙고 각종 정보들을 적어주어 config 에서 인식될수 있도록 해주자. 그 다음 DataSource 설정을 해준다.

```java
@Slf4j
@Configuration
@PropertySource("classpath:/application.properties")
public class DatabaseConfiguration {
	@Bean
	@ConfigurationProperties(prefix = "spring.datasource.hikari")
	public HikariConfig hikariConfig() {
		return new HikariConfig();
	}

	@Bean
	public DataSource dataSource() {
		DataSource dataSource = new HikariDataSource(hikariConfig());
		log.info("datasource : {}", dataSource);
		return dataSource;
	}
}
```

위 내용은 DataSource 를 hikariConfig에서 설정한 정보로 만들어 준다는 의미이다. 이렇게만 하고 프로젝트를 다시 실행시켜보면 logger 에 의해 datasource 의 정보를 볼수가 있다.

```shell
2019-04-22 00:27:35.048  INFO 23040 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2019-04-22 00:27:36.221  INFO 23040 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2019-04-22 00:27:36.222  INFO 23040 --- [           main] c.e.m.config.DatabaseConfiguration       : datasource : HikariDataSource (HikariPool-1)
2019-04-22 00:27:36.527  INFO 23040 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'

```

여기까지 우선 Datasource 설정이 끝났다. 
Q : `com.mysql.cj.jdbc.Driver` 에서 `cj`가 뭐지?
A : 해당 클래스는 더이상 사용하지 않아 `com.mysql.jdbc.Driver`로 설정하고 실행시켜보면 아래 문구를 볼수가 있다.
> Loading class \`com.mysql.jdbc.Driver'. This is deprecated. The new driver class is \`com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.

당황하지 말고 클래스를 바꿔주자.

## MyBatis 연동하기
DB를 연동했으니 이제 쿼리를 작성하고 원하는 결과를 얻기위해 MyBatis를 활용할 차례다. 위에서 작성한 `DatabaseConfiguration`에 추가로 다음과 같이 작성해주자.

```java
public class DatabaseConfiguration {
	@Autowired
	private ApplicationContext applicationContext;

	@Bean
	public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
		SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
		sqlSessionFactoryBean.setDataSource(dataSource);
		sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath:/mapper/**/*.xml"));
		return sqlSessionFactoryBean.getObject();
	}

	@Bean
	public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
		return new SqlSessionTemplate(sqlSessionFactory);
	}
}
```

이 설정은 위에서 설정한 datasource를 사용하고 쿼리가 작성되는 xml위치를 지정해 줌으로써 추후 `Mapper` or `DAO` 레벨에서 사용되는 쿼리를 인식해주는 과정이다. 여기서 `classpath`는 `src/main/resourcs`이고 해당 쿼리가 있는 xml 위치는 본인의 취향대로 위치키시고 그에 맞도록 설정해주면 된다.
이렇게 한뒤 MySQL Workbench 로 DB에 접속후 임의의 데이터를 생성한 다음

{{< image src="/images/spring-boot-mybatis-mysql-xml/4.jpg" caption="" width="80%" >}}

DAO 를 만들어 주고 이를 호출해보면 정상적으로 데이터를 읽어오는것이 확인된다.
- DAO
```java
package com.express.magarine.api;

import org.apache.ibatis.session.SqlSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

@Repository
public class ApiDao {
	protected static final String NAMESPACE = "com.express.magarine.api.";

	@Autowired
	private SqlSession sqlSession;

	public String selectName(){
		return sqlSession.selectOne(NAMESPACE + "selectName");
	}
}
```
- query xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "mybatis-3-mapper.dtd">

<mapper namespace="com.express.magarine.api">
	<select id="selectName" resultType="string">
		SELECT name
		FROM test
		LIMIT 1
	</select>
</mapper>
```
- Controller
```java
@Slf4j
@RestController
public class ApiController {
	@Autowired
	private ApiDao apiDao;

	@GetMapping(path = "/helloWorld")
	public String helloWorld() {
		return String.format("%s %s", apiDao.selectName(), LocalDateTime.now().format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));
	}
}
```
- 결과

{{< image src="/images/spring-boot-mybatis-mysql-xml/5.jpg" caption="" width="80%" >}}

## 마치며

{{< image src="/images/spring-boot-mybatis-mysql-xml/gvsc.png" caption="" width="80%" >}}

이 코드를, 그리고 이 포스팅을 작성하기 직전까지만 해도 "그냥 하면 되는거 아니야?"라고 생각했지만 알고있는 지식과 막상 해보는건 정말 하늘과 땅차이 라는걸 다시한번 느끼게 되었다. (자괴감의 연속...) 더불어 Spring Boot 의 간편함에 놀라웠고 이제 회사일이 조금 잠잠해졌으니 (과연?) Spring Boot로 이것저것 만들며 스터디를 해야겠다고 다짐해본다.

참고 URL
- https://spring.io/guides/gs/accessing-data-mysql/
- http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/