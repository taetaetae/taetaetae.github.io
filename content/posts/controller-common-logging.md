---
title: Spring에서 Request를 우아하게 로깅하기
date: 2019-06-30 18:39:47
categories:
  - tech
tags:
  - spring
  - logging
  - HttpServletRequestWrapper
  - Filter
  - AOP
url : /2019/06/30/controller-common-logging/
featuredImage: /images/controller-common-logging/spring_boot_logging.png
images :
  - /images/controller-common-logging/spring_boot_logging.png
---

스프링 기반의 웹 어플리케이션을 만들다 보면 요청을 처리하는데 맨 처음에 위치하고 있는 `Controller`(이하 컨트롤러)라는 레이어를 만들게 된다. 그럴때면 사용자가 어떤 요청(Request)을 하였는지에 대해 확인이 필요할 수 있다. <!--more --> 물론 확인을 안해도 무방하지만 가급적 로깅은 시스템 로직에 영향을 주지 않는 범위에서 최대한 다양하게 `미리` 해두는게 나중에 유지보수시 편할 수 있다. (예전 조직장님께서 말씀하신게 아직도 머릿속에 꽉 자리잡고 있다...)
아~주 일반적으로, 컨트롤러에서는 다음과 같이 메소드 단위로 파라미터를 직접 로깅하게 된다.
```java
@Slf4j
@RestController
public class SampleController {

	@GetMapping("/test1")
	public String test1(@RequestParam String id) {
		log.info("id : {}", id);
		return "length : " + id.length();
	}
}

```
이렇게 되면 사용자가 `GET /test1` 이라는 요청을 보낼때 어떤 파라미터로 호출하였는지에 대해 로깅이 남게 되는데 항상 `log.info("id : {}", id);` 과 같이 수동으로 로깅을 남겨야 하는 불편함이 생긴다. 물론 꼼꼼하게 메소드마다 로깅을 적어주면 전혀 문제될게 없지만 이러한 컨트롤러 ~ 메소드가 한두개가 아닌 수십 또는 수백개일 경우엔 그때마다 로깅을 적어줘야 하는 불편함이 있을 수 있다. 또한 자칫 깜박하고 로깅을 빼먹고 배포를 하게 된 경우 모니터링시 로깅을 하지 않아서 다시 로깅하고 배포를 하는, 별것도 아닌데(?) "정말 불편한" 상황이 있을 수 있다.
이번 포스팅에서는 사용자의 요청을 모니터링 하기 위해 컨트롤러마다 코드를 작성해가며 로깅을 하는것이 아니라 `HttpServletRequestWrapper` 라는 것과 `Filter`, `AOP`를 이용하여 **Request의 정보를 한곳에서 우아하게 로깅하는 방법**에 대해 알아보고자 한다.

## 요구사항

{{< image src="/images/controller-common-logging/bull_fighting.gif" caption="와 개발하자아!<br>출처 : https://gfycat.com/ko/brightevilaoudad" width="80%" >}}


투우사가 흔드는 빨간 천을 보며 돌진하는 황소처럼 (쓰고보니 너무 TMI 같다....) 당장 코딩을 시작하며 개발을 할 수도 있지만 정작 원하는 기능이 무엇인지 천천히 정리하고 넘어갈 필요가 있는 것 같다. (어쩔땐 오히려 후자가 더 빠른 개발을 하게 되는것 같다.)
1. GET, POST 등 다양한 http method 로 구현된 모든 컨트롤러의 파라미터와 기타 Request 정보가 로깅이 되야 한다.
2. 컨트롤러, 메소드가 늘어날때마다 별도의 코드 추가 없이 한곳에서 공통적으로 로깅이 되야 한다.
3. URL 중 특정 패턴으로 들어오는 요청은 다른 방식으로 로깅을 하거나, 로깅에서 제외할 수 있어야 한다.
4. 앞서 말했듯 다른 비지니스 로직에 영향을 주지 않아야 한다.

## 구현하기 - Request 의 파라미터 정리
Request 의 모든 로깅을 한곳에서 처리하기 위해서 filter(필터)를 활용하였다. 필터는 Dispatcher servlet의 앞단에 위치하고 있기 때문에 모든 정보를 확인할 수 있는데 용이하다. 물론 인터셉터를 활용해서도 방법이 있겠지만 본 포스팅 에서는 필터를 활용해서 구현하는것을 목적으로 한다. (사실 인터셉터로 몇번 시도해보다가 실패해서...유유 )

{{< image src="/images/controller-common-logging/spring-request-lifecycle.jpg" caption="Spring MVC Request Life Cycle<br>출처 : https://justforchangesake.wordpress.com/2014/05/07/spring-mvc-request-life-cycle/" width="80%" >}}

Filter를 만들기 전에 Filter에서 사용할 주요 핵심(?) 클래스가 필요한데 `HttpServletRequest` 를 Wrapping 해서 사용하기 위해 `HttpServletRequestWrapper`를 상속받는 클래스를 만들자. Request 에 담겨있는 param 과 body로 요청이 들어올 경우 body에 있는 내용을 param 에 담는 로직이다. 주요 설명은 코드 안에서 주석으로 설명하겠다. 
```java
public class ReadableRequestWrapper extends HttpServletRequestWrapper { // 상속
	private final Charset encoding;
	private byte[] rawData;
	private Map<String, String[]> params = new HashMap<>();

	public ReadableRequestWrapper(HttpServletRequest request) {
		super(request);
		this.params.putAll(request.getParameterMap()); // 원래의 파라미터를 저장

		String charEncoding = request.getCharacterEncoding(); // 인코딩 설정
		this.encoding = StringUtils.isBlank(charEncoding) ? StandardCharsets.UTF_8 : Charset.forName(charEncoding);

		try {
			InputStream is = request.getInputStream();
			this.rawData = IOUtils.toByteArray(is); // InputStream 을 별도로 저장한 다음 getReader() 에서 새 스트림으로 생성

			// body 파싱
			String collect = this.getReader().lines().collect(Collectors.joining(System.lineSeparator()));
			if (StringUtils.isEmpty(collect)) { // body 가 없을경우 로깅 제외
				return;
			}
			if (request.getContentType() != null && request.getContentType().contains(
				ContentType.MULTIPART_FORM_DATA.getMimeType())) { // 파일 업로드시 로깅제외
				return;
			}
			JSONParser jsonParser = new JSONParser();
			Object parse = jsonParser.parse(collect);
			if (parse instanceof JSONArray) {
				JSONArray jsonArray = (JSONArray)jsonParser.parse(collect);
				setParameter("requestBody", jsonArray.toJSONString());
			} else {
				JSONObject jsonObject = (JSONObject)jsonParser.parse(collect);
				Iterator iterator = jsonObject.keySet().iterator();
				while (iterator.hasNext()) {
					String key = (String)iterator.next();
					setParameter(key, jsonObject.get(key).toString().replace("\"", "\\\""));
				}
			}
		} catch (Exception e) {
			log.error("ReadableRequestWrapper init error", e);
		}
	}

	@Override
	public String getParameter(String name) {
		String[] paramArray = getParameterValues(name);
		if (paramArray != null && paramArray.length > 0) {
			return paramArray[0];
		} else {
			return null;
		}
	}

	@Override
	public Map<String, String[]> getParameterMap() {
		return Collections.unmodifiableMap(params);
	}

	@Override
	public Enumeration<String> getParameterNames() {
		return Collections.enumeration(params.keySet());
	}

	@Override
	public String[] getParameterValues(String name) {
		String[] result = null;
		String[] dummyParamValue = params.get(name);

		if (dummyParamValue != null) {
			result = new String[dummyParamValue.length];
			System.arraycopy(dummyParamValue, 0, result, 0, dummyParamValue.length);
		}
		return result;
	}

	public void setParameter(String name, String value) {
		String[] param = {value};
		setParameter(name, param);
	}

	public void setParameter(String name, String[] values) {
		params.put(name, values);
	}

	@Override
	public ServletInputStream getInputStream() {
		final ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(this.rawData);

		return new ServletInputStream() {
			@Override
			public boolean isFinished() {
				return false;
			}

			@Override
			public boolean isReady() {
				return false;
			}

			@Override
			public void setReadListener(ReadListener readListener) {
				// Do nothing
			}

			public int read() {
				return byteArrayInputStream.read();
			}
		};
	}

	@Override
	public BufferedReader getReader() {
		return new BufferedReader(new InputStreamReader(this.getInputStream(), this.encoding));
	}
}
```
가장 중요한 부분은 `IOUtils.toByteArray(is)` 요 부분인데, InputStream은 한번밖에 읽을 수 없기 때문에 이 필터에서 스트림을 읽는 대신, 래퍼 구현으로 새 스트림 생성하도록 작업을 하였다. 자칫 잘못하다간 body의 내용이 유실될 수도 있기 때문이다.
참조 : 
http://stackoverflow.com/questions/10210645/http-servlet-request-lose-params-from-post-body-after-read-it-once
http://stackoverflow.com/questions/3769259/why-is-the-parameter-value-an-object-hash-code-for-request-getparametermap-ge

위에서 만든 래퍼 클래스를 이제 필터에서 적용해보자.
```java
public class ReadableRequestWrapperFilter implements Filter {
	@Override
	public void init(FilterConfig filterConfig) {
		// Do nothing
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
		throws IOException, ServletException {
		ReadableRequestWrapper wrapper = new ReadableRequestWrapper((HttpServletRequest)request);
		chain.doFilter(wrapper, response);
	}

	@Override
	public void destroy() {
		// Do nothing
	}
}
```

이렇게 되면 param으로 넘어온 파라미터나 body로 넘어온 정보들이 파싱되어 param에 담기게 된다. 
(JSON으로 파싱하는 부분은 조금 지저분해 보일 수 있지만 다양한 테스트를 해보면서 얻은 결과물이다. 더 좋은 방식이 있다면 언제든지 PullRequest 또는 댓글 환영!)

## 구현하기 - 컨트롤러에 AOP 셋팅
이제 Request 의 params 를 한곳에서 로깅할 차례이다. 물론 위에서 만든 필터에서 로깅을 해도 되지만 필터에 로직이 들어가는 것보다 별도의 로직에서 처리하는게 맞다고 생각되어 분리를 하였다. (관심사의 분리)
AOP를 활용하여 다음과 같이 로직을 작성하였다. 아래 로직은 이해하기에 큰 어려움은 없을것 같지만 주석으로 설명을 하겠다.
```java
@Component
@Aspect
@Slf4j
public class LoggerAspect {
	@Pointcut("execution(* com.taetaetae..*Controller.*(..))") // 이런 패턴이 실행될 경우 수행
	public void loggerPointCut() {
	}

	@Around("loggerPointCut()")
	public Object methodLogger(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
		try {
			Object result = proceedingJoinPoint.proceed();
			HttpServletRequest request = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getRequest(); // request 정보를 가져온다.

			String controllerName = proceedingJoinPoint.getSignature().getDeclaringType().getSimpleName();
			String methodName = proceedingJoinPoint.getSignature().getName();

			Map<String, Object> params = new HashMap<>();

			try {
				params.put("controller", controllerName);
				params.put("method", methodName);
				params.put("params", getParams(request));
				params.put("log_time", new Date());
				params.put("request_uri", request.getRequestURI());
				params.put("http_method", request.getMethod());
			} catch (Exception e) {
				log.error("LoggerAspect error", e);
			}
			log.info("params : {}", params); // param에 담긴 정보들을 한번에 로깅한다.

			return result;

		} catch (Throwable throwable) {
			throw throwable;
		}
	}

	/**
	 * request 에 담긴 정보를 JSONObject 형태로 반환한다.
	 * @param request
	 * @return
	 */
	private static JSONObject getParams(HttpServletRequest request) {
		JSONObject jsonObject = new JSONObject();
		Enumeration<String> params = request.getParameterNames();
		while (params.hasMoreElements()) {
			String param = params.nextElement();
			String replaceParam = param.replaceAll("\\.", "-");
			jsonObject.put(replaceParam, request.getParameter(param));
		}
		return jsonObject;
	}
}
```
위 코드에서는 단순히 console log를 찍었지만 필자가 운영하고 있는 어드민 툴에서는 Request 정보를 Elasticsearch 에 인덱싱하고 이를 키바나에서 보다 쉽게 조회 및 모니터링이 가능하도록 구현하였다. 로깅을 어떤식으로 남길지, 로깅이 아닌 특정 상황에서 알림을 보낸다거나 등등 이 부분은 실제 구현시에 상황에 맞춰서 만들면 될 것 같다.
또한 `Pointcut` 부분을 다르게 활용하여 위에서 이야기한 요구사항의 3번 (URL 중 특정 패턴으로 들어오는 요청은 다른 방식으로 로깅을 하거나, 로깅에서 제외할 수 있어야 한다.) 를 해결할 수 있다.
아참, AOP의 개념을 좀더 알고 싶다면 jojoldu 님의 블로그 포스팅을 추천한다. 
https://jojoldu.tistory.com/71

그래서 아래처럼 테스트를 해보면 GET 이던 POST 이던 로깅이 되는것을 확인할수 있다.

{{< image src="/images/controller-common-logging/test.jpg" caption="더 이쁘게 로깅을 하는건 자유!" width="80%" >}}


## 마치며
사실 이 로직을 만들게 된 계기는 팀원중에 한분이 "GET 의 파라미터는 로깅이 되는데 POST 의 파라미터는 로깅이 안되요" 라는 말에 투우사 앞에 있는 황소마냥 찾아보다 만들게 되었다. 참고로 이 부분은 하나의 `무기`가 될것 같아 github에 공개를 하였고 위에서 적었던 요구사항보다 더 좋은 내용이 있다면 언제든지 PullRequest 를 날려주기를 바란다. Spring Boot 환경에서 Filter를 추가하며 구성하였다. 
https://github.com/taetaetae/request_logging
필터와 인터셉터, AOP 등 평소 자주 사용하지 않은 (이미 누군가 다 만들어 둔) 부분을 만지면서 다시한번 Spring의 주요 개념을 숙지하는데 도움이 되었던 경험으로 남을 것 같다.