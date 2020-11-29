---
title: Oracle + Mybatis 환경에서의 Date 다루기
date: 2017-03-23 11:16:05
categories: [tech]
tags:
  - mybatis
  - date
  - oracle
  - archives-2017
url : /2017/03/23/oracle-mybatis-date/
featuredImage: /images/oracle-mybatis-date/oracle_mybatis_date.png
images :
  - /images/oracle-mybatis-date/oracle_mybatis_date.png

---
## 상황
- Oracle, Java 8, mybatis3 환경
- Date컬럼에 데이터가 있는데 이를 select query로 조회하여 Model에 바인딩 시키고자 함.
<!-- more -->
- 쿼리에 아무 기능을 추가하지 않고 Date 형태로 Model에 바인딩을 하면 시분초가 없어진 `2017-01-01 00:00:00` 형태로 남게됨
- 그래서 아래처럼 쿼리 작성할 때마다 TO_CHAR를 사용해서 포맷에 맞추어 형변환을 시키고 Date 또는 String으로 Model에 바인딩 하곤 했음.

```sql
SELECT
TO_CHAR(reg_ymdt, 'YYYY-MM-DD HH24:MI:SS') AS registDate
FROM
...
```

- 이렇게 하다보니 query 만들때마다 형변환하는 쿼리를 만들어줘야하고, 자칫 포맷형식을 다르게 적으면 엉뚱한 결과를 초래하거나, Date형을 그대로 받아 사용해야하는 상황에서는 다시 형변환하는 과정(`String to Date`)을 해줘야만 함. .. `귀차니즘의 시작 : 삽질`

## 1. 삽질의 시작
### 1-1. `오라클의 DATE형` → `java.sql.Date` 의 경우
- mybatis에서는 자동적으로 `org.apache.ibatis.type.SqlDateTypeHandler`를 호출하게됨 [mybatis 3 문서 참고](http://www.mybatis.org/mybatis-3/ko/configuration.html#typeHandlers)
- 해당 핸들러의 내부 데이터 변환 코드는 다음과 같음

```java
@Override
public Date getNullableResult(ResultSet rs, String columnName)  throws SQLException {
	return rs.getDate(columnName);
}
```
- `java.sql.ResultSet.getDate()`메소드를 호출하면 실제 'yyyy-mm-dd' 만 가져와 리턴하게됨 (여기서 디버깅 해보면 rs.getTimestamp(columnName)값은 시분초까지 다 들어가 있음)
- 따라서 시간값이 없는 `yyyy-mm-dd` 형태로 리턴이 됨

### 1-2. `오라클의 DATE형` → `java.util.Date` 의 경우
- mybatis에서는 자동적으로 `org.apache.ibatis.type.DateOnlyTypeHandler`를 호출하게됨 [mybatis 3 문서 참고](http://www.mybatis.org/mybatis-3/ko/configuration.html#typeHandlers)
- 해당 핸들러의 내부 데이터 변환 코드는 다음과 같음

```java
@Override
public Date getNullableResult(ResultSet rs, String columnName)  throws SQLException {
	java.sql.Date sqlDate = rs.getDate(columnName);
	if (sqlDate != null) {
		return new java.util.Date(sqlDate.getTime());
	}
	return null;
}
```

- 위의 `org.apache.ibatis.type.SqlDateTypeHandler` 변환코드에서 발생한 문제점과 같이 `yyyy-mm-dd` 만 가져와서 java.sql.Date 객체를 만들고, 이 정보를 토대로 java.util.Date 객체를 만들게 되는데 앞서 시간값을 뺀 정보로 만들어졌기 때문에 결국 동일하게 `yyyy-mm-dd` 형태로 리턴이 됨

## 2. 삽질완료, 해결의 시작
- 오라클 + mybatis 환경에서 Date타입을 다루기 위해서는 타입핸들러를 명시적으로 만들어줘야 한다는걸 알게됨.
#### 2-1. `오라클의 DATE형` → `java.sql.Date` 의 경우
- 아래처럼 코드를 작성하여 커스텀 핸들러를 만들어 등록을 시켜준다.
- mybatis-config.xml

```xml
<typeHandlers>
	<typeHandler handler="com.naver.dbill.admin.common.handler.CustomDateHandler"/>
</typeHandlers>
```
- CustomDateHandler.java

```java
...
import java.sql.Date;
...
public class CustomDateHandler extends BaseTypeHandler<Date> {
	...
	@Override
	public Date getNullableResult(ResultSet rs, String columnName) throws SQLException {
		Timestamp sqlTimestamp = rs.getTimestamp(columnName);
		if (sqlTimestamp != null) {
			return new Date(sqlTimestamp.getTime());
		}
		return null;
	}
	...
}
```
- 위 코드를 작성하고 실행해보면 정상적으로 시분초 값이 있는 완전한 Date 형태를 볼수 있다.
#### 2-2. `오라클의 DATE형` → `java.util.Date` 의 경우
- 아래처럼 코드를 작성하여 커스텀 핸들러를 만들어 등록을 시켜준다.
- 단, [mybatis 3 문서](http://www.mybatis.org/mybatis-3/ko/configuration.html#typeHandlers)를 보면 `java.sql.Date` 와는 다르게 기본으로 설정된 typeHandler가 JDBC에 따라 3가지가 있다.
- 따라서 작성한 커스텀 핸들러를 적용하기 위해서는 명시적으로 `자바타입` 과 `JDBC타입` 을 적어줘야 정상적으로 오버라이딩이 되어 해당 핸들러를 사용하게 된다.
- mybatis-config.xml

```xml
<typeHandlers>
		<typeHandler handler="com.naver.dbill.admin.common.handler.CustomDateHandler" javaType="java.util.Date" jdbcType="DATE"/>
</typeHandlers>
```
- CustomDateHandler.java 는 위와 동일하다. ( import java.util.Date; 사용으로 변경 )

### 삽질하며 알게된 보너스 지식
- `java.sql.Date` 는 `java.util.Date` 을 상속받았다.

```java
public class Date extends java.util.Date {
}
```
- 검색을 하다보면 알수있겠지만  `java.sql.Date` 는 JDBC등을 이용해서 데이터베이스의 데이터를 사용하는데 적합하고, `java.util.Date` 은 보다 범용적인 날짜나 시각정보를 다룰때 적합하다고 한다.
- toString 메소드의 리턴 Format 형태
  - `java.sql.Date` : yyyy-mm-dd
  - `java.util.Date` : EEE MMM dd HH:mm:ss zzz yyyy
- mybatis 에서 형변환은 [mybatis 3 문서](http://www.mybatis.org/mybatis-3/ko/configuration.html#typeHandlers)에 나와있는 자바타입과 JDBC타입이 일치할 경우에 해당 타입 핸들러를 기본으로 사용하게 된다.

### 정상혁 님 조언 ( http://d2.naver.com/helloworld/645609 작성하신분 )
  - Oracle의 JDBC 드라이버가 예상 밖으로 동작하네요. Oracle의 DATE 타입도 문서를 보니 시분초까지 저장하게 되어 있는데, Oracle JDBC 구현체가 DATE 타입의 철학을 오해한게 아닌가하는 생각도 듭니다.
  - 참고로 java.sql.Date, java.sql.TimeStamp는 잘못된 설계라는 비판이 많습니다.
  - 저도 Java의 날짜와 시간 API 라는 글에서 아래와 같이 적은 적이 있습니다.
>java.sql.Date 클래스는 상위 클래스인 java.util.Date 클래스와 이름이 같다. 이 클래스를 두고 Java 플랫폼 설계자는 클래스 이름을 지으면서 깜빡 존 듯하다는 조롱까지 나왔다.[24]

> 그리고 이 클래스는 Comparable 인터페이스에 대한 정의를 클래스 선언에서 하지 않았기 때문에 Comparable과 관련된 Generics 선언을 복잡하게 만들었다.[25]

> java.sql.TimeStamp 클래스는 java.util.Date 클래스에 나노초(nanosecond) 필드를 더한 클래스이다. 이 클래스는 equals() 선언의 대칭성을 어겼다. Date 타입과 TimeStamp 타입을 섞어 쓰면 a.equals(b)가 true라도 b.equals(a)는 false인 경우가 생길 수 있다.[26]


  - 이런 이유 때문에 저는 가급적 Java8에 나온 ZonedDateTime 류를 모델객체에서는 쓰고 있기는합니다. 하지만 그 클래스도 JDBC 레벨에서 제대로 매핑을 안 해주는 경우가 있어서 converter, typeHandler류를 따로 만들어야합니다. 참고로 Spring JDBC에서는 아래와 같이 Converter를 만들어서 해결했습니다.

```java
public class ZonedDateTimeConverter implements Converter<Timestamp, ZonedDateTime> {
  @Override
  public ZonedDateTime convert(Timestamp source) {
      return ZonedDateTime.ofInstant(source.toInstant(), ZoneId.of("UTC"));
  }
}
```
  - java.sql.Date vs java.util.Date 둘 중에 선택한다면 모델 객체에서는 java.util.Date가 더 어울린다는 생각이 듭니다.  모델에 java.sql.Date가 있는 것은  Controller에서 SqlException이 있는 것 같은 비슷한 느낌이랄까요..^^;

## 마치며
- 삽질을 하더라도 가급적이면 영양가 있는 삽질이 되야 할것같다. (하루종일 이것 붙잡다가 업무를 못해버리는;;)
- API문서, 블로그문서, 검색결과에 맹신하지말고 실제 소스까지 들어가봐서 확신을 갖자.
