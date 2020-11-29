---
title: 스프링 부트로 멀티모듈 셋팅하기
date: 2020-01-19 10:29:03
categories:
  - tech
tags: 
  - spring boot
  - module
  - structure
  - maven
  - archives-2020
url : /2020/01/19/spring-boot-maven-multi-module/
featuredImage: /images/spring-boot-maven-multi-module/multimodule.png
images :
  - /images/spring-boot-maven-multi-module/multimodule.png
---

서비스를 처음 만들기 시작할때면 각 직군별로 생각하는 포인트가 다양하다. 설계, 기획, 디자인, 개발. 여기서 개발은 프로젝트 셋팅을 어떻게 해야하지? 하는 고민을 하기 마련이다. 아주 간단하게 하나의 모듈로 모든 기능을 담당하도록 만들 수 있지만 기능별로 모듈을 나눠서 셋팅하는게 관리측면에서 장점이라 생각한다.<!--more -->

예를 들어보자. 도서관의 들어온 책 정보를 외부에 제공하는 "API", 주기적으로 책 정보를 업데이트 하는 "Batch". 이렇게 크게 두가지의 모듈이 있어야 한다고 가정했을때 어떤식으로 모듈을 설계할 수 있을까? 

이번 포스팅에서는 스프링 부트와 메이븐을 활용해서 하나의 프로젝트(컴포넌트)에서 여러 모듈을 관리할 수 있는 [Spring Multi Module](https://spring.io/guides/gs/multi-module/)을 셋팅하는 방법에 대해 알아보고자 한다. 필자도 셋팅하기 전에는 "그냥 하면 되는거 아니야?"라며 우습게 보다 아주 사소한 부분들에서 엄청난 삽질을 해서 그런지 꼭 포스팅으로 남겨놔야 겠다고 다짐했고 이렇게 정리를 할 수 있게 되어서 다행이라 생각한다.

{{< image src="/images/spring-boot-maven-multi-module/team_structure.png" caption="어쩌면 우리가 있는 팀도 멀티모듈이 아닐까? <br>출처 : https://bcho.tistory.com/813" width="80%" >}}

## 왜 멀티모듈로 셋팅할까?
위에서 예시로 이야기 한것처럼 현재 우리가 셋팅해야할 모듈은 크게 두가지 이다.
- API : 외부에 도서관에 들어온 책 정보를 알려주는 모듈
- Batch : 주기적으로 도서관의 책 정보를 갱신하는 모듈

한번 생각을 해보자. 위에서 말한 모듈들 중에 동시에 사용할것만 같은 정보가 있다. "책 정보". 각 모듈마다 "책 정보"를 가져오는 로직을 작성하는것 보다 한곳에서 해당로직을 구현하고 이를 여러곳에서 사용하는게 사용하는게 중복코드를 방지할수 있는 방법이란건 쉽게 알아차릴수 있다. 그렇다면 어떻게 모듈을 분리할수 있을까?

필자의 경험으로 미루어 볼때 크게 두가지 방법이 있는것 같다. 
- 공통으로 사용하는 모듈을 jar로 만들고 이를 메이븐 원격 저장소에 deploy, 사용하는 모듈에서 디펜던시에 추가하여 사용
- 멀티모듈로 구성하고 사용하는 모듈에서 디펜던시에 추가하여 사용

첫번째 방법의 가장 큰 단점은, 공통으로 사용하는 모듈이 변경될때마다 버전을 바꿔주고 (안바꿔도 되지만 사용하는 모듈에서 캐시 갱신을 해야하는 불편함이 생긴다.) 메이븐 원격 저장소에 deploy를 해줘야 한다. 그에 반해 두번째 방법은 이런과정없이 함께 빌드만 해주면 끝나고 IDE에서 개발시 한 모듈에서 동시에 수정과 사용이 가능하기 때문에 훨씬 편리하다.

[은총알은 없다](https://johngrib.github.io/wiki/No-Silver-Bullet/) 라는 말처럼, 정답은 없다. 하지만 이런저런 방법들을 미리 알아두면 적시적소에 사용할 수 있는. 필자가 다른글들에서도 언급을 자주하던 "나만의 무기"가 되지 않을까?

## 멀티모듈 셋팅하기
위에서 이야기 했던 "API", "Batch"와는 별도로 공통으로 사용하는 모듈인 "Core" 이렇게 총 3개의 모듈을 만들예정이다.
> 다른 이야기지만, 공통으로 사용할 것 "같아서" 미리 공통로직을 작성하는 습관은 좋지 않는것 같다. 그러다보면 쓸데없이 공통로직이 무거워지므로 실제로 사용하면서 중복코드가 발생할때 그때 공통로직으로 리펙토링 해도 늦지 않는것 같다. (꼰데인가...)

구현하는 환경은 다음과 같다.
- Spring Boot 2.2.3
- Maven
- IntelliJ

우선 IDE의 힘을 빌려 하나의 스프링 부트 프로젝트를 생성해본다.

{{< image src="/images/spring-boot-maven-multi-module/spring_boot_init.jpg" caption="다음 > 다음 > 다음" width="80%" >}}

그 다음 만든 프로젝트에서 우클릭 후 새로운 모듈을 선택. Maven 모듈을 선택하고 적당한 이름을 적어준다.
{{< image src="/images/spring-boot-maven-multi-module/new_module.jpg" caption="다음 > 다음 > 다음 222" width="80%" >}}

"API", "Batch", "Core" 라는 모듈을 추가하고 실제 모듈이 되는 "API", "Batch"에 Build plugin 을 셋팅해주자. 그렇게 하고 각 Pom.xml을 보면 아래와 같다. ("API" 모듈에 대해서만 집중적으로 이야기 하려 한다. "Batch" 모듈도 동일한 형식으로 작성하기 때문.)

- 최 상위 Pom.xml (library)
modules 하위에 멀티모듈로 설정한 모듈들의 이름이 들어가 있는것을 확인할 수 있다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<packaging>pom</packaging>
	<modules>
		<module>api</module>
		<module>core</module>
		<module>batch</module>
	</modules>
	
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<groupId>com.taetaetae</groupId>
	<artifactId>library</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>library</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	</dependencies>
</project>
```

- API Pom.xml
parent 부분이 설정되어 있는 모습을 볼수 있고, core 모듈을 사용하기 위해 dependency 에 추가를 해준다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<parent>
		<artifactId>library</artifactId>
		<groupId>com.taetaetae</groupId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>
	<modelVersion>4.0.0</modelVersion>
	<packaging>jar</packaging>
	<artifactId>api</artifactId>

	<dependencies>
		<dependency>
			<groupId>com.taetaetae</groupId>
			<artifactId>core</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
	</dependencies>

	<build>
		<finalName>library-api</finalName>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

- Core Pom.xml
Core 모듈은 "jar"로 패키징 되어 다른 곳에서 사용되어야 하기 때문에 packaging 만 설정해준다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<parent>
		<artifactId>library</artifactId>
		<groupId>com.taetaetae</groupId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>
	<modelVersion>4.0.0</modelVersion>
	<packaging>jar</packaging>
	<artifactId>core</artifactId>

	<version>0.0.1-SNAPSHOT</version>

</project>
```

이렇게 하고서 Core 모듈에 공통으로 사용될 로직을 작성하고, API 모듈에서 이를 사용하는 로직을 작성한뒤, 빌드를 해보면 에러 없이 정상 작동을 하는 모습을 볼 수 있다.
- 메이븐 빌드 Goal : mvn clean install -pl api -am

> -pl [ ] : 지정된 이름의 모듈만 빌드한다.\
-am : 연결된 상위 모듈까지 같이 빌드한다.\
[Reference 참고](https://books.sonatype.com/mvnref-book/reference/_using_advanced_reactor_options.html#_specifying_a_subset_of_projects)

```markdown
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO] 
[INFO] library                                                            [pom]
[INFO] core                                                               [jar]
[INFO] api                                                                [jar]
[INFO] 
[INFO] -----------------------< com.taetaetae:library >------------------------

...중략...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for library 0.0.1-SNAPSHOT:
[INFO] 
[INFO] library ............................................ SUCCESS [  0.715 s]
[INFO] core ............................................... SUCCESS [  3.005 s]
[INFO] api ................................................ SUCCESS [  1.715 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  6.008 s
[INFO] Finished at: 2020-01-19T11:26:46+09:00
[INFO] ------------------------------------------------------------------------
```
본문 최 하단에 해당 소스가 업로드 된 Github에서 확인이 가능하겠지만, ("도서관" 이라는 목적에는 안맞지만...) 정수를 더하고 빼는 유틸을 Core에 만들고 이를 Api에 있는 컨트롤러에서 사용하도록 만들어 보았다. (어디까지나 모듈에서 멀티모듈로 되어있는 다른 모듈에 접근이 가능한지를 보기 위함이라... 예시가 우아하진 않다.)

### # 마치며
언제부터인가 단순 로직개발보다 구조관점에서 바라보는 연습을 하곤한다. 아무리 알고리즘이 잘 작성되고 우아한 코드일지라도 구조가 개발 생산성 측면과 유지보수 측면에서 분리하면 아무 소용 없는것 같다. 위에서도 이야기 했듯이 멀티모듈이 무조건적인 정답은 아니지만 시스템을 구성하는데 있어 다양한 선택지를 알고있는 여유를 갖는것도 좋아보인다.

※ [Github 예제 소스](https://github.com/taetaetae/spring-boot-maven-multi-module)