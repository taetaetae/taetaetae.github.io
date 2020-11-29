---
title: Spring MVC Redirect 처리중에 발생한 Out Of Memory 원인 분석하기
date: 2019-01-10 18:39:47
categories:
  - tech
tags: 
  - spring
  - redirect
  - out of memory
  - heap dump
url : /2019/01/10/spring-redirect-oom/
featuredImage: /images/spring-redirect-oom/test1-pinpoint.jpg
images :
  - /images/spring-redirect-oom/test1-pinpoint.jpg

---
초창기 신입시절에 배우거나 사용했던 기술적인 방법들이 있다. 시간이 지날수록 왠만해선 다른방법은 사용하지 않으려 하고 `습관`처럼 기존에 사용했던 방법을 고수하는 버릇이 있다. 그 이유는 과거에 사용했을때 아무 탈 없이 잘 되었기 때문에, 그리고 빠른 구현 때문이라는 핑계일 것 같다. <!-- more -->이러한 버릇은 비단 이 글을 적고있는 필자 뿐만이 아니라 대부분의 개발자들이 가지고 있을꺼라 조심스레 추측해본다. (아니라면...더욱 분발 해야겠다...ㅠ)
최근 운영하고 있는 서비스에서 장애 상황까지 갈수있는 위험한 상황이 있었는데 팀내 코드리뷰를 통해 문제점을 파악할 수 있었다. 그 원인은 Spring MVC Controller 레벨에서 redirect 처리를 할때 return값의 Cardinality가 높을경우 다음과 같이 사용하면 안된다고...

```java
@RequestMapping(value = "/test", method = RequestMethod.GET)
public String test() {
	String url = "어떠한 로직에 의해 생성되는 url";
	return "redirect:" + url; // <- 위험 포인트!
}
```

이 코드가 왜? 어디가 어때서?
이제까지 Controller 레벨에서 redirect 처리를 할때 아무생각없이 위에 있는 코드 형태로 구현을 했는데 저러한 코드 때문에 OOM이 발생하여 fullGC 가 여러번 발생하고, 일시적으로 서비스가 지연되는 현상이 발생했다고 한다. 자주 사용하던 방법이였는데 장애를 유발할수 있는 위험한 방법이였다니... 
이번 포스팅에서는 이러한 방법이 왜 잘못되었는지 실제로 테스트를 통해 몸소(?) 체감을 해보고, 그럼 어떤 방법으로 redirect 처리를 해야 하는가와 개선을 함으로써 기존방식에 비해 어떤점이 좋아졌는지에 대해서 정리해보고자 한다. 

> 뭔가 **내것으로 만들기** 시리즈물이 나올것만 같은 느낌이다...


## 기존방식의 문제점 재현 및 다양한 원인분석
기존방식으로 했을때 왜 OOM이 발생했을까? 우리는 개발자이기 때문에 이런저런 글들만 보고 추측 할것이 아니라 직접 재현을 해보고 다양한 시각에서 원인분석을 해보자.
먼저 기본적인 Spring MVC 뼈대를 만들고 redirect 하는 return 값의 Cardinality가 높도록 random string 을 만들어 주도록 한다. 즉, `/random`을 호출하면 `/result/ETmHfowFkU`처럼 random string 이 만들어 지며 redirect 처리가 되는 매우 심플한 구조이다.

```java
// Spring 버전은 4.0.6.RELEASE
@Controller
@RequestMapping("/")
public class TestController {
	@RequestMapping(value = "random", method = RequestMethod.GET)
	public String random() {
		return "redirect:result/" + UUID.randomUUID();
	}

	@RequestMapping(value = "result/{message}", method = RequestMethod.GET)
	public String result(ModelMap model, @PathVariable String message) {
		model.addAttribute("message", message);
		return "result";
	}
}
```

또한 해당 프로젝트에서는 AOP를 사용하고 있었기 때문에 그때와 동일한 상황으로 재현을 하기 위해 AOP관련 설정도 추가해준다.

```java
@Configuration
@EnableWebMvc
@EnableAspectJAutoProxy
@ComponentScan
public class HelloWorldConfiguration {
	
	@Bean(name="HelloWorld")
	public ViewResolver viewResolver() {
		InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
		viewResolver.setViewClass(JstlView.class);
		viewResolver.setPrefix("/WEB-INF/views/");
		viewResolver.setSuffix(".jsp");

		return viewResolver;
	}
}
```

이렇게 한뒤 tomcat으로 최대/최소 메모리를 256m으로 설정후 해당 모듈을 띄워준다. 그다음 메모리 상태를 보기 위해 tomcat에 pinpoint를 연동하고 마지막으로 호출테스트를 위해 nGrinder을 설정해준다. 특별한 설정은 없고 위 컨트롤러의 url (/random) 을 여러번 호출하도록 하였다. nGrinder을 설정하는대에는 [이 블로그 포스팅](https://black9p.github.io/2019/01/02/nGrinder-%EA%B0%84%ED%8E%B8-%EC%82%AC%EC%9A%A9%EA%B0%80%EC%9D%B4%EB%93%9C/)을 참고해서 설정하였다.

자, 이제 테스트를 시작해보자. (마치 수술 집도하는것 같은 기분으로...간호사~ 칼!)
1. nGrinder
  nGrinder의 기본 스크립트에서 url만 해당 서버로 호출되도록 바꿔주고 총 가상 사용자는 2,000으로 시간은 5분으로 설정후에 테스트 시작을 하였더니 다음과 같은 그래프를 볼수 있었다.

  {{< image src="/images/spring-redirect-oom/test1-ngrinder.jpg"  caption="" width="100%" >}}

  TPS가 불안정해지다가 어느시점부터 낮아지는것을 확인할 수 있다. 이게 서비스 였다면 사용자가 접속하는데 불편을 느꼈을꺼라 추측을 해본다. 또한 아주 간단한 random string 을 리턴하는 페이지 임에도 불구하고 에러 응답이 적지 않은것을 확인할 수 있었다.

2. pinpoint
  메모리 상태는 어떤지 확인하기 위해 pinpoint를 확인해보면 다음과 같은 그래프를 볼수 있었다.

  {{< image src="/images/spring-redirect-oom/test1-pinpoint.jpg"  caption="" width="100%" >}}

  보기만해도 심장이 벌렁벌렁(?) 뛸 정도로 무서운 그림이다. 실제로 서비스에 (이정도까진 아니였지만) 비슷한 상황이 발생했었다. 메모리가 테스트를 점점 하면 할수록 올라가다가 fullGC가 발생하더니 대나무 숲에 있는 대나무마냥 fullGC가 빼곡히 발생하였다. (이러니... 페이지 접근에 지연이 생긴것 같다.)

3. Heap dump
  그럼 실제로 메모리는 어떤 상태였고 어디서 메모리를 많이 사용하고(점유하고) 있는지를 확인하기 위해 Heap dump를 생성해 보았다. 힙덤프 분석하는데 잘 알려진 [Memory Analyzer (MAT)](https://www.eclipse.org/mat/)를 다운받고 해당 프로세스의 힙덤프를 생성한다음 분석을 해봤더니 아래와 같은 화면을 볼 수 있었다.

  {{< image src="/images/spring-redirect-oom/test1-dump1.jpg" caption="" width="100%" >}}  

  힙덤프 파일을 열자마자 (저 문제 있어요~ 도와주세요 하듯) 뭔가 많이 점유하고 있는것처럼 보이는 파이그래프가 Overview에 보였다. Reports 영역에 있는 Leak Suspects를 확인해보니 아래 경로에서 많이 사용하는 것을 확인할 수 있었다.
  > java.util.concurrent.ConcurrentHashMap$Node
  
  {{< image src="/images/spring-redirect-oom/test1-dump2.jpg" caption="" width="100%" >}}

  이 툴에서는 OQL이라고 힙덤프에 있는 데이터를 일반 SQL처럼 쿼리처럼 볼수 있었다. 그래서 아래처럼 쿼리를 작성해서 봤더니 결과만 봐도 어디서 메모리를 점유하고 있는지 한눈에 볼수 있었다
  ```sql
  SELECT o.key.toString() 
  FROM java.util.concurrent.ConcurrentHashMap$Node o 
  WHERE ((o.key != null) and (o.key.toString().indexOf("org.springframework.web.servlet.view.RedirectView_redirect") = 0))
  ```

  {{< image src="/images/spring-redirect-oom/test1-dump3.jpg" caption="" width="100%" >}}

  무작위로 만들어진 url에 대한 정보를 캐시하고 있는 듯한 결과였다.


## 방식 개선 및 변화 비교
결국 `return "redirect:" + url;` 와 같은 처리가 문제를 야기했던 것이였다. 그럼 redirect 처리를 어떻게 하는게 좋을까? 
조금 검색을 해보면 `RedirectView` 나 `ModelAndView`를 사용하라고 권장하고 있다. 물론 redirect 되는 url의 Cardinality가 높지않고 고정적이라면 지금의 `return "redirect:" + url;` 이 방식을 사용해도 무방할수 있다. 하지만 컨트롤러 메소드가 String타입을 return 하게 되면 View 클래스로 변환작업을 진행하게 된다고 한다.
이 작업중에 `org.springframework.beans.factory.config.BeanPostProcessor`구현체들도 같이 진행되고, 이중에 하나가 `AnnotationAwareAspectJAutoProxyCreator` 라고 있는데 해당 클래스 내부적으로 `ConcurrentHashMap<Object, Boolean>` 타입 객체에 key : viewName, value : 필요 여부(boolean) 형태로 갯수 제한 없이 저장하고 있다. 그러다보니 url의 종류가 많아질수록 메모리가 많이 사용될 수밖에 없었던 것 같다. 
즉, 동일한 url에 대해 View 객체를 캐싱하고 있으니 위와 같이 url의 종류가 다양할경우 (특히 로그인 같은 처리를 할때 고유값을 파라미터로 넘기는 경우) 캐싱 객체 숫자가 많아지기 마련이다.

실제로 코드를 보면 캐싱을 하고있는것을 볼수 있다. [Spring-project github](https://github.com/spring-projects/spring-framework/blob/master/spring-aop/src/main/java/org/springframework/aop/framework/autoproxy/AbstractAutoProxyCreator.java#L358)
```java
// ... 생략
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
  // ... 생략

  if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
    this.advisedBeans.put(cacheKey, Boolean.FALSE); // <- 여기 
    return bean;
  }

  // ... 생략

  this.advisedBeans.put(cacheKey, Boolean.FALSE); // <- 여기 
  return bean;
}
// ... 생략
```

자, 그럼 개선방법을 알아봤으니 한번 비교를 해보자. 아래처럼 `ModelAndView`를 사용해서 redirect처리를 할수있도록 코드를 변경하고

```java
@Controller
@RequestMapping("/")
public class TestController {
  @RequestMapping(value = "random", method = RequestMethod.GET)
  public ModelAndView random() {
    ModelAndView modelAndView = new ModelAndView();

    RedirectView redirectView = new RedirectView();
    redirectView.setUrl("result/" + UUID.randomUUID());

    modelAndView.setView(redirectView);
    return modelAndView;
  }

  @RequestMapping(value = "/result/{message}", method = RequestMethod.GET)
  public String result(ModelMap model, @PathVariable String message) {
    model.addAttribute("message", message);
    return "result";
  }
}
```
기존과 동일한 방법과 환경에서 테스트를 해보자.

1. nGrinder
  위해서 했던 방법과 동일한 가상 사용자 수와 동일한 시간으로 테스트를 해보니 다음과 같은 그래프를 볼수 있었다.
  {{< image src="/images/spring-redirect-oom/test2-ngrinder.jpg" caption="" width="100%" >}}

  위에서 봤던 들쭉날쭉 그래프보다 훨씬 더 안정적인것을 볼 수 있었고, 에러도 단 한건도 없이 훨씬 높은 TPS를 끝까지 일정하게 유지하는 모습을 볼수 있었다. (로직상 에러가 나는게 이상한... 아니 안나야 정상이다.) 

2. pinpoint
  TPS가 안정적이였기 때문에 메모리의 상태를 안봐도 되겠지만 비교의 목적이 있기 때문에 pinpoint 의 그래프를 한번 보자.

  {{< image src="/images/spring-redirect-oom/test2-pinpoint.jpg" caption="" width="100%" >}}

  nGrinder로 테스트하는 시점만 잠깐 메모리가 올라가다가 다시 내려오고 전에 있었던 fullGC도 없고 위에서 테스트 했던 그래프 보다는 안정적인 그래프라고 볼수 있었다.

3. Heap dump
  메모리가 안정적이였지만 혹시 pinpoint에서 잘못 집계하거나 그래프만 보고 맹신할수 없었기 때문에 이번에도 Heap dump를 생성해 보았다.

  {{< image src="/images/spring-redirect-oom/test1-dump2.jpg" caption="" width="100%" >}}

  일단 점유하고 있는 메모리의 크기가 약 10분의 1정도로 줄어든것을 확인할수 있었고, 위에서 했던 OQL을 이용한 메모리 점유를 확인해봐도 기존에 있던 `RedirectView_redirect`관련 데이터가 아예 없음을 확인할 수 있었다.

  {{< image src="/images/spring-redirect-oom/test2-dump2.jpg" caption="" width="100%" >}}


코드 몇줄 변경한것밖에 없는데 같은 테스트 환경에서 확연히 좋아진것을 확인할 수 있다. (뿌듯) 전체적으로 다시 비교를 해보면 아래와 같이 이쁜(?) 변화를 볼수가 있다.

{{< image src="/images/spring-redirect-oom/test-diff.jpg" caption="" width="100%" >}}


## 마치며
평소에 자주 사용하던 방식인데 성능적으로 자칫 치명적인 결과를 가져올수 있다고 하여 재현을 안해볼수가 없었다. 만약 악의적으로 url뒤에 무작위 문자열을 더해서 ddos공격을 했더라면? 얼마 안가서 서버가 터졌을지도 모른다.
지금이라도 알아서 다행이라는 생각과 재현을 안해보고 그냥 그런가보다 하며 넘어갔다면 실제 Spring 내부 코드까지 볼일이 있었을까 하는 생각을 해본다. 
이번에 재현을 해보면서 nGrinder 로 성능테스트에 pinpoint 모니터링, 마지막으로 힙덤프 분석까지. 꼭 이번 url redirect 문제만이 아니라 다른 성능적인 이슈가 생길때 마치 `치트키`처럼 활용할수 있을 `나만의 무기`를 얻은것 같아 다시한번 뿌듯함을 느낀다.

마지막으로, 이렇게 재현까지 하도록 `자극`을 주신 팀 동료분께 감사드린다고 전하고 싶다. (보실지 안보실지 모르겠지만 ^^;)

---

- 관련 참고글
  - https://www.baeldung.com/spring-redirect-and-forward
  - https://www.slideshare.net/benelog/ss-35627826