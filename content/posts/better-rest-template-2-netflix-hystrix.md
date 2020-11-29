---
title: 조금 더 괜찮은 Rest Template 2부 - Circuit-breaker
date: 2020-03-29 23:09:16
categories:
  - tech
tags: 
  - spring boot
  - circuit breaker
  - Netflix
  - Hystrix
  - archives-2020
url : /2020/03/29/better-rest-template-2-netflix-hystrix/
featuredImage: /images/better-rest-template-2-netflix-hystrix/netflix_hystrix.jpg
images :
  - /images/better-rest-template-2-netflix-hystrix/netflix_hystrix.jpg
---

[지난 포스팅](/2020/03/22/better-rest-template-1-retryable/)에서는 Retryable 를 활용해서 간헐적인 네트워크 오류를 "재시도"를 함으로써 아주 간단하면서도 강력하게 해결할 수 있는 방법에 대해 알아보았다. 실제로 필자가 운영하는 서비스 에서도 Retryable 를 이용하기 전과 후를 비교해보면 간헐적인 네트워크 오류의 빈도수가 확실히 줄어든것을 확인할 수 있었다. <!--more -->

이렇게 "재시도"를 해서 요청했을때 성공 응답을 받을 경우엔 문제가 안되지만 네트워크 오류가 아닌 실제로 호출을 받는 해당 서버에서 문제가 발생했다면 어떨까? 예컨대, 해당 서버에서 DB를 조회하는 API를 호출한다고 가정했을때 DB 자체에서 어떠한 오류가 난다면. 이런 경우는 단순히 "재시도"로 해결할 수 없는 문제다.

물론 Retryable 의 Recover 어노테이션을 활용했기 때문에 클라이언트 즉, 사용자에게는 오류응답이 발생을 안했겠지만 호출 받는 서버 자체에서의 에러가 발생하는데 이런식의 재시도를 계속 시도한다면 호출 받는 서버 입장에서는 이 "재시도" request 또한 "부하" 로 받게 되고 결국 2차, 3차 장애가 이어질 수 밖에 없다.

기존 한덩어리로 관리되던 Monolithic Architecture 에서는 자체적으로 관리하기 때문에 이러한 에러 컨트롤 또한 자체적으로 관리를 할 수 있지만, 모듈이 모듈을 호출하게 되는 Microservice Architecture 로 바뀌다보니 이런 "연쇄 장애(?)" 같은 현상이 발생하게 되는 경우가 있다. 호출을 받는 서버의 상태가 이상하면 (에러응답이 지정한 임계치를 벗어나는 수준으로 맞춰서 발생한다면) 적절하게 호출을 하지 않고 (2차 장애를 내지 않도록 호출 자체를 하지 않고) 어느정도 기다리다 클라이언트에게는 에러응답이 아닌 미리 정해둔 응답을 내려주고, 에러가 복구되면 다시 호출하도록 하는 "무언가" 가 필요하지 않을까?

{{< image src="/images/better-rest-template-2-netflix-hystrix/domino.gif" caption="연쇄 장애. 제발 멈춰... <br> 출처 : http://dpg.danawa.com/mobile/community/view?boardSeq=175&listSeq=4066389" width="30%" >}}

지난 포스팅에 이어 이번 포스팅 에서는 그 "무언가". 즉, Circuit-breaker 에 대해 알아보고 직접 구현 및 테스트 하면서 돌아가는 원리에 대해 이해 해보고자 한다. 막상 개념은 머릿속에 있지만 직접 구현해보지 않으면 내것이 아니기에, 직접 구현하고 설정값들을 바꿔가면서 언젠가 필요한 순간에 꺼내서 사용할 수 있는 나만의 "무기" 를 만들어 보고자 한다.

## Circuit breaker ?
(한국 발음으로) 서킷브레이커를 검색해보면 주식시장 관련된 내용이 꽤 나온다. (앗, 잠깐 눈물좀...) [서킷 브레이커](http://bitly.kr/kSm6Y). 이 용어는 다양한 곳에서 사용되는데 "회로 차단기" 라고도 검색이 된다. 해당 내용을 발췌해보면 다음과 같다.
> 회로 차단기는 전기 회로에서 과부하가 걸리거나 단락으로 인한 피해를 막기 위해 자동으로 회로를 정지시키는 장치이다. 과부하 차단기와 누전 차단기로 나뉜다. 퓨즈와 다른 점은, 차단기는 어느 정도 시간이 지난 뒤, 원래의 기능이 동작하도록 복귀된다.

여기서 가장 중요한 문장은 "피해를 막기 위해 자동으로 회로를 정지시키는", "어느정도 시간이 지난뒤 원래의 기능이 동작하도록 복귀된다" 이 부분이 가장 중요한 것 같다. 시스템 구성이 점점 Microservice Architecture 로 바뀌어 가는 시점에서 이러한 "서킷브레이커"는 자동으로 모듈간의 호출 에러를 감지하고 위에서 말한 "연쇄 장애"를 사전에 막을 수 있는 아주 중요한 기능이라 생각된다.

"circuit breaker spring" 이라는 키워드로 검색해보면 이러한 고민을 이미 Netflix 라는 회사에서 [Hystrix](https://github.com/Netflix/Hystrix) 라는 이름으로 개발이 된것을 알 수 있다. 이 core 모듈을 Spring 에서 한번 더 감싸서 Spring Boot 에서 사용하기 좋게 spring-cloud-starter-netflix-hystrix 라는 이름으로 만들어 둔 것이 있는데 이것을 활용해 보기로 하자. 

## 구현
늘 그랬듯이 SpringBoot 프로젝트를 만들고 테스트할 Controller 를 만들어 주자. 원래대로라면 호출을 하는 모듈과 호출을 받는 모듈, 2개의 모듈을 만들어서 테스트 해야 하지만 편의를 위해 하나의 모듈에서 두개의 Controller 을 만들고 테스트 해보는 것으로 하자.
```java
@RestController
public class MainController {

	private final MainService mainService;

	@GetMapping("index")
	public String index(String key){
		return mainService.getResult(key);
	}

	public MainController(MainService mainService) {
		this.mainService = mainService;
	}
}

@Slf4j
@Service
public class MainService {

	private RestTemplate restTemplate;

	public String getResult(String key) {
		return restTemplate.getForObject("http://localhost:8080/target?key=" + key, String.class);
	}

	public MainService(RestTemplate restTemplate) {
		this.restTemplate = restTemplate;
	}
}


@Slf4j
@RestController
public class TargetContoller {

	@GetMapping("/target")
	public String target(String key) {
		log.info("input key : {}", key);
		if (!StringUtils.equals("taetaetae", key)) {
			throw new RuntimeException("Invalid key");
		}
		return "target";
	}
}

```

사실 설명할 부분도 없긴 하지만 그래도 적어보면, `/index`라는 주소와 `key`라는 파라미터로 요청을 하면 `/target`이라는 컨트롤러가 받아서 `key` 값에 따라 정상 응답을 줄지 아니면 에러를 응답하는 코드이다. 목표로 하는건, 파라미터를 일부러 잘못줘서 에러를 발생하는데 Hystrix를 적용해서 설정한 기준에 따라 응답은 미리 정해둔 응답을, `TargetContoller`에서는 잠시동안 요청을 받지 않다가 조금있다가 다시 호출하면 요청을 받는 시나리오로 작성해 보려 한다.
우선 필요한 dependency를 추가해준다.
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
	<version>2.2.2.RELEASE</version>
</dependency>

```

처음에 한참 했갈렸는데 "spring-cloud-starter-hystrix"가 아니고 "spring-cloud-starter-netflix-hystrix"를 적용해야 이상없이 돌아간다. (참고 : https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-hystrix) 그 다음 실제 적용할 코드에 Hystrix 설정을 적용해 주자.

```java
@EnableCircuitBreaker // 1
@SpringBootApplication
public class HystrixApplication {
	public static void main(String[] args) {
		SpringApplication.run(HystrixApplication.class, args);
	}
}
```
1. SpringBoot를 사용하면서 비슷하게 적용되는 EnableXXX 어노테이션, 이번에도 @EnableCircuitBreaker 어노테이션을 적용해 주면서 CircuitBreaker 를 사용하겠다고 지정해 주자.

```java
@Service
public class MainService {

	private RestTemplate restTemplate;

	@HystrixCommand( // 1
		fallbackMethod = "fallbackMethod", // 2
		commandProperties = {
			@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "500"), // 3
			@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "2"), // 4
			@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "2000"), // 5
			@HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "30") // 6
		})
	public String getResult(String key) {
		return restTemplate.getForObject("http://localhost:8080/target?key=" + key, String.class);
	}

	public String fallbackMethod(String key) {
		log.info("fallbackMethod, param : {}", key);
		return "defaultResult";
	}

	public MainService(RestTemplate restTemplate) {
		this.restTemplate = restTemplate;
	}
}
```
1. [스프링 가이드](https://spring.io/guides/gs/circuit-breaker/)에 따르면 `@Component` 나 `@Service` 하위에서 `@HystrixCommand`어노테이션을 찾아 작동한다고 나와있다. 때문에 우리가 적용할 "MainService.getResult()"에 지정해 주도록 하자.
  > Spring Cloud Netflix Hystrix looks for any method annotated with the @HystrixCommand annotation and wraps that method in a proxy connected to a circuit breaker so that Hystrix can monitor it. This currently works only in a class marked with @Component or @Service. 

2. fallbackMethod 을 이용하여 호출했을때 에러가 발생하면 실행할 메소드를 지정해주자. 즉, 설정한 기준에 도달하여 에러응답 대신 지정한 응답을 내려줄 경우 해당 메소드를 사용하게 된다.

3. 타임아웃을 지정해 준다. 여기서는 500ms 안에 응답이 없을 경우 에러라고 판단.

4. 서킷브레이커가 open 되는 최소 단위. 필자는 테스트를 위해 극단적(2)으로 지정하였지만 상황에 따라 해당 값을 커스터마징 할 필요가 있다.

5. 서킷브레이커가 지속되는 시간. 2초동안 지속되도록 한다.

6. 서킷브레이커가 open 되는 에러율. 다른값들과 우선순위를 잘 따져봐야 한다. 테스트를 위해 무조건 한번이라도 에러 발생할 경우 open 해도록 지정해두었다.

이 외에도 정말 [다양한 설정값](https://github.com/Netflix/Hystrix/wiki/configuration)들이 있는데 기본 설정값으로 했을때 시스템 성격에 맞지 않는다면 커스터마이징을 통해 현 시스템에 가장 적절한 값을 찾아야 할 것이다.

## 테스트
자. 뭔가 여러가지 설정을 했는데 이제 테스트를 해보자. 테스트를 하면서 설정값들을 이해하는게 가장 빠르다. 자세한 로그를 확인하기 위해 편의상 application.properties 에 `debug=true` 설정을 해두었다.

우선 정상응답을 얻기 위해 "/index?key=taetaetae" 라고 호출해보면 response 또한 "ok" 라고 나오는 것을 알 수 있다.
```shell
2020-03-30 01:55:29.866 DEBUG 13496 --- [nio-8080-exec-9] o.s.web.servlet.DispatcherServlet        : GET "/index?key=taetaetae", parameters={masked}
2020-03-30 01:55:29.871 DEBUG 13496 --- [nio-8080-exec-9] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to com.taetaetae.hystrix.main.MainController#index(String)
2020-03-30 01:55:30.057 DEBUG 13496 --- [x-MainService-1] o.s.web.client.RestTemplate              : HTTP GET http://localhost:8080/target?key=taetaetae
2020-03-30 01:55:30.059 DEBUG 13496 --- [x-MainService-1] o.s.web.client.RestTemplate              : Accept=[text/plain, application/json, application/*+json, */*]
2020-03-30 01:55:30.065 DEBUG 13496 --- [io-8080-exec-10] o.s.web.servlet.DispatcherServlet        : GET "/target?key=taetaetae", parameters={masked}
2020-03-30 01:55:30.065 DEBUG 13496 --- [io-8080-exec-10] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to com.taetaetae.hystrix.target.TargetContoller#target(String)
2020-03-30 01:55:30.066  INFO 13496 --- [io-8080-exec-10] c.t.hystrix.target.TargetContoller       : input key : taetaetae
2020-03-30 01:55:30.073 DEBUG 13496 --- [io-8080-exec-10] m.m.a.RequestResponseBodyMethodProcessor : Using 'text/plain', given [text/plain, application/json, application/*+json, */*] and supported [text/plain, */*, text/plain, */*, application/json, application/*+json, application/json, application/*+json]
2020-03-30 01:55:30.074 DEBUG 13496 --- [io-8080-exec-10] m.m.a.RequestResponseBodyMethodProcessor : Writing ["ok"]
2020-03-30 01:55:30.077 DEBUG 13496 --- [io-8080-exec-10] o.s.web.servlet.DispatcherServlet        : Completed 200 OK
2020-03-30 01:55:30.079 DEBUG 13496 --- [x-MainService-1] o.s.web.client.RestTemplate              : Response 200 OK
2020-03-30 01:55:30.080 DEBUG 13496 --- [x-MainService-1] o.s.web.client.RestTemplate              : Reading to [java.lang.String] as "text/plain;charset=UTF-8"
2020-03-30 01:55:30.087 DEBUG 13496 --- [nio-8080-exec-9] m.m.a.RequestResponseBodyMethodProcessor : Using 'text/html', given [text/html, application/xhtml+xml, image/webp, image/apng, application/xml;q=0.9, application/signed-exchange;v=b3;q=0.9, */*;q=0.8] and supported [text/plain, */*, text/plain, */*, application/json, application/*+json, application/json, application/*+json]
2020-03-30 01:55:30.087 DEBUG 13496 --- [nio-8080-exec-9] m.m.a.RequestResponseBodyMethodProcessor : Writing ["ok"]
2020-03-30 01:55:30.089 DEBUG 13496 --- [nio-8080-exec-9] o.s.web.servlet.DispatcherServlet        : Completed 200 OK
```

자 이제 실패가 되도록 잘못된 파라미터를 호출하게 되면 어떻게 될까? "/index?key=taekwan"
```shell
2020-03-30 01:57:48.025 DEBUG 19084 --- [nio-8080-exec-9] o.s.web.servlet.DispatcherServlet        : GET "/index?key=taekwan", parameters={masked}
2020-03-30 01:57:48.031 DEBUG 19084 --- [nio-8080-exec-9] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to com.taetaetae.hystrix.main.MainController#index(String)
2020-03-30 01:57:48.222 DEBUG 19084 --- [x-MainService-1] o.s.web.client.RestTemplate              : HTTP GET http://localhost:8080/target?key=taekwan
2020-03-30 01:57:48.224 DEBUG 19084 --- [x-MainService-1] o.s.web.client.RestTemplate              : Accept=[text/plain, application/json, application/*+json, */*]
2020-03-30 01:57:48.229 DEBUG 19084 --- [io-8080-exec-10] o.s.web.servlet.DispatcherServlet        : GET "/target?key=taekwan", parameters={masked}
2020-03-30 01:57:48.229 DEBUG 19084 --- [io-8080-exec-10] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to com.taetaetae.hystrix.target.TargetContoller#target(String)
2020-03-30 01:57:48.230  INFO 19084 --- [io-8080-exec-10] c.t.hystrix.target.TargetContoller       : input key : taekwan
2020-03-30 01:57:48.237 DEBUG 19084 --- [io-8080-exec-10] o.s.web.servlet.DispatcherServlet        : Failed to complete request: java.lang.RuntimeException: Invalid key
2020-03-30 01:57:48.244 ERROR 19084 --- [io-8080-exec-10] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.RuntimeException: Invalid key] with root cause

java.lang.RuntimeException: Invalid key
	at com.taetaetae.hystrix.target.TargetContoller.target(TargetContoller.java:19) ~[classes/:na]
	... 중략

2020-03-30 01:57:48.247 DEBUG 19084 --- [io-8080-exec-10] o.s.web.servlet.DispatcherServlet        : "ERROR" dispatch for GET "/error?key=taekwan", parameters={masked}
2020-03-30 01:57:48.250 DEBUG 19084 --- [io-8080-exec-10] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController#error(HttpServletRequest)
2020-03-30 01:57:48.256 DEBUG 19084 --- [io-8080-exec-10] o.s.w.s.m.m.a.HttpEntityMethodProcessor  : Using 'application/json', given [text/plain, application/json, application/*+json, */*] and supported [application/json, application/*+json, application/json, application/*+json]
2020-03-30 01:57:48.257 DEBUG 19084 --- [io-8080-exec-10] o.s.w.s.m.m.a.HttpEntityMethodProcessor  : Writing [{timestamp=Mon Mar 30 01:57:48 KST 2020, status=500, error=Internal Server Error, message=Invalid ke (truncated)...]
2020-03-30 01:57:48.267 DEBUG 19084 --- [io-8080-exec-10] o.s.web.servlet.DispatcherServlet        : Exiting from "ERROR" dispatch, status 500
2020-03-30 01:57:48.267 DEBUG 19084 --- [x-MainService-1] o.s.web.client.RestTemplate              : Response 500 INTERNAL_SERVER_ERROR
2020-03-30 01:57:48.277  INFO 19084 --- [x-MainService-1] com.taetaetae.hystrix.main.MainService   : fallbackMethod, param : taekwan
2020-03-30 01:57:48.282 DEBUG 19084 --- [nio-8080-exec-9] m.m.a.RequestResponseBodyMethodProcessor : Using 'text/html', given [text/html, application/xhtml+xml, image/webp, image/apng, application/xml;q=0.9, application/signed-exchange;v=b3;q=0.9, */*;q=0.8] and supported [text/plain, */*, text/plain, */*, application/json, application/*+json, application/json, application/*+json]
2020-03-30 01:57:48.282 DEBUG 19084 --- [nio-8080-exec-9] m.m.a.RequestResponseBodyMethodProcessor : Writing ["defaultResult"]
2020-03-30 01:57:48.283 DEBUG 19084 --- [nio-8080-exec-9] o.s.web.servlet.DispatcherServlet        : Completed 200 OK
```

(로그가 길지만...) 위 내용을 보면 fallbackMethod 가 호출된 것을 확인할 수 있고, 응답 또한 200 OK 에 미리 지정해 둔 "defaultResult"를 내려준 것을 확인할 수 있다. 하지만 `TargetContoller` 에서 요청이 들어오면 로깅하려고 한 부분이 찍힌것을 보면, 에러 응답에 대한 fallbackMethod 는 호출 되었지만 실제로 서킷브레이커는 open 이 안된것을 확인할 수 있다. 그럼 좀더 빠른 시간내에 여러번 호출해보면 어떻게 될까??

```shell
2020-03-30 02:03:28.040 DEBUG 19084 --- [nio-8080-exec-5] o.s.web.servlet.DispatcherServlet        : GET "/index?key=taekwan", parameters={masked}
2020-03-30 02:03:28.040 DEBUG 19084 --- [nio-8080-exec-5] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to com.taetaetae.hystrix.main.MainController#index(String)
2020-03-30 02:03:28.042 DEBUG 19084 --- [x-MainService-5] o.s.web.client.RestTemplate              : HTTP GET http://localhost:8080/target?key=taekwan
2020-03-30 02:03:28.042 DEBUG 19084 --- [x-MainService-5] o.s.web.client.RestTemplate              : Accept=[text/plain, application/json, application/*+json, */*]
2020-03-30 02:03:28.046 DEBUG 19084 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : GET "/target?key=taekwan", parameters={masked}
2020-03-30 02:03:28.047 DEBUG 19084 --- [nio-8080-exec-1] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to com.taetaetae.hystrix.target.TargetContoller#target(String)
2020-03-30 02:03:28.047  INFO 19084 --- [nio-8080-exec-1] c.t.hystrix.target.TargetContoller       : input key : taekwan
2020-03-30 02:03:28.047 DEBUG 19084 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Failed to complete request: java.lang.RuntimeException: Invalid key
2020-03-30 02:03:28.048 ERROR 19084 --- [nio-8080-exec-1] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.RuntimeException: Invalid key] with root cause

java.lang.RuntimeException: Invalid key
	at com.taetaetae.hystrix.target.TargetContoller.target(TargetContoller.java:19) ~[classes/:na]
	... 중략	

2020-03-30 02:03:28.048 DEBUG 19084 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : "ERROR" dispatch for GET "/error?key=taekwan", parameters={masked}
2020-03-30 02:03:28.048 DEBUG 19084 --- [nio-8080-exec-1] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController#error(HttpServletRequest)
2020-03-30 02:03:28.049 DEBUG 19084 --- [nio-8080-exec-1] o.s.w.s.m.m.a.HttpEntityMethodProcessor  : Using 'application/json', given [text/plain, application/json, application/*+json, */*] and supported [application/json, application/*+json, application/json, application/*+json]
2020-03-30 02:03:28.049 DEBUG 19084 --- [nio-8080-exec-1] o.s.w.s.m.m.a.HttpEntityMethodProcessor  : Writing [{timestamp=Mon Mar 30 02:03:28 KST 2020, status=500, error=Internal Server Error, message=Invalid ke (truncated)...]
2020-03-30 02:03:28.049 DEBUG 19084 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Exiting from "ERROR" dispatch, status 500
2020-03-30 02:03:28.049 DEBUG 19084 --- [x-MainService-5] o.s.web.client.RestTemplate              : Response 500 INTERNAL_SERVER_ERROR
2020-03-30 02:03:28.050  INFO 19084 --- [x-MainService-5] com.taetaetae.hystrix.main.MainService   : fallbackMethod, param : taekwan
2020-03-30 02:03:28.051 DEBUG 19084 --- [nio-8080-exec-5] m.m.a.RequestResponseBodyMethodProcessor : Using 'text/html', given [text/html, application/xhtml+xml, image/webp, image/apng, application/xml;q=0.9, application/signed-exchange;v=b3;q=0.9, */*;q=0.8] and supported [text/plain, */*, text/plain, */*, application/json, application/*+json, application/json, application/*+json]
2020-03-30 02:03:28.051 DEBUG 19084 --- [nio-8080-exec-5] m.m.a.RequestResponseBodyMethodProcessor : Writing ["defaultResult"]
2020-03-30 02:03:28.052 DEBUG 19084 --- [nio-8080-exec-5] o.s.web.servlet.DispatcherServlet        : Completed 200 OK


2020-03-30 02:03:28.679 DEBUG 19084 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : GET "/index?key=taekwan", parameters={masked}
2020-03-30 02:03:28.679 DEBUG 19084 --- [nio-8080-exec-1] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to com.taetaetae.hystrix.main.MainController#index(String)
2020-03-30 02:03:28.680  INFO 19084 --- [nio-8080-exec-1] com.taetaetae.hystrix.main.MainService   : fallbackMethod, param : taekwan
2020-03-30 02:03:28.681 DEBUG 19084 --- [nio-8080-exec-1] m.m.a.RequestResponseBodyMethodProcessor : Using 'text/html', given [text/html, application/xhtml+xml, image/webp, image/apng, application/xml;q=0.9, application/signed-exchange;v=b3;q=0.9, */*;q=0.8] and supported [text/plain, */*, text/plain, */*, application/json, application/*+json, application/json, application/*+json]
2020-03-30 02:03:28.681 DEBUG 19084 --- [nio-8080-exec-1] m.m.a.RequestResponseBodyMethodProcessor : Writing ["defaultResult"]
2020-03-30 02:03:28.682 DEBUG 19084 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed 200 OK


2020-03-30 02:03:29.327 DEBUG 19084 --- [nio-8080-exec-5] o.s.web.servlet.DispatcherServlet        : GET "/index?key=taekwan", parameters={masked}
2020-03-30 02:03:29.327 DEBUG 19084 --- [nio-8080-exec-5] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to com.taetaetae.hystrix.main.MainController#index(String)
2020-03-30 02:03:29.328  INFO 19084 --- [nio-8080-exec-5] com.taetaetae.hystrix.main.MainService   : fallbackMethod, param : taekwan
2020-03-30 02:03:29.328 DEBUG 19084 --- [nio-8080-exec-5] m.m.a.RequestResponseBodyMethodProcessor : Using 'text/html', given [text/html, application/xhtml+xml, image/webp, image/apng, application/xml;q=0.9, application/signed-exchange;v=b3;q=0.9, */*;q=0.8] and supported [text/plain, */*, text/plain, */*, application/json, application/*+json, application/json, application/*+json]
2020-03-30 02:03:29.328 DEBUG 19084 --- [nio-8080-exec-5] m.m.a.RequestResponseBodyMethodProcessor : Writing ["defaultResult"]
2020-03-30 02:03:29.329 DEBUG 19084 --- [nio-8080-exec-5] o.s.web.servlet.DispatcherServlet        : Completed 200 OK


2020-03-30 02:03:30.071 DEBUG 19084 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : GET "/index?key=taekwan", parameters={masked}
2020-03-30 02:03:30.072 DEBUG 19084 --- [nio-8080-exec-1] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to com.taetaetae.hystrix.main.MainController#index(String)
2020-03-30 02:03:30.072  INFO 19084 --- [nio-8080-exec-1] com.taetaetae.hystrix.main.MainService   : fallbackMethod, param : taekwan
2020-03-30 02:03:30.073 DEBUG 19084 --- [nio-8080-exec-1] m.m.a.RequestResponseBodyMethodProcessor : Using 'text/html', given [text/html, application/xhtml+xml, image/webp, image/apng, application/xml;q=0.9, application/signed-exchange;v=b3;q=0.9, */*;q=0.8] and supported [text/plain, */*, text/plain, */*, application/json, application/*+json, application/json, application/*+json]
2020-03-30 02:03:30.073 DEBUG 19084 --- [nio-8080-exec-1] m.m.a.RequestResponseBodyMethodProcessor : Writing ["defaultResult"]
2020-03-30 02:03:30.073 DEBUG 19084 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed 200 OK
```

로그를 자세히 보면, 처음엔 `TargetContoller`에서 찍은 로그가 있으면서 fallbackMethod 의 로그가 있지만 그 뒤에는 `TargetContoller`에서 찍은 로그는 없고 fallbackMethod 의 로그만 있는 것을 확인할 수 있다. 즉, 위에서 설정한 타임아웃 500ms 안에 2번 에러가 발생하였기 때문에 서킷브레이커가 동작해서 `TargetContoller` 로의 요청을 하지 않았기 때문이다. 이렇게 되면 `TargetContoller`입장에서는 연속적으로 요청이 오지 않기 때문에 부담이 줄어드는 효과가 있다. 

## 모니터링
서비스를 운영하면서 서킷브레이커가 동작되었는지를 확인하기 위해서 위에서 했던것 처럼 곳곳에 심어둔 로그를 눈이 빠져라 보고 있어야만 할까? Netflix 의 Hystrix 는 아주 우아하게 이를 웹에서 모니터링 할 수 있게도 구현해놨다. (참 대단하다...) 그럼 모니터링을 해보자.
모니터링을 하기 위해서는 dependency 를 추가해야 한다.
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
	<version>2.2.2.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

SpringBoot에서 제공하는 actuator 를 활용하여 정보를 받도록 해주고, application.properties 설정에서도 Hystrix 관련된 정보를 받아볼 수 있게 아래처럼 추가해주자.
```shell
management.endpoints.web.exposure.include=hystrix.stream
```

그럼 끝이다. (응? 이게 끝이라고?) 그렇다. 필자도 아주 놀랬는데 생각해보니 모니터링을 위해 별도의 코드가 들어가는 자체가 이상한 부분같다. 아주 심플하게 모듈과 설정만 추가해주면 모니터링이 가능하다. 그럼 진짜 모니터링을 해보자. 일부러 여러번 호출해서 서킷브레이커를 발동시키면 모니터링 페이지에서는 어떤식으로 나오는지 확인해보자. 우선 이렇게 설정한뒤 실행을 하면 `/hystrix`로 접근이 가능하고 귀엽지만 뭔가 놀랜 표정의 곰돌이가 반기는 것을 확인할 수 있다. 그다음 url 에 `http://localhost:8080/actuator/hystrix.stream` 를 입력후 "Monitor Stream"을 클릭하면 아래와 같은 화면을 볼 수 있다.

{{< image src="/images/better-rest-template-2-netflix-hystrix/hystrix.jpg" caption="곰돌이가 조금 화난것 같다. (코딩좀 잘하라고?)" width="80%" >}}

뭔가 로딩중인것 같은데 올바른 요청 `/index?key=taetaetae` 을 해보면 그래프가 바뀌고 잘못된 요청 `/index?key=taekwan` 을 여러번 해보다가 잠시 멈추고 를 반복해보면 서킷브레이커가 open 되었다가 다시 close 된것을 확인할 수 있다.

{{< image src="/images/better-rest-template-2-netflix-hystrix/monitor_execute.gif" caption="에러가 발생하면 open, 설정한 시간이 지나고 정상 응답이 되면 close" width="80%" >}}

##  마치며
물론 위에서 설정한 내용은 서비스에 적용하기도 부끄러울만큼 아주 극단적이고 다양한 상황이 전혀 고려되지 않은 설정값이다. 현 시스템에 맞춰 설정값을 커스터마이징 하며 최적의 설정값을 찾아야 할 것이다. 또한 무조건 "설정한 값에 의해서 서킷브레이커가 잘 동작 하겠지" 라고 믿는것(?)보다 모니터링을 추가로 설정해서 실제로 언제 서킷브레이커가 작동을 하는지 확인을 해봐야 할 것 같다. 나아가서 이 모니터링 페이지가 있다고 해서 주식 차트 보는것 마냥 계속 보고 있을수는 없는일. 모니터링 페이지를 봐야할 시점이 오게 된다면 자동으로 알림을 받도록 해서 필요할 때만 모니터링을 할 수 있도록 해야 할 것 같다. (이상하게 주식으로 시작해서 주식으로 끝나는 것 같은 느낌은 뭐지...)

물론 이번에도 위에서 사용한 코드는 [필자의 Github Repo](https://github.com/taetaetae/hystrix)에서 확인이 가능하다.

참고 url
- https://github.com/Netflix/Hystrix
- https://spring.io/guides/gs/circuit-breaker/
- https://woowabros.github.io/experience/2017/08/21/hystrix-tunning.html