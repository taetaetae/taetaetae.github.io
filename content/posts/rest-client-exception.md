---
title: RestClientException 처리
date: 2018-03-17 19:19:39
categories: [tech]
tags:
  - RestTemplate
  - RestClientException
  - archives-2018
url : /2018/03/17/rest-client-exception/

---
Spring 환경에서 (Spring5는 달라졌지만...) 외부 API로의 호출을 할때 자주 쓰이는 RestTemplate. 이때 request에 대해 정상적인 응답이 아닌 경우에 대한 처리는 어떻게 할까? 막연하게 생각을 해보면 Http Status Code를 받아서 판별을 하면 되지만 Http Status Code만 봐도 엄청~많다. <!-- more --> ( 위키피디아 참고 : https://en.wikipedia.org/wiki/List_of_HTTP_status_codes )
if~else 로 다 나눌수도 없고... 우선 성공/실패에 대한 판별은 어떻게 할까 하고 코드를 파고파고 들어가다가 알게된 부분에 대해 정리를 해보겠다.

## Http Status Code 그룹정의
Spring-project github에 가보면 HttpStatus 하위에 내부 Enum으로 아래처럼 정의되어 있다. ( [링크](
https://github.com/spring-projects/spring-framework/blob/4.3.x/spring-web/src/main/java/org/springframework/http/HttpStatus.java#L505) )
```java
public enum HttpStatus {
	
	...

	public Series series() {
		return Series.valueOf(this);
	}

	...

	public enum Series {

		INFORMATIONAL(1),
		SUCCESSFUL(2),
		REDIRECTION(3),
		CLIENT_ERROR(4),
		SERVER_ERROR(5);

		...

		public static Series valueOf(int status) {
			int seriesCode = status / 100;
			for (Series series : values()) {
				if (series.value == seriesCode) {
					return series;
				}
			}
			throw new IllegalArgumentException("No matching constant for [" + status + "]");
		}
	}
}	
```
약 80여개 응답코드들을 크게 5개 묶음으로 정의해 놓은것을 볼수있다. 즉, 201 / 201 / 202 같은 녀석들은 전부 `SUCCESSFUL` 성공 으로 그룹핑이 되는것을 확인할수 있다.

## 그럼 RestTemplate 에서는 어떻게 처리하고 있나?
코드를 쭉쭉 따라가다 보면 아래처럼 4xx, 5xx 는 에러라고 판단하고 그에 따라 RestClientException 을 반환하는것을 확인할수 있다. 
- RestTemplate 호출 [링크](https://github.com/spring-projects/spring-framework/blob/4.3.x/spring-web/src/main/java/org/springframework/web/client/RestTemplate.java#L649 )
```java
try {
	ClientHttpRequest request = createRequest(url, method);
	if (requestCallback != null) {
		requestCallback.doWithRequest(request);
	}
	response = request.execute();
	handleResponse(url, method, response);
	if (responseExtractor != null) {
		return responseExtractor.extractData(response);
	}
	else {
		return null;
	}
}
catch (IOException ex) {
```
- 에러 판별 [링크](
https://github.com/spring-projects/spring-framework/blob/4.3.x/spring-web/src/main/java/org/springframework/web/client/DefaultResponseErrorHandler.java)
```java
protected boolean hasError(HttpStatus statusCode) {
	return (statusCode.series() == HttpStatus.Series.CLIENT_ERROR || statusCode.series() == HttpStatus.Series.SERVER_ERROR);
}
```

## 성공/실패 처리는 어떻게 할것인가
각자 정의하기 나름일것 같다. 우선 아주 간단하게 RestTemplate 를 사용할때 예외처리를 하여 정의된 대로 4xx, 5xx가 에러라고 판단할 수 있을것 같고
```java
try {
    responseBody = restTemplate.postForObject(url, httpEntity, byte[].class);
} catch (RestClientException e) { // 에러인 경우 RestClientException 을 내뱉는다.
    log.error("##### restTemplate error, url = {}", url, e);
}
```
정의된 에러(?)와는 조금 다르게 처리하고 싶다면 `DefaultResponseErrorHandler`을 상속받고 `hasError`메소드를 무조건 패스하도록 Override 하고난 다음 응답 코드를 받아서 처리하는 방법이 있을수 있겠다. 
(물론 이러한 방법 말고 다양한 방법이 있을것 같다.)