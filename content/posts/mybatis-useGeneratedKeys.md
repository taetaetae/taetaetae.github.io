---
title: mybatis insert/update 쿼리실행후 결과 가져오기
date: 2017-04-04 11:41:28
categories: [tech]
tags:
  - mybatis
  - oracle
  - archives-2017
url : /2017/04/04/mybatis-useGeneratedKeys/
featuredImage: /images/mybatis-useGeneratedKeys/mybatis.png
images :
  - /images/mybatis-useGeneratedKeys/mybatis.png
---

`Select`문이 아닌 다른 `SQL Query`(insert, update 등) 를 실행하고서 결과를 봐야하는 상황이 생긴다. 정확히 잘 수행되었나에 대한 확인. 어떻게 쿼리가 잘 수행되었나를 확인하는 방법은 다음과 같다.
※ 참고 url : http://www.mybatis.org/mybatis-3/ko/sqlmap-xml.html

## useGeneratedKeys, keyProperty 옵션
사용하는 데이터베이스가 자동생성키를 지원한다면(mySql 같은) 해당옵션을 이용해 결과를 리턴 받을수 있다.
예로들어 파라미터로 아래 모델객체를 넘긴다고 가정하고
```java
public Student {
  int id;
  String name;
  String email;
  Date regist_date;
}
```
아래 mybatis 구문으로 insert를 시도하게되면, 파라미터로 넘긴 Student 객체의 id값에 insert 했을때의 key값(id)이 들어오게 된다.
```java
Student student = new Student();
student.setName('bla');
student.setEmail('bla@naver.com');

mapper.insertStudents(student); // 쿼리실행
student.getId(); // 추출 가능
```
```xml
<insert id="insertStudents" useGeneratedKeys="true" keyProperty="id" parameterType="Student">
  insert into Students ( name, email )
  values ( #{name}, #{email} )
</insert>
```

## selectKey 옵션
Oracle 같은 경우는 Auto Increment 가 없고 Sequence를 사용해야만 하기 때문에 위 옵션을 사용할수가 없다. 하지만 다른 우회적인(?) 방법으로 위와같은 효과를 볼수가 있다.
파라미터의 모델이나 java구문은 위와 동일하고 xml 쿼리 부분만 아래와 같이 설정해주면 된다.
```xml
<insert id="insertStudents" parameterType="Student">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select SEQ_ID.nexyval FROM DUAL
  </selectKey>
  insert into Students
    (id, name , email)
  values
    (#{id}, #{name}, #{email})
</insert>
```
위와같은 코드에서 쿼리가 실행되기 전에 id값에 Sequence에 의해 값을 셋팅하게 되고, 자동적으로 해당 값을 Student의 id에 set하게 되서 동일한 결과를 볼수가 있다.

항상 테이블의 key값에만 해당하는것이 아니다. key값과는 전혀 상관없는 값도 `selectKey` 구문으로 리턴할수가 있는데 `order`옵션을 `AFTER`로 주고 리턴하고자 하는 값을 명시해주면 된다.
아래 코드에서는 입력할시 id값을 Sequence에서 가져오는게 아니라 수동으로 넣어주고, 입력했던 id에 맞는 regist_date 값을 리턴받아 위에서처럼 동일하게 값를 가져올수 있다.
```xml
<insert id="insertStudents" parameterType="Student">
  <selectKey keyProperty="regist_date" resultType="java.util.Date" order="AFTER">
    select regist_date FROM students WHERE id = #{id}
  </selectKey>
  insert into Students
    (id, name , email, regist_date)
  values
    (#{id}, #{name}, #{email}, syadate)
</insert>
```
