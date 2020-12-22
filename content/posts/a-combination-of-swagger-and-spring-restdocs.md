---
title: Swagger와 Spring Restdocs의 우아한 조합 (by OpenAPI Spec)
date: 2020-12-22T10:41:40+09:00
categories:
  - tech
tags:
  - Swagger
  - Swagger-ui
  - OpenAPI
  - archives-2020
featuredImage: /images/a-combination-of-swagger-and-spring-restdocs/main.jpg
images :
  - /images/a-combination-of-swagger-and-spring-restdocs/main.jpg
---
　﻿MSA 환경에서의 API 문서화는 어떤 식으로 구성하는 걸까? 예컨대, 모듈이 10개 있다고 하면 각 모듈마다 API 문서가 만들어질 테고 API 문서를 클라이언트에 제공하기 위해서 각각의 (10개의) URL를 전달해야 할 텐데 이게 과연 효율적일까? 물론 기능별로 URL이 분리된다는 장점이 있고 굳이 모아보자면 각 API 문서를 다시 한번 크롤링 하여 검색할 수 있도록 제공하는 것도 하나의 방법이 될 수 있다. 하지만 이러한 방법들은 요구 사항을 위한 별도의 작업을 하게 되니 일을 위한 일이 되는 것 같아 뭔가 아쉬웠다. 좋은 방법이 없을까?

## 고민의 시작

{{< image src="/images/a-combination-of-swagger-and-spring-restdocs/line-1.png" caption="﻿닉네임이나 프로필 사진은 그들의 개인 정보를 위해 임의로 지정하였다." width="80%" >}}

　﻿필자와 함께 개발자의 인생을 시작한 멋진 친구들에게 정확히 올해 6월 초에 고민을 털어놓으며 좋은 방법이 없을지에 대한 논의를 했던 적이 있다. 그런데 친구 중 한 명이 잊고 있었던 그 이슈에 대해서 다시 꺼내며 URL 하나를 던져준다. 참 고마운 친구들. 
> thanks to 34. [asuraiv](http://asuraiv.tistory.com/), [black9p](https://black9p.github.io/) 

{{< image src="/images/a-combination-of-swagger-and-spring-restdocs/line-2.png" caption="언 반년이 지났으나 필자도 잊고 있었던 이슈를 그는 기억하고 있었다." width="40%" >}}

　﻿올해 NHN FORWARD에서 진행했던 세션 중에서 [MSA 환경에서 API 문서 관리하기: 생성부터 배포까지](https://forward.nhn.com/session/3)라는 제목의 내용이었고, 정확하게 필자가 고민했던 부분을 콕! 집어서 해결해 준 사례였다. 역시 세상엔 엄청난 고수들이 내가 고민했던 부분들을 이미(혹은 이후에라도) 고민하고 해결한 경우가 많다는 것을 느끼고 공유의 힘이 이렇게도 대단하구나 하며 놀라움을 금치 못하였다.﻿

{{< youtube qguXHW0s8RY >}}

　﻿이번 포스팅에서는 OpenAPI Spec 을 활용하여 Spring Restdocs로 만들어지는 문서를 Swagger UI에서 보는 흐름을 실제로 구현해 보고자 한다. 즉, Swagger 나 Spring Restdocs 뭐로 만들든 간에 OpenAPI Spec에 맞춰서만 만든다면 한곳에서 볼 수 있겠다는 희망이 보였다. 며칠 전 작성한 [OpenAPI 와 Swagger-ui 포스팅](/posts/openapi-and-swagger-ui-in-spring-boot/)을 본 독자들은 지금의 포스팅을 작성하기 위한 밑거름이었다는 사실을 눈치챘을 수도 있을 것 같다.

> ﻿좋은 내용을 공유해 주신 (저의 고민을 완벽하게 해결해 주신) NHN FORWARD 발표자분께 이 포스팅을 빌어 감사의 인사를 보냅니다. :) 당장 팀 내에도 적용해봐야겠어요!!

## ﻿Spring Restdocs에서 OpenAPI Spec 추출
　﻿누가 또 친절하게 오픈소스로 만들어놨다. https://github.com/ePages-de/restdocs-api-spec 에서 관련 내용을 확인할 수가 있는데 해당 링크에서는 gradle 버전이고 https://github.com/BerkleyTechnologyServices/restdocs-spec 는 maven 버전이라고 한다. 마침 필자의 Github에 Maven 버전으로 SpringRestdocs를 세팅해둔 [Repository](https://github.com/taetaetae/SpringRestDocs-In-SpringBoot) 가 있어서 이를 활용해보고자 한다.﻿

### pom.xml 추가
　﻿관련 dependency를 추가하자. jcenter라고 bintray.com 에서 운영되는 Maven Repository에 올려진 오픈소스이니 repository 도 추가해 주자.

```xml
<properties>
	<restdocs-api-spec.version>0.10.0</restdocs-api-spec.version>
	<restdocs-spec.version>0.19</restdocs-spec.version>
</properties>

<repositories>
	<repository>
		<id>jcenter</id>
		<url>https://jcenter.bintray.com</url>
	</repository>
</repositories>

<dependency>
	<groupId>com.epages</groupId>
	<artifactId>restdocs-api-spec</artifactId>
	<version>${restdocs-api-spec.version}</version>
	<scope>test</scope>
</dependency>

<dependency>
	<groupId>com.epages</groupId>
	<artifactId>restdocs-api-spec-mockmvc</artifactId>
	<version>${restdocs-api-spec.version}</version>
	<scope>test</scope>
</dependency>
```

﻿위의 dependency에서 제공해 주는 모듈로 테스트의 SpringRestdocs를 만들었다면 OpenAPI Spec 을 만들어 주는 plugin 또한 추가해 주자

```xml
<pluginRepositories>
	<pluginRepository>
		<id>jcenter</id>
		<url>https://jcenter.bintray.com</url>
	</pluginRepository>
</pluginRepositories>

<plugin>
	<groupId>com.github.berkleytechnologyservices.restdocs-spec</groupId>
	<artifactId>restdocs-spec-maven-plugin</artifactId>
	<version>${restdocs-spec.version}</version>
	<executions>
		<execution>
			<goals>
				<goal>generate</goal>
			</goals>
			<configuration>
				<specification>OPENAPI_V3</specification>
				<format>JSON</format>
				<outputDirectory>${project.build.directory}/classes/static/docs</outputDirectory>
			</configuration>
		</execution>
	</executions>
</plugin>
```

﻿위 plugin 설정을 보면 format 을 JSON으로 한 것을 볼 수 있는데 YAML로도 만들 수 있다. 자세한 사용방법은 위에서 명시한 링크를 참고해보는 게 좋을 것 같다.

### 문서화 로직 추가
　﻿기존에 SpringRestdocs를 작성하는 로직은 `org.springframework.restdocs.mockmvc.MockMvcRestDocumentation`에서 제공해 주는 메서드를 사용했지만 위에서 이야기 한 오픈소스를 사용하기 위해 `com.epages.restdocs.apispec.MockMvcRestDocumentationWrapper`를 사용하도록 하자. 변경을 최소화하기 위해 import만 변경하도록 한다.

```java
//import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.*;
import static com.epages.restdocs.apispec.MockMvcRestDocumentationWrapper.*;
```

﻿위와 같이 설정하고 Maven 빌드를 해보면 plugin에서 지정한 경로에 JSON 파일이 생성된 것을 확인할 수 있다.

{{< image src="/images/a-combination-of-swagger-and-spring-restdocs/generate-openapi-json.png" caption="경로 지정은 자유. 모든 설정은 변경이 가능하다." width="80%" >}}

만들어진 JSON 파일은 아래와 같다.

```json
{
  "openapi" : "3.0.1",
  "info" : {
    "title" : "spring-rest-docs",
    "description" : "Demo project for Spring Boot",
    "version" : "0.0.1-SNAPSHOT"
  },
  "servers" : [ {
    "url" : "http://localhost"
  } ],
  "tags" : [ ],
  "paths" : {
    "/book/{id}" : {
      "get" : {
        "tags" : [ "book" ],
        "operationId" : "book",
        "parameters" : [ {
          "name" : "id",
          "in" : "path",
          "description" : "book unique id",
          "required" : true,
          "schema" : {
            "type" : "string"
          }
        } ],
        "responses" : {
          "200" : {
            "description" : "200",
            "content" : {
              "application/json" : {
                "schema" : {
                  "$ref" : "#/components/schemas/book-id416760944"
                },
                "examples" : {
                  "book" : {
                    "value" : "{\"id\":1,\"title\":\"spring rest docs in spring boot\",\"author\":\"taetaetae\"}"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "components" : {
    "schemas" : {
      "book-id416760944" : {
        "type" : "object",
        "properties" : {
          "author" : {
            "type" : "string",
            "description" : "author"
          },
          "id" : {
            "type" : "number",
            "description" : "book unique id"
          },
          "title" : {
            "type" : "string",
            "description" : "title"
          }
        }
      }
    }
  }
}
```

## Swagger UI 연동

　﻿자. 이제 오픈소스를 이용해서 만들어진 OpenAPI Spec JSON 파일을 앞서 경험해본 Swagger UI에 적용해볼 차례이다. 해당 파일을 적당한 위치에 두고 아래처럼 docker 명령을 재실행해서 Swagger로 만들어진 문서, SpringRestdocs로 만들어진 문서를 다 같이 띄워보자. 각 JSON 파일들은 `/home/data`하위에 위치시킨다.

```shell
docker run -d -p 80:8080 \
  -e URLS_PRIMARY_NAME=SpringRestdocs \
  -e URLS="[ \
    { url: 'docs/swagger.json', name: 'Swagger' } \2
    , { url: 'docs/restdocs.json', name: 'SpringRestdocs' } \
  ]" \
  -v /home/data:/usr/share/nginx/html/docs/ \
  swaggerapi/swagger-ui 
```

도커에 익숙하지 않는 독자들을 위해 (그중에 필자도 포함...) 명령어를 분석해보면 다음과 같다.

| 구문 | 설명 |
| --- | --- | 
| -d | 백그라운드 모드로 실행 |
| -p 80:8000 | 포트 바인딩. 외부에서 80으로 접속하면 내부 8080으로 바인딩 시킨다. |
| -e URLS_PRIMARY_NAME=SpringRestdocs | 첫번째 보여질 URL 이름의 환경변수 |
| -e URLS="" | Swagger UI 에서 보여줄 API 정의 문서 환경변수. 여러개인 경우 JSON 배열형식으로 표현한다. |
| -v /home/data:/usr/share/nginx/html/docs/ | 볼륨 바인딩. 로컬의 `/home/data` 를 도커 내부의 `/usr/share/nginx/html/docs/` 으로 바인딩 시킨다. |
| swaggerapi/swagger-ui | 해당 이미지를 띄운다. |

도커에 대한 보다 자세한 설명은 [도커 공식문서](https://docs.docker.com/)를 보는게 좋고 Swagger UI 의 옵션에 대한 내용 역시 [Swagger UI 공식문서](https://swagger.io/docs/open-source-tools/swagger-ui/usage/configuration/)를 참조하는게 좋을것 같다.

　이렇게 띄우고 나면 어떤 모습일까? 백문이 불여일타. 실행해보고 화면을 살펴보자.

{{< image src="/images/a-combination-of-swagger-and-spring-restdocs/swagger-ui.png" caption="여러 API 문서를 한곳에서!" width="80%" >}}

﻿좌측은 처음 접속했을 때의 화면. 위에서 설정한 대로 SpringRestdocs로 만들어진 API 문서가 먼저 보이고 SpringRestdocs에서 만들어졌던 내용들이 그대로 Swagger UI에서도 동일하게 볼 수 있다. 우측 상단에 셀렉트 박스에서 Swagger로 만들어진 API 문서를 선택해보면 당연히 얼마 전에 만들었던 Swagger로 만든 API 문서를 확인할 수 있다.

## 마치며　﻿
　이렇게 되면 SpringRestdocs의 장점인 `테스트를 통과한 검증된 API의 문서` 와 Swagger의 장점인 `Web UI에서 테스트 가능` 한 두마리 토끼를 다 잡은 결과를 만들 수 있었다. 또한 필자가 고민했던 `MSA 환경에서 여러 곳에 흩어진 API 문서를 한곳에서 모으는 효과`까지. 일석삼조의 효과를 경험할 수 있어서 너무 좋았다.

﻿　더불어 Swagger 와 SpringRestdocs는 약간 `라이벌`처럼 각자의 장단점이 있다 정도로만 생각했는데 이 둘을 상황에 맞춰 조합을 하니 오히려 `시너지효과`를 얻을 수 있던 점 또한 너무 좋았다.

　고민은 누구나 하지만 결국 끈기 있게 파고들어 해결했는가가 더 중요하다는 점을 다시 한번 느낄 수 있었다.

> '꿩 먹고 알 먹고 둥지 털어 불 때고', '도랑 치고 가재 잡고 발 담그고 물 구경 하고', '마당 쓸고 동전 줍고', '누이 좋고 매부 좋고', '님도 보고 뽕도 따고'. '포스팅하고 내 것으로 만들고'