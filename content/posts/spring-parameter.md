---
title: 스프링환경에서의 파라미터 관련 정리
date: 2017-03-12 18:03:01
categories: [tech]
tags:
  - spring
  - ReqeustParam
  - RequestBody
  - ModelAttribute
  - spring command 객체
  - archives-2017
url : /2017/03/12/spring-parameter/
featuredImage: /images/spring-parameter/spring-parameter-1.jpg
images :
  - /images/spring-parameter/spring-parameter-1.jpg

---
일반적인 웹 프로젝트 구성에서는 `Controller`레벨에서 응답을 받고 비지니스 로직을 처리 후에 다시 `View`레벨로 넘어가는게 통상적인 흐름이다. 이 부분에서 파라미터 관련한 여러가지 부분에 대해 정리해보고자 한다.
<!-- more -->
## httpServletRequest.getParameter()
아래소스처럼 `HttpServletRequest`의 getParameter() 메서드를 이용하여 파라미터값을 가져올 수 있다.
```java
@RequestMapping("/")
public String home(HttpServletRequest httpServletRequest) {
    String id = httpServletRequest.getParameter("id");

    return "home";
}
```

## @RequestParam
또다른 방법으로는 `@RequestParam` 어노테이션을 이용하면 간단하게 파라미터값을 가져올수 있다. 우선, 해당 어노테이션의 옵션값들에 대해 간략하게 확인하고 넘어가는게 좋을듯 싶다. [API문서 4.3.6 기준](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestParam.html)

| 이름 | 타입 | 설명 |
| --- | --- | --- |
| name, value (Alias for name) | String | 파라미터 이름 |
| required | boolean | 해당 파라미터가 반드시 필수인지 여부, 기본값은 true |
| defaultValue | String | 해당 파라미터의 기본값 |

위 옵션값들을 조합하여 컨트롤러 메소드에 적용해보면 아래 소스와 같이 만들어지고, 이렇게 reqeust에서 파라미터값을 가져올수 있다.
```java
@RequestMapping("/")
public String home(@RequestParam(value="id", defaultValue="false") String id) {
    return "home";
}
```
이 어노테이션을 이용하게되면 자칫 잘못하다간 에러를 만날수가 있는데 `required`값을 true로 해놓고 (필수 파라미터 설정) 해당 파라미터를 사용하지 않고 요청을 보내게 되면 HTTP 400 에러를 받게 되니 각 옵션들을 정확히 확인하고 사용해야 할 것 같다.
물론 컨트롤러의 메소드에서 해당 어노테이션을 사용하지 않고도 아래 코드처럼 바로 받을수 있다. 이렇게 바로 받을 경우는 필수 파라미터값이 false로 설정이 되고 변수명과 동일한 파라미터만 받을수 있게 되며 기본값 설정을 할수는 없다. 방법의 차이라서 상황에 따라 맞춰 사용하면 될듯 하다.
```java
@RequestMapping("/")
public String home(String id) {
    return "home";
}
```

## @RequestBody
`@RequestBody`어노테이션을 사용할 경우 반드시 POST형식으로 응답을 받는 구조여야만 한다. 이를테면 JSON 이나 XML같은 데이터를 적절한 messageConverter로 읽을때 사용하거나, POJO형태의 데이터 전체로 받을경우에 사용된다. 단, 이 어노테이션을 사용하여 파라미터를 받을 경우 별도의 추가 설정(POJO 의 get/set 이나 json/xml 등의 messageConverter 등)을 해줘야 적절하게 데이터를 받을수가 있다.
```java
@PostMapping("/")
public String home(@ReqeustBody Student student) {
    return "home";
}
```

## @ModelAttribute
`@RequestParam`과 비슷한데 1:1로 파라미터를 받을경우는 `@RequestParam`를 사용하고, 도메인이나 오브젝트로 파라미터를 받을 경우는 `@ModelAttribute`으로 받을수 있다. 또한 이 어노테이션을 사용하면 검증(Validation)작업을 추가로 할수 있는데 예로들어 null이라던지, 각 멤버변수마다 valid옵션을 줄수가 있고 여기서 에러가 날 경우 BindException 이 발생한다.

## Spring command 객체
컨트롤러에서 파라미터로 받은 정보에 대해서는 view 에서 바로 사용이 가능하다. 예로 들어 아래그림처럼 이렇게 컨트롤러가 구성되어있고

{{< image src="/images/spring-parameter/spring-parameter-1.jpg" src_l="/images/spring-parameter/spring-parameter-1.jpg" >}}


이렇게 모델이 구성되어있을때 

{{< image src="/images/spring-parameter/spring-parameter-2.jpg" src_l="/images/spring-parameter/spring-parameter-2.jpg" >}}

view 에서 이런식으로 구성되어있다고 가정해보자. 

{{< image src="/images/spring-parameter/spring-parameter-3.jpg" src_l="/images/spring-parameter/spring-parameter-3.jpg" >}}

이때 `/student?name=taetaetae&age=32&address=green-factory`로 호출을 해보면 구지 Model에 값을 셋팅해주지 않아도 다음과 같이 정보를 읽을수 있게 된다. 

{{< image src="/images/spring-parameter/spring-parameter-4.jpg" src_l="/images/spring-parameter/spring-parameter-4.jpg" >}}


## 마치며
스프링에서 파라미터를 받는 방법은 상당히 다양하다. 이게 정답이다 정의할수 없을정도로. 상황에 따라 맞는 방법으로 파라미터를 받아야 하겠고, 각 방법에 장/단점을 최대한 살려서 좀더 깔끔한 코드를 작성할수 있어야 하겠다.
