---
title: Spring Transactional 설정 및 주요속성
date: 2017-01-08 17:19:30
categories:
  - tech
tags:
  - spring
  - Transaction
  - archives-2017
url : /2017/01/08/transactional-setting-and-property/
featuredImage: /images/transactional-setting-and-property/transactional.png

---
지난번에는 트랜잭션의 설정값에 대해 알아본 바 있다. [ [Spring Transaction 옵션](/2016/10/08/20161008) ]
이번 포스팅에서는 실제로 스프링 환경에서 어떤식으로 설정해야 `@Transactional` 어노테이션을 사용할수 있는지, 그리고 어떤 속성들이 있는지에 대해 알아보고자 한다.<!-- more -->

## 설정
기존 xml방식에서는 다음과 같이 설정을 한다.
```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
     <property name="dataSource" ref="dataSource"/>
</bean>
<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
```
혹, JavaConfig 방식으로 설정하기 위해서는 다음과 같이 설정한다.
```java
@EnableTransactionManagement
public class AppConfig {
    ...

    @Bean
    public PlatformTransactionManager transactionManager() throws URISyntaxException, GeneralSecurityException, ParseException, IOException {
        return new DataSourceTransactionManager(dataSource());
    }
}
```
위와같이 설정을 해주면 트랜잭션을 설정하고자 하는 곳 어디서든 `@Transactional` 어노테이션을 지정해서 적용이 가능하다.
```java
public class UserService{

  @Transactional
  public boolean insertUser(User user){
    ...
  }
}
```

## 주요속성
`@Transactional` 어노테이션의 주요속성은 다음과 같다.

| 속성 |	설 명	| 사용 예 |
| --- | --- | --- |
| isolation	| Transaction의 isolation Level. 별도로 정의하지 않으면 DB의 Isolation Level을 따름.	|  @Transactional(isolation=Isolation.DEFAULT) |
| propagation	| 트랜잭션 전파규칙을 정의 , Default=REQURIED |	@Transactional(propagation=Propagation.REQUIRED) |
| readOnly |	해당 Transaction을 읽기 전용 모드로 처리 (Default = false)	| @Transactional(readOnly = true) |
| rollbackFor |	정의된 Exception에 대해서는 rollback을 수행 | 	@Transactional(rollbackFor=Exception.class) |
| noRollbackFor	| 정의된 Exception에 대해서는 rollback을 수행하지 않음. |	@Transactional(noRollbackFor=Exception.class) |
| timeout	| 지정한 시간 내에 해당 메소드 수행이 완료되지 않은 경우 rollback 수행.  -1일 경우 no timeout (Default = -1) | @Transactional(timeout=10) |

## 마치며
자칫 잘못했다가는 원치않는 트랜잭션으로 잘못된 결과를 초래할수 있기때문에 기본값은 숙지하는게 좋을것 같다.
