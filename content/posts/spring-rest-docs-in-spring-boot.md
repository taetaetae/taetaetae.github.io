---
title: SpringRestDocs를 SpringBoot에 적용하기
date: 2020-03-08 23:16:59
categories:
  - tech
tags: 
  - spring boot
  - spring rest docs
  - archives-2020
url : /2020/03/08/spring-rest-docs-in-spring-boot/
featuredImage: /images/spring-rest-docs-in-spring-boot/logo.jpg
---

API를 개발하고 제공하기 위해서는 그에 해당하는 API 명세를 작성해서 사용하는 곳에 전달하게 된다. 어떤 URL에 어떤 파라미터를 사용해서 어떻게 요청을 하면 어떤 결과를 응답으로 내려주는지에 대한 관련 정보들. 이러한 "API 문서" 를 제공하는 방식은 상황에 따라 다양한 방법으로 사용되곤 한다. <!--more -->

{{< image src="/images/spring-rest-docs-in-spring-boot/api-document.jpg" caption="API 코드와 해당 문서의 동기화가 자동으로 되어야 조금 편해질것 같다는 생각이 들었다. <br>출처 : https://dribbble.com/shots/3386291-API-Documentation" width="50%" >}}

필자는 주로 "위키"(또는 일반 문서)를 활용해서 전달하곤 했었는데 API의 형태가 달라질 때마다 해당 위키를 수정해야만 하는 번거로움이 있었다. API 수정하면 위키도 수정하고. 깜박하고 위키 수정을 안하게 될 경우 왜 API 명세가 다르냐는 문의가... 그러다 알게된 Spring Rest Docs. (아무리 좋은 기술, 좋은 툴 이라 해도 실제로 본인이 필요로 하고 사용을 해야하는 이유가 생길때 비로소 빛을 발하는것 같은 느낌이다.)
> 이 포스팅에서는 swegger 와 비교하는 내용은 제외할까 한다. 워낙 유명한 두 양대 산맥(?)이라 검색해보면 각각의 장단점이 자세히 나와있기에...

최근 들어 TestCode 의 중요성을 절실하게 느끼고 있었고, TestCode 를 작성하면 자연스럽게 문서를 만들어 주는 부분이 가장 매력적이라고 생각이 들었다. 이를 반대로 생각하면, TestCode 가 실패할 경우 빌드 자체가 안되기에 어쩔수 없이 TestCode를 성공시켜야만 하고, 자연스럽게 정상적인(최신화 된) API 문서가 만들어지게 된다. 

이번 포스팅에서는 다음과 같은 목표를 두고 실무에서 언제든지 활용이 가능한 약간의 "가이드" 같은 내용으로 작성해 보고자 한다. 
- Spring Boot 최신 버전에서 Spring Rest Docs 를 설정한다.
- 임의의 API 를 만들고 그에 따른 TestCase 를 작성한다.
- Spring.profile 에 따라 Spring Rest Docs Url 을 접근 가능/불가능 할 수 있게 한다.

물론 필자의 방법이 다를수도 있지만, 이러한 방법을 토대로 보다 더 우아하고 아름다운 방법을 알아갈수 있지 않을까 하는 기대로.

## Spring Boot 에 Spring Rest Docs 셋팅하고 TestCase 작성하기
우선 Spring Boot 프로젝트를 만든다. https://start.spring.io/ 에서 만들어도 되고 IDE 에서 제공하는 툴로 만들어도 되고. 만드는 방식은 무방하다. 그 다음 필요한 dependency 를 추가해 준다.
```xml
<dependency>
	<groupId>org.springframework.restdocs</groupId>
	<artifactId>spring-restdocs-mockmvc</artifactId>
	<scope>test</scope>
</dependency>
```

임의로 API를 작성하고
- 모델 
```java
@Getter
@Setter
public class Book {
	private Integer id;
	private String title;
	private String author;
}
```
- 컨트롤러
```java
@RestController
public class BookController {

	@GetMapping("/book/{id}")
	public Book getABook(@PathVariable Integer id) {
		Book book = new Book();
		book.setId(id);
		book.setTitle("spring rest docs in spring boot");
		book.setAuthor("taetaetae");
		return book;
	}
}
```

해당 컨트롤러에 대한 TestCase 를 작성하자.
```java
@WebMvcTest(BookController.class)
@AutoConfigureRestDocs // (1)
public class BookControllerTest {

	@Autowired
	private MockMvc mockMvc; // (2)

	@Test
	public void test_책을_조회하면_null이_아닌_객체를_리턴한다() throws Exception {
		mockMvc.perform(get("/book/{id}", 1)
			.accept(MediaType.APPLICATION_JSON))
			.andDo(MockMvcResultHandlers.print())
			.andExpect(MockMvcResultMatchers.status().isOk())
			.andDo(document("book", // (3)
				pathParameters(
					parameterWithName("id").description("book unique id") // (4)
				),
				responseFields(
					fieldWithPath("id").description("book unique id"),
					fieldWithPath("title").description("title"),
					fieldWithPath("author").description("author")
				)
			))
			.andExpect(jsonPath("$.id", is(notNullValue()))) // (5)
			.andExpect(jsonPath("$.title", is(notNullValue())))
			.andExpect(jsonPath("$.author", is(notNullValue())));
	}
}
```

(1) Spring Boot 에서는 해당 어노테이션으로 여러줄에 걸쳐 설정해야 할 Spring Rest Docs 관련 설정을 아주 간단하게 해결할 수 있게 된다. ([참고](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-autoconfigured-rest-docs))

(2) [공식 도큐먼트](https://docs.spring.io/spring-restdocs/docs/2.0.4.RELEASE/reference/html5/#getting-started-sample-applications) 에서는 4가지 방식을 말하고 있는데 이 포스팅 에서는 "MockMvc" 을 사용하고자 한다.

(3) "book" 이라는 identifier 를 지정하면 해당 TestCase 가 수행될때 snippets 가 생성되는데 해당 identifier 묶음으로 생성이 된다.

(4) request의 파라미터 필드, response의 필드의 설명을 적어줌으로써 이 정보를 가지고 snippets 가 생성이 되고 결과적으로 API 문서가 만들어 진다.

(5) 필자가 가장 매력적이라 생각되는 부분. 이 부분에서 테스트를 동시에 함으로써 응답이 달라지거나 잘못된 응답이 내려올 경우 TestCase가 실패하게 되어 API문서 또한 생성되지 않게 된다.

여기까지 한 뒤 빌드시 문서가 제대로 만들어 지도록 빌드 플러그인을 추가해 준다.
```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.asciidoctor</groupId>
			<artifactId>asciidoctor-maven-plugin</artifactId>
			<version>1.5.8</version>
			<executions>
				<execution>
					<id>generate-docs</id>
					<phase>prepare-package</phase>
					<goals>
						<goal>process-asciidoc</goal>
					</goals>
					<configuration>
						<backend>html</backend>
						<doctype>book</doctype>
					</configuration>
				</execution>
			</executions>
			<dependencies>
				<dependency>
					<groupId>org.springframework.restdocs</groupId>
					<artifactId>spring-restdocs-asciidoctor</artifactId>
					<version>${spring-restdocs.version}</version>
				</dependency>
			</dependencies>
		</plugin>
	</plugins>
</build>
```

빌드 플러그인에 의해 snippets를 조합하여 최종 API 문서가 만들어질수 있도록 src/main/ 하위에 asciidoc 이라는 폴더를 만들고 그 하위에 적당한 이름의 adoc 파일을 작성한다. 파일 이름은 나중에 만들어질 API문서의 html 파일 이름이 된다.

```adoc
= RESTful Notes API Guide
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 4
:sectnums:
:sectlinks:
:sectanchors:

[[api]]
== Book Api
api 에 관련된 설명을 이곳에 적습니다.

include::{snippets}/book/curl-request.adoc[]
include::{snippets}/book/http-request.adoc[]
include::{snippets}/book/path-parameters.adoc[]
include::{snippets}/book/http-response.adoc[]
include::{snippets}/book/response-fields.adoc[]
```

위 adoc 파일 작성은 생소하니 [asscidoc document](https://asciidoctor.org/docs/user-manual/)를 참조하면서 작성하면 도움이 될것같다.
그 다음 메이븐 빌드를 하면 TestCase 가 돌면서 snippets 이 생성되고 이를 가지고 빌드 플러그인에 의해 최종 API 문서가 생성이 된다.
```
mvn install

[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.taetaetae.springrestdocs.books.BookControllerTest

...

[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] 
[INFO] --- asciidoctor-maven-plugin:1.5.8:process-asciidoc (generate-docs) @ spring-rest-docs ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] Rendered 경로\spring-rest-docs\src\main\asciidoc\index.adoc
```

그러면 target/generated-docs 하위에 index.html 파일이 생성된 것을 확인할 수 있다.

{{< image src="/images/spring-rest-docs-in-spring-boot/index_html.jpg" caption="생성된것을 볼 수 있고 자동으로 만들어진 snippets들도 볼 수 있다." width="30%" >}}

이 파일을 열어보면 다음처럼 이쁘게 문서가 작성된 것을 확인할 수 있다.

{{< image src="/images/spring-rest-docs-in-spring-boot/docs.jpg" caption="얼마든지 입맛에 맞게 수정이 가능하다." width="80%" >}}

그러면 이 파일을 어떻게 사용하는 곳에 제공할 수 있을까? html 만 따로 전달해야 할까? 메이븐 플러그인 중 "maven-resources-plugin"을 활용하여 API서버를 띄우면 외부에서 URL 로 접근할 수 있도록 설정해보자.
```xml
<build>
	<plugins>
		<plugin>
			<artifactId>maven-resources-plugin</artifactId>
			<version>2.7</version>
			<executions>
				<execution>
					<id>copy-resources</id>
					<phase>prepare-package</phase>
					<goals>
						<goal>copy-resources</goal>
					</goals>
					<configuration>
						<outputDirectory>
							${project.build.outputDirectory}/static/docs
						</outputDirectory>
						<resources>
							<resource>
								<directory>
									${project.build.directory}/generated-docs
								</directory>
							</resource>
						</resources>
					</configuration>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

쉽게말해, 만들어진 문서를 특정 경로로 복사하게 되면 외부에서도 해당 경로로 접근시 볼 수 있게 된다. 지금은 "{domain}/docs/index.html" 을 접근하면 문서를 볼 수 있다.

## Spring.profile 에 따라 Spring Rest Docs Url 을 접근 가능/불가능 할 수 있게 한다.
필요에 따라 운영환경에서는 해당 API 명세를 보여주게 되면 보안상으로 좋지 못하므로 API 문서의 접근을 막아야 한다. 뭔가 아주 간단하게 spring boot 의 설정을 통해 처리 할 수 있을꺼라 생각했지만, 아무리 찾아봐도 그런 기능은 없어보인다.ㅠㅠ 어쩔수 없이 필터를 추가하여 해당 url을 운영환경에서는 FORBIDDEN 처리를 해보도록 하자.
```java
@Component
@WebFilter(urlPatterns = "/docs/index.html")
public class SpringRestDocsAccessFilter implements Filter {

	@Value("${spring.profiles.active}")
	private String phase;

	@Override
	public void init(FilterConfig filterConfig) {
	}

	@Override
	public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
		throws IOException, ServletException {

		if (StringUtils.equals("release", phase)) { // (1)
			HttpServletResponse response = (HttpServletResponse)servletResponse;
			response.sendError(HttpServletResponse.SC_FORBIDDEN);
			filterChain.doFilter(servletRequest, response);
			return;
		}

		filterChain.doFilter(servletRequest, servletResponse);
	}

	@Override
	public void destroy() {
	}
}
```
(1) spring profile 이 release 일 경우 일부러 FORBIDDEN 처리를 해준다.

이렇게 되면 개발환경에서만 접근이 가능하고 운영환경에서는 접근이 불가능한, 테스트 케이스를 성공한 동기화 + 자동화 된 API 문서를 만들 수 있게 되었다.

## 마치며
언제나 그렇듯, 초기 설정은 실제 비즈니스 로직 개발보다 훨씬 공수가 더 드는 건 어쩔 수없다. 하지만 약간의 노력으로 안정되고 자동화된 개발 프로세스를 구축한다면 그 작은 날갯짓이 나중엔 큰 바람을 일으킬 수 있지 않을까 하는 기대를 해본다.

참, 위에서 만든 코드는 [Github](https://github.com/taetaetae/SpringRestDocs-In-SpringBoot) 에 있으니 참고 바란다.