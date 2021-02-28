---
title: lombok(롬복)소개 및 설치
date: 2017-02-22 17:48:42
categories: [tech]
tags:
  - java
  - lombok
  - archives-2017
url : /2017/02/22/lombok/
featuredImage: /images/lombok/lombok.png
images :
  - /images/lombok/lombok.png
  
---
일반적으로 자바개발을 하다보면 `Model` 을 만들고 각 멤버변수를 접근할수 있는 (각 요소들이 private 접근권한을 가지고 있을때) method 를 만들게 된다. IDE에서 제공하는 아래처럼... (윈도우/이클립스 기준)<!-- more -->
- get/set 메소드 :  `Alt` + `Shift` + `S` + `R`
- toString 메소드 :  `Alt` + `Shift` + `S` + `S`
- 기타 등등...
```java
public class Student {
    private int id;
    private String name;
    private int grade;
    private String department;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getGrade() {
        return grade;
    }

    public void setGrade(int grade) {
        this.grade = grade;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }

    @Override
    public String toString() {
        return "Student [id=" + id + ", name=" + name + ", grade=" + grade + ", department=" + department + "]";
    }    
}
```
이렇게 하는 방법도 있지만 어노테이션 설정으로 적용할수 있는 간단한 라이브러리를 소개하고자 한다.
바로 `lombok`, 공식 홈페이지 : https://projectlombok.org
설치 및 사용방법은 아주 간단하다. 공식 홈페이지에서 jar를 다운받고 실행, 아래처럼 이클립스 실행파일 경로를 설정해준다음에 인스톨을 누르면 된다.
![](/images/lombok/lombok.png)
maven 환경에서 dependency를 가져오기 위해서는 당연히 추가설정을 해줘야 한다.
```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.10</version> <!--버전은 그때 맞춰서-->
</dependency>
```
실제로 코드상에서 사용방법은 다음과 같다. 정말 간단히, 어노테이션만 적용해주면 끝!
```java
import lombok.Data;

@Data
public class Student {
    private int id;
    private String name;
    private int grade;
    private String department;
}
```
그럼 이렇게 기본적인 method들이 생성된다.
![](/images/lombok/lombok-annotation.png)
일반적으로 `@Data`를 사용하고 상황에 따라 필요한 어노테이션만 지정도 가능하다고 한다.
- @Getter and @Setter
- @NonNull
- @ToString
- @EqualsAndHashCode
- @Data
- @Cleanup
- @Synchronized
- @SneakyThrows
참고 URL : http://jnb.ociweb.com/jnb/jnbJan2010.html
