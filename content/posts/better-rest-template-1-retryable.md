---
title: 조금 더 괜찮은 Rest Template 1부 - Retryable
date: 2020-03-22 15:30:35
categories:
  - tech
tags: 
  - spring boot
  - retryable
  - archives-2020
url : /2020/03/22/better-rest-template-1-retryable/
featuredImage: /images/better-rest-template-1-retryable/icons8.png
---

웹 어플리케이션을 만들면서 꼭 한번 쯤 만나게 되는 "RestTemplate". 접근 가능한 외부 HTTP URL(보통 API)을 호출하는 방법중에 하나로 springframework 에서 제공해주는 모듈이다. 특히 큰 한덩어리로 관리되던 Monolithic Architecture 에서 요청을 하고(client) 응답을 주는(server)  <!--more -->즉, Endpoint가 작은 단위로 분리되는 Microservice Architecture 로 바뀌면서 각 서비스간 호출방식이 HTTP 일 경우 자주 사용되곤 하는 것 같다. (webClient 등 다른 여러 호출 방법들이 있다.)
만약, 요청을 하는 클라이언트 입장에서 응답을 주는 서버의 상태가 불안정 하다고 가정했을때, 어떤식으로 처리해야 할까? 예컨대, 요청 10번에 한번은 어떠한 이슈로 응답이 지연되거나 서버에러가 발생한다고 하면 클라이언트를 사용하는 사용자 입장에서는 간헐적인 오류응답에 답답함을 호소할 수도 있다. 그럼 잠시 눈을 감고 생각해보자. 
가볍게 생각하면 아래처럼 아주 간단하게 "예외처리"를 이용할 수도 있다.
```java
try {
	// http call
} catch (Exception e){
	// 서버에러가 아닌 약속된 에러응답을 리턴
}
```

하지만 이것도 정답이 아닐수 있는게, "간헐적인 오류"로 인해 사용자는 오류화면을 봐야하기 때문에 클라이언트에 대한 신뢰를 저버릴 수밖에 없다. 그럼 어떻게 해야할까? 여러가지 해결방법이 있겠지만 간단하면서도 강력하다고 생각되는 방법이 바로 "재시도" 라고 생각한다. 클라이언트를 사용하는 사용자가 눈치 못챌만큼 빠르게 재시도를 한다면 에러가 나도 다시한번 호출해서 성공할 수 있는 가능성이 높기 때문이다. (그치만 근본적인 원인은 해결해야...)

{{< image src="/images/better-rest-template-1-retryable/retry.png" caption="실제로 조금있다 해보면 되는 경우가 많으니 안될때는 조금 (천천히) 시도해보자. <br>출처 : http://www.segye.com/newsView/20200302504384" width="40%" >}}

이번 포스팅에서는 RestTemplate 를 이용할때 "재시도" 할 수 있는 방법에 대해 알아보고자 한다. 아주 간단할지 모르지만 노력에 비해 효과가 상당하다고 생각하기 때문에 정리해 두고 싶었다.

## Spring Retry
[공식 Github](https://github.com/spring-projects/spring-retry)에 소개를 빌리자면, Spring 어플리케이션에 대한 재시도 지원을 제공한다고 한다. 위에서 이야기 했던 "RestTemplate"과는 사실 무관하고, 이를 활용해서 재시도 하는 "RetryRestTemplate"를 구현해보려 하는것이다. 우선 이 "Spring-Retry"의 예제를 보면 아주 심플하게 사용할 수 있다. 우선 pom에 구현에 필요한 dependency 를 추가하고 아래 코드를 보자.
```xml
<dependency>
	<groupId>org.springframework.retry</groupId>
	<artifactId>spring-retry</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

```java
@Configuration
@EnableRetry // 1
public class Application {

    @Bean
    public Service service() {
        return new Service();
    }

}

@Service
class Service {
    @Retryable(RemoteAccessException.class) // 2
    public void service() {
        // ... do something
    }
    @Recover // 3
    public void recover(RemoteAccessException e) {
       // ... panic
    }
}
```
1. @EnableRetry 어노테이션을 @Configuration을 지정한 클래스 중 하나에 추가한다.
2. 재시도 하려는 메소드에 @Retryable 어노테이션을 지정해준다.
3. 재시도가 완료되는 시점에서 실행하고 싶을때 선언하는 어노테이션, @Retryable 동일한 클래스에서 선언되어야 하고 return type 은 @Retryable을 지정한 메소드와 동일해야 한다.

## Retry Rest Template
이렇게 springframework 에서 제공해주는 spring-retry 를 이용해서 이번 포스팅의 목표인 재시도를 하는 Retry Rest Template 를 구성해보자. 우선, RestTemplate 를 Bean 으로 등록하고, 위에서 이야기 한 어노테이션들로 구성해보자.
```java
@EnableRetry
@Configuration
public class RetryableRestTemplateConfiguration {

	@Bean
	public RestTemplate retryableRestTemplate() {
		SimpleClientHttpRequestFactory clientHttpRequestFactory = new SimpleClientHttpRequestFactory(); // 1
		clientHttpRequestFactory.setReadTimeout(2000);
		clientHttpRequestFactory.setConnectTimeout(500);

		RestTemplate restTemplate = new RestTemplate(clientHttpRequestFactory) {
			@Override
			@Retryable(value = RestClientException.class, maxAttempts = 3, backoff = @Backoff(delay = 1000)) // 2
			public <T> ResponseEntity<T> exchange(URI url, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType)
				throws RestClientException {
				return super.exchange(url, method, requestEntity, responseType); 
			}

			@Recover
			public <T> ResponseEntity<String> exchangeRecover(RestClientException e) {
				return ResponseEntity.badRequest().body("bad request T.T"); // 3
			}
		};

		return restTemplate;
	}
}
```

1. SimpleClientHttpRequestFactory 를 만들고 각 타임아웃을 설정해준 다음 RestTemplate 파라미터로 넘겨준다.
2. 사용하는 곳에서 exchange 메소드를 이용할 것이므로 해당 메소드를 오버라이드 해준다. 먼저 해당 메소드에서 "RestClientException"이 발생할 경우 Retry 로직을 수행한다고 정해주고, 최대 시도는 3번, backoff 설정중 delay를 1000ms(1초)로 지정해서 재시도가 진행되도록 해준다.
3. 2 에서 지정한 재시도가 끝나면 (재시도를 전부 다 하면) 해당 메소드를 수행하게 되어있고, 임의로 응답에 지정한 문구를 넘겨준다.

이렇게 하고 실제로 사용하는 로직에서 일부러 잘못된 URL을 호출해 보도록 하자. 그리고서 로그를 자세히 보도록 application.properties 에 "debug=true" 설정을 해준다.
```java
@RestController
public class TestController {

	@Autowired
	private RestTemplate retryableRestTemplate;

	@GetMapping(value = "/employees", produces = MediaType.APPLICATION_JSON_VALUE)
	@ResponseBody
	public String employees() throws URISyntaxException {
		final String baseUrl = "http://dummy.restapiexample.com/api/v1/employeeszzz"; // zzz 가 빠져야 한다.
		URI uri = new URI(baseUrl);
		ResponseEntity<String> exchange = retryableRestTemplate.exchange(uri, HttpMethod.GET, null, String.class);
		return exchange.getBody();
	}
}
```

이렇게 하고 실행을 시켜보면 다음과 같이 재시도 관련 로깅이 찍히는 것을 볼 수 있고
```shell
23:05:50.893 DEBUG 21016 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : GET "/employees", parameters={}
23:05:50.898 DEBUG 21016 --- [nio-8080-exec-1] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to com.taetaetae.retryableresttemplate.TestController#employees()
23:05:50.994 DEBUG 21016 --- [nio-8080-exec-1] org.springframework.web.HttpLogging      : HTTP GET http://dummy.restapiexample.com/api/v1/employeeszzz
23:05:50.999 DEBUG 21016 --- [nio-8080-exec-1] org.springframework.web.HttpLogging      : Accept=[text/plain, application/json, application/*+json, */*]
23:05:51.861 DEBUG 21016 --- [nio-8080-exec-1] org.springframework.web.HttpLogging      : Response 404 NOT_FOUND
23:05:52.869 DEBUG 21016 --- [nio-8080-exec-1] org.springframework.web.HttpLogging      : HTTP GET http://dummy.restapiexample.com/api/v1/employeeszzz
23:05:52.869 DEBUG 21016 --- [nio-8080-exec-1] org.springframework.web.HttpLogging      : Accept=[text/plain, application/json, application/*+json, */*]
23:05:53.603 DEBUG 21016 --- [nio-8080-exec-1] org.springframework.web.HttpLogging      : Response 404 NOT_FOUND
23:05:54.605 DEBUG 21016 --- [nio-8080-exec-1] org.springframework.web.HttpLogging      : HTTP GET http://dummy.restapiexample.com/api/v1/employeeszzz
23:05:54.606 DEBUG 21016 --- [nio-8080-exec-1] org.springframework.web.HttpLogging      : Accept=[text/plain, application/json, application/*+json, */*]
23:05:55.305 DEBUG 21016 --- [nio-8080-exec-1] org.springframework.web.HttpLogging      : Response 404 NOT_FOUND
23:05:57.192 DEBUG 21016 --- [nio-8080-exec-1] m.m.a.RequestResponseBodyMethodProcessor : Using 'application/json;q=0.8', given [text/html, application/xhtml+xml, image/webp, image/apng, application/xml;q=0.9, application/signed-exchange;v=b3;q=0.9, */*;q=0.8] and supported [application/json]
23:05:57.193 DEBUG 21016 --- [nio-8080-exec-1] m.m.a.RequestResponseBodyMethodProcessor : Writing ["bad request T.T"]
23:05:57.202 DEBUG 21016 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed 200 OK
```

브라우저에서 해당 url을 접근해보면 @Recover 에서 지정했던 결과를 볼 수 있게 된다. 사실 이러한 방법에 대해 삽질하기 전에 다른 방법들을 찾아봤지만 restTemplate 을 사용하는 곳에서 각각 retry관련 로직을 추가해야 했기에 뭔가 어글리해 보여서 삽질을 시작하게 되었다. 다행히 성공.

{{< image src="/images/better-rest-template-1-retryable/response.jpg" caption="예제로 임의 문자열을 리턴했지만 상황에 따라 얼마든지 커스터마이징이 가능하다." width="50%" >}}

## 마치며
어떻게 보면 너무나 당연하게 "여러번 재시도 요청하면 되지?" 라는 말을 할 수 있지만, "입코딩" 하는 것과 실제로 코드를 구현하는건 다른 이야기인것 같다. 정말 작은 로직 추가로 꽤 큰 효과를 볼 수 있어 다행이라 생각한다.
이러한 "재시도" 말고도 요청하고자 하는 곳의 서버의 상태가 안좋을 때 서버에러가 아닌 다른 명시적인 에러를 반환할 수 있는 방법이 다양할 것 같다. 모든것엔 정답이 없는 것 처럼. 
제목에서 알 수 있듯이 다음 "2부" 에서는 "Retry"가 아닌 "Circuit Breaker"를 사용하여 "재시도"의 방법보다 조금 다른 측면에서 조금 더 괜찮은 방법으로 RestTemplate 를 사용해 보고자 한다.

위에서 사용한 코드는 [필자의 Github Repo](https://github.com/taetaetae/retryable-resttemplate)에서 확인이 가능하다.

참고 url
- https://dzone.com/articles/how-to-use-spring-retry
- https://www.baeldung.com/spring-retry
- https://github.com/spring-projects/spring-retry
- https://docs.spring.io/spring-batch/docs/current/reference/html/retry.html