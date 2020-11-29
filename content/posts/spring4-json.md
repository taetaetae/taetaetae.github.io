---
title: spring4에서 json view 활용하기(with @ResponseBody)
date: 2017-01-07 15:47:59
tags: 
  - spring
  - json
  - archives-2017
categories:
  - tech
url : /2017/01/07/spring4-json/
featuredImage: /images/spring4-json/json.png  
---
수많은 블로거분들의 도움을 받고자 구글링을 해서 적용을 해봤지만 너무많은 삽질을 했다.(해봤던 방식은 `jsonViewResolver` 를 따로 설정해보거나, `@RequestMapping` 옵션을 바꿔보는 수준..) 특히나 Spring설정방식이 예전 방식이였던 `xml`이 아닌 `javaconfig`였기 때문에 더욱더 자료가 없었고.. <!-- more --> 한참을 삽질하다 해결을 하여 포스팅하게 된다. 우선 환경은 `spring 4.3.4.RELEASE`, `Maven`, `jdk8`임을 밝힌다.

## pom.xml
`jackson-mapper-asl`을 이용해서 하라는 블로거들도 있었지만, 아무리해도(뭔가 Spring버전과 맞지 않는듯 했다.) 잘 안되어 아래와 같은 dependency를 주었다.
```xml
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>2.5.1</version>
</dependency>
```

## Controller
아래와같이 `@ResponseBody` 어노테이션을 설정해주고 리턴은 해당 모델을 넘기면 된다.
```java
@RequestMapping(value="/test")
@ResponseBody
public Map<String, Object> test(){
    	Map<String, Object> map = new HashMap<String, Object>();
    	map.put("1", "111");
    	map.put("2", 222);
    	return map;
    }
```
그리고 호출을 해보면 기대했던것처럼 이쁘게 json형태로 나온다.
```json
{
"1": "111",
"2": 222
}
```
물론, list 나 array, 일반 객체도 가능하다.

## 정리
삽질을 끝에 알게된 사실(?)을 정리해보자.
다른측면에서 분석을 해보면. `@ResponseBody`을 이용하여 view 에 json 형태로 나타내고자 할 경우 가능한 상황은 `toString`으로 했을때 json형태로 나올수 있으면 가능하다. 예로들어 아래처럼 클래스에 Lombok 어노테이션인 `@Data`가 붙으면 자동으로 `toString`을 오버라이딩 해주기 때문에 해당 클래스를 리턴하게되면 자동으로 json 처리가 된다.
 ```java
 @Data
 public Student{
   private String id;
   private String name;
   ...
 }
 ```
 `@ResponseBody`을 붙이고 `List<Student>`를 리턴하게 되면 에러가 나는데, 이럴경우 별도 라이브러리를 추가해줘야 자동으로 변환되어 json 형태로 나올수 있게 된다. (list.toString을 하면 json형태가 아닌 이상한 문자형태로 나오기 때문... Map같은것도 마찬가지 이유로 별도 라이브러리를 추가해줘야 정상적으로 나온다.)

## 마치며
단순히 **@ResponseBody를 사용해서 json으로 리턴하려면 어떤 라이브러리를 추가해야한다** 로 생각했던것에서, 이것저것 테스트 한 결과 **toString을 할수 있어야 하고 그 값이 json형태이면 가능하다** 로 결론이 지어졌다. 확실히 장님 코끼리 만지듯이 '그런가보다'하고 넘어가면 삽질이 진짜 불필요한 삽질이 되는것 같다. 구글링을 해보고, 테스트를 해봐서, 결론적으로 `내것`으로 만드는 습관을 가져야 겠다.
