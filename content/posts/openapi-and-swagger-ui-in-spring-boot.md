---
title: OpenAPI 와 Swagger-ui 적용하기
date: 2020-12-20T11:42:40+09:00
categories:
  - tech
tags:
  - Swagger
  - Swagger-ui
  - OpenAPI
featuredImage: /images/openapi-and-swagger-ui-in-spring-boot/main.png
images :
  - /images/openapi-and-swagger-ui-in-spring-boot/main.png
---

　﻿API를 개발하고 사용방법에 대한 명세를 작성하는 방법은 여러 가지가 있다. 가장 심플하게 개발 코드와는 별도로 직접 수기로 작성하여 파일 혹은 문서 링크를 전달하는 방법이 있다. 하지만 개발 코드와 별도로 직접 작성을 한다는 점에서 오타/실수가 발생할 수 있고 최신화가 안되는 여러 가지 문제가 발생한다. 그에 등장한 API 문서화 자동화 툴의 양대 산맥인 SpringRestDocs 와 Swagger.

　﻿과거 [SpringRestDocs 에 대한 포스팅](/2020/03/08/spring-rest-docs-in-spring-boot/)을 했기에 이번엔 Swagger에 대한 사용방법에 대해 정리해보고자 한다. 이 둘의 장단점은 너무 뚜렷하기에 API문서를 제공하는 상황에 따라 적절하게 선택하여 사용할 수 있었으면 좋겠다.

## ﻿SpringBoot에 Swagger 적용
　기본 SpringBoot 가 셋팅되어 있다는 가정하에 Swagger 관련 dependency를 추가해주자. 아참, 이제부터의 프로젝트 셋팅은 Gradle로 하려한다. (물론 Maven으로 해도 무방하지만...)

```gradle
dependencies {
    implementation "io.springfox:springfox-boot-starter:3.0.0"
}
```

　﻿이후 JavaConfig 을 아래와 같이 설정하는데 아래 내용은 아주 기본 세팅이니 자세한 내용은 [공식 도큐문서](https://springfox.github.io/springfox/docs/current/#springfox-spring-mvc-and-spring-boot)를 참고해 보면 좋을 것 같다. (물론 샘플 프로젝트를 만들며 필요할 것 같은 내용은 아래에서 설명하겠다.)﻿

```java
@EnableSwagger2
@Configuration
public class SwaggerConfig {

	@Bean
	public Docket api() {
		return new Docket(DocumentationType.SWAGGER_2)
			.select()
			.apis(RequestHandlerSelectors.any())
			.paths(PathSelectors.any())
			.build();
	}
}
```

　﻿테스트할 컨트롤러를 아래처럼 심플하게 작성하고(사칙연산...) 실행을 시킨 후 `/swagger-ui/`에 접속을 해보면 swagger 관련 javaConfig 하나만 추가했는데 문서가 만들어진 것을 확인할 수 있다. (http method는 편의상 다양하게 작성했으니 왜 DELETE 인가라는 의문은 접어두자.)

```java
@RestController
public class SampleController {

	@GetMapping(value = "/addition")
	public Integer addition(Integer num1, Integer num2) {
		return num1 + num2;
	}

	@PostMapping(value = "/subtraction")
	public Integer subtraction(Integer num1, Integer num2) {
		return num1 - num2;
	}

	@PutMapping(value = "/multiplication")
	public Integer multiplication(Integer num1, Integer num2) {
		return num1 * num2;
	}

	@DeleteMapping(value = "/division")
	public Integer division(Integer num1, Integer num2) {
		return num1 / num2;
	}
}
```

{{< image src="/images/openapi-and-swagger-ui-in-spring-boot/default-config.png" caption="기본 셋팅만 했는데 이런 화면이 나타났다." width="80%" >}}

　﻿위에서 했던 설정들 중 몇 가지만 좀 더 자세히 살펴보자.

| 설정 | 설명 |
| --- | --- | 
| Docket | ﻿Springfox 프레임 워크의 기본 인터페이스가 될 빌더로 구성을 위한 여러 가지 기본값과 편리한 방법을 제공하고 있다. 이후 `select()`로 ApiSelectorBuilder를 반환받아 사용할 수 있도록 해준다. |
| apis | ﻿어떤 위치에 있는 API들을 가져올 것인가에 대한 정의. `RequestHandlerSelectors.any()`이라고 했으니 SpringBoot에서 기본으로 제공하는 `basic-error-controller` 도 API 문서로 만들어진 것을 확인할 수 있다. 특정 패키지만 적용하기 위해서는 `RequestHandlerSelectors.basePackage("com.taetaetae.swagger.api")` 와 같은 형식으로 지정하면 해당 패키지 하위에 있는 Controller를 기준으로 문서를 만들어 준다﻿. |
| paths | ﻿이름에서도 눈치를 챌 수 있듯이 특정 path만 필터링해서 문서를 만들어 준다. |
| useDefaultResponseMessages | ﻿기본 http 응답 코드를 사용해야 하는지를 나타내는 플래그 |

﻿이외에도 security 나 공통으로 사용되는 파라미터 등 다양한 옵션을 설정할 수 있으니 가능하면 상황에 맞게 설정을 변경해 보는 것도 좋을 것 같다. 다른 설정들을 추가시켜서 좀 더 친절하게 만들어 보면 아래처럼 만들 수 있고 해당 코드는 Github에서 확인 가능하다.

{{< image src="/images/openapi-and-swagger-ui-in-spring-boot/change-config.png" caption="API 문서화는 최대한 친절하게!!" width="80%" >}}

## OpenAPI
　﻿Swagger 공식 홈페이지를 이리저리 둘러보면 OpenAPI라는 내용이 많이 나온다. 그렇다면 OpenAPI는 무엇일까? 문서에 나와있는 내용을 직역해보면 Swagger 사양으로 알려져 있으며 RESTful 웹 서비스를 설명, 생성, 소비 및 시각화하기 위한 기계 판독 가능 인터페이스 파일에 대한 사양이라고 한다. 즉, API 자체를 설명하는 인터페이스 스펙이라고 이해를 해볼 수 있다. 위에서 만들어졌던 Swagger를 보면 http://localhost:8080/v2/api-docs?group=Test API 라고 나와있는데 이를 클릭해보면 아래와 같이 json 형태로 보인다. 다시 보면 우리가 이제까지 Swagger으로 만든 API 문서를 설명하는 인터페이스 스펙으로 이해할 수 있다. (꽤 길어서 덧셈 API를 제외하고 지웠다.) 그렇다면 이 JSON 파일을 어떻게 활용할 수 있을까?﻿

```json
{
  "swagger": "2.0",
  "info": {
    "description": "스웨거를 활용해서 API문서를 만들어보아요",
    "version": "1.0.1",
    "title": "사칙연산 API",
    "termsOfService": "https://taetaetae.github.io",
    "contact": {
      "name": "taetaetae",
      "url": "https://taetaetae.github.io/resume",
      "email": "taetaetae_@naver.com"
    }
  },
  "host": "localhost:8080",
  "basePath": "/",
  "tags": [
    {
      "name": "sample-controller",
      "description": "Sample Controller"
    }
  ],
  "paths": {
    "/addition": {
      "get": {
        "tags": [
          "sample-controller"
        ],
        "summary": "덧셈",
        "description": "num1 과 num2 를 더합니다.",
        "operationId": "additionUsingGET",
        "produces": [
          "*/*"
        ],
        "parameters": [
          {
            "name": "num1",
            "in": "query",
            "description": "num1",
            "required": false,
            "type": "integer",
            "format": "int32"
          },
          {
            "name": "num2",
            "in": "query",
            "description": "num2",
            "required": false,
            "type": "integer",
            "format": "int32"
          }
        ],
        "responses": {
          "200": {
            "description": "OK",
            "schema": {
              "type": "integer",
              "format": "int32"
            }
          }
        }
      }
    }
  }
}
```

## SwaggerUI
　﻿이것도 [공식 홈페이지](https://swagger.io/tools/swagger-ui/)를 보면, 별도 구현 로직 없이도 API 리소스를 시각화할 수 있다고 나와있다. 즉, 위에서 했던 것들은 문서화가 포함되어 있는 application 을 별도의 서버로 띄워야 했지만 SwaggerUI를 사용하면 그럴 필요가 없다는 것. 다시 말하면, A라는 application 과 B라는 application 이 있다고 가정했을 때 위에서 했던 방식으로 하면 A, B 각각 서버를 띄워서 API 문서를 시각화할 수 있었지만 SwaggerUI를 사용하면 공통으로 하나의 서버에서 (SwaggerUI 가 운영되는 서버) OpenAPI 스펙에 맞춘 문서만 있으면 모두 표현이 가능하다는 것. 이러면 OpenAPI 스펙에 맞춰서 API 문서를 만들면 한곳에서 볼 수 있으니 너무 편하지 않을까? (특히 MSA 환경에서.) 당장 해보자.﻿

　﻿[도큐먼트](https://github.com/swagger-api/swagger-ui/blob/master/docs/usage/installation.md)에는 NPM Registry, Docker, unpkg 방법을 제시하고 있는데 필자는 Docker로 설치해보고자 한다. (Docker 좀 자주 접해보고 싶어서...) CentOS 환경에서 아무것도 설치되어 있지 않아있다는 가정하에 설치를 해보면 다음과 같은 순서로 진행된다.﻿

```markdown
# yum 최신 업데이트
sudo yum update

# docker 설치 및 시작
sudo yum install docker
sudo systemctl start docker

# swagger-ui docker 이미지 pull 
sudo docker pull swaggerapi/swagger-ui

# docker 이미지 run, 백그라운드 모드로, 
docker run -d -p 80:8080 -e SWAGGER_JSON=/mnt/api.json -v /home/test:/mnt swaggerapi/swagger-ui
```

　﻿앞서 json 을 `/home/test`하위에 `api.json`이라는 파일로 만들어두고 이미지를 띄운 뒤 서버의 80 port로 접근을 해보면 아래와 같이 앞서 직접 서버를 띄워서 했던 것처럼 동일하게 보인다. (wow)

{{< image src="/images/openapi-and-swagger-ui-in-spring-boot/swagger-ui-docker.png" caption="한 곳으로 모을수 있다는 큰 장점이 있다." width="80%" >}}

## 마치며
　﻿사실 Swagger를 SpringBoot에 적용하기까지만 포스팅하려다 SwaggerUI까지 해보게 되었다. 새로운 기술이 있으면 감으로만 수박 겉핥기 식으로 보는 것보다 직접 내가 해보았는지가 정말 중요한 것 같다. SpringRestDocs 보다 훨씬 빠르고 가벼운 설정으로 문서화가 가능했지만 테스트 코드 없이 검증되지 않은 API 문서가 만들어진다는 불안함은 감출 수 없었다. 이게 정답이다 할 건 아니지만 둘 다 각자의 장점이 뚜렷하기 때문에 상황에 따라 잘 선택해서 사용해야 하겠다.