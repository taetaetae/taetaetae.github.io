---
title: 스프링을 활용한 대용량 파일 업로드 구현
date: 2019-07-21 22:09:58
categories:
  - tech
tags: 
  - fileupload
  - spring
url : /2019/07/21/spring-file-upload/
featuredImage: /images/spring-file-upload/upload.png

---
개발을 하다보면 실제로 직접 구현을 해본적은 없지만 여기저기서 들어본 지식과 그 동안의 짬밥(?)으로 추측해볼수 있는 부분들이 있다. 물론 모든일에 정답은 없겠지만 요즘 느끼는건 책에서 공부만 해본것과 다른 블로그들에서 눈으로만 보고 넘어가는것들 그리고 직접 손가락을 움직여가며 왜 여기서는 이 방법을 사용하지 고민하면서 구현을 해본다는건 정말 엄청나게 큰 차이가 있는것 같다. <!--more -->
웹 어플리케이션을 개발하다보면 한번 쯤 만나게 되는 `파일 업로드` 기능. 필자도 몇번 구현은 해봤지만 그냥 단순히 `구현`만 해본 상태였다가 최근에 그냥 파일 업로드가 아닌 `대용량` 파일 업로드에서의 문제가 발생하여 여기저기 삽질을 하게 되었고 정리도 해볼겸 스프링에서의 대용량 파일 업로드시 한번쯤 고려해봐야 할 부분에 대해 정리를 해보려고 한다.
> 물론 구글에서 검색을 해보면 아마 필자가 쓴것 보다 더 자세하고 좋은 글들이 있겠지만 필자는 보다 `대용량`에 집중에서 작성해 보고자 한다. 명심하자. "아무리 흐린 잉크라도 좋은 기억력보다 낫다" 라는 말이 있듯이

## 스프링을 활용한 파일 업로드 구현
우선 완전 초기상태에서 시작하기 위해 스프링 부트 프로젝트를 만들고 간단하게 파일 업로드를 할 수 있는 form 페이지와 업로드 버튼을 눌렀을때 작동하게 되는 컨트롤러를 만들어 보자.
```java
import java.io.File;
import java.io.IOException;
import java.io.InputStream;

import org.apache.commons.io.FileUtils;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Controller
public class FileUploadController {

	// 너무 간단 ...
	@RequestMapping("/form")
	public String form() {
		return "form";
	}

	@RequestMapping(value = "/upload", method = RequestMethod.POST)
	public String upload(@RequestParam("file") MultipartFile multipartFile) {
		log.info("### upload");
		File targetFile = new File("/home1/irteam/" + multipartFile.getOriginalFilename());
		try {
			InputStream fileStream = multipartFile.getInputStream();
			FileUtils.copyInputStreamToFile(fileStream, targetFile);
		} catch (IOException e) {
			FileUtils.deleteQuietly(targetFile);
			e.printStackTrace();
		}
		return "redirect:/form";
	}
}

```
`upload` 요청이 들어오면 `file`이라는 이름의 파라미터로 `MultipartFile`을 받고 파일의 이름을 확인 후 스트림을 읽어 특정 경로에 파일로 저장하는 로직이다. 그다음 `/form`을 접속하게 되면 나오는 폼 화면을 만들자. 이것도 아주 심플하게!
```html
<h1>파일 업로드</h1>
<form action="/upload" method="post" enctype="multipart/form-data">
	<input type="file" value="파일 선택" name="file"/>
	<input type="submit" value="업로드"/>
</form>
```
`multipart/form-data` 라는 Content-Type 을 명시해주고 파일을 선택하면 `/upload`로 POST요청을 하도록 설정한다. 이렇게 되면 너무 간단하게 + 이상없이 파일이 업로드가 잘 되니 이게 이야기 할 꺼리인가(?) 싶을정도로 심플하다.

## 그런데 파일 크기가 크다면?

{{< image src="/images/spring-file-upload/omg.jpg" caption="설마 파일 업로드 하는 용량이 크겠어?... 왠지 파일의 용량이 크면 문제있을것 같은데... <br>출처 : https://m.blog.naver.com/naibbo0407/30170815180" width="80%" >}}

개발을 하다보면 항상 생각해야 할 부분중에 하나가 바로 확장성인것 같다. 이 부분에서 역시 문제가 되었던 것. 평소보다 용량이 큰 파일이 업로드가 되면서 (평소 3~400MB 였다가 3~4GB정도의 파일이 업로드가 되는 매직) 업로드가 안되는 상황이 발생하였다. 당연히 문제가 발생하면 누군가 말했듯 로그부터 살펴보았는데 Apache - (AJP) - Tomcat 으로 구성된 환경에서 tomcat 로그에 `### upload`라는 로그가 없고 아파치 로그엔 `502 에러`가 발생한 것이었다. 왜 톰켓 로그도 안남고 그전에 에러가 발생하였을까?
이때부터 (근거없는 추측을 하며...) 고난과 역경의 삽질을 하기 시작하게 된다. 톰켓 버전이 문제일까? 로그가 안찍혔다면 다른 필터나 인터셉터에서 무언가를 먹고(?)있는건 아닐까? 잠깐, 근데 원래 대용량 업로드가 되긴 해? 파일 업로드/다운로드 하는 사이트 보면 별도 프로그램으로 하던데... 꼬불꼬불 미로속을 헤메는것만 같았던 삽질의 문제는 결국 `메모리`에 있었다.
파일을 업로드 하게 되면 해당 내용을 우선 메모리에 담게 되고 다 담은 후 메모리에 있는 내용을 was에 전달한 뒤 HttpServletRequest 로 넘어오게 된다.(Apache > Tomcat) 그런데 파일을 업로드 하면서 메모리에 파일이 써지다가 메모리 부족으로 OOM이 발생하게 되버린 것이었다. 또한 스프링 파일 최대크기를 별도로 지정하지 않고 있었기 때문에 메모리가 충분했다 하더라도 에러가 발생했을 상황이었다. ( https://spring.io/guides/gs/uploading-files/ 참조 )
```
spring.servlet.multipart.max-file-size
spring.servlet.multipart.max-request-size
```
그러다보니 웹 서버인 아파치에서는 was 에러인 `503`이 아닌 `502`라고 에러를 발생하던 것이였고 지나고 보면 정말 아무것도 아닌 간단한 설정들을 놓친 문제였는데 꽤나 긴 시간을 허비해야만 했던 안타깝지만 보람찼던 (응?) 트러블 슈팅이었다.

## 삽질은 곧 경험이 되고 시야가 된다.
legacy 로직이다보니 was가 파일 업로드 처리를 하게 되었는데 가급적이면 was가 처리하는것 보다는 static 파일을 처리할 수 있는 별도의 웹서버를 만드는게 어떨까 생각이 든다. (조금 알아보니 nodejs 모듈인 [multer](https://github.com/expressjs/multer) 라는게 있다.) 물론 파일 업로드 한 뒤에 별도의 로직을 처리하려면 was가 관여를 해야겠지만 이 부분은 설계를 어떻게 하냐에 따라 충분히 해결할 수 있을것으로 보인다. (웹서버에서 파일을 업로드 한 뒤 비동기로 파일 업로드 완료여부에 따라 was에서 처리를 한다거나 등...)
더불어 항상 어플리케이션을 만들때에는 `예외처리`라는 것을 생각하면서 개발해야한다고 느끼게 되었다. NPE 같은 사소한 로직에서의 예외처리부터 파일 업로드시 서버의 메모리를 생각할수 있는 시야. 이런게 경험이 아닐까 싶다.
또한 (잘 돌아가니까) 환경설정 값을 수정하지 않고 배포하는 것보단 가급적 어떤 설정값들에 의해서 어플리케이션이 돌아가는지 특히, 스프링 같은 프레임워크의 도움을 받는다면 해당 프레임워크의 설정값들을 수정하며 성능에 이득을 취할 부분들은 없는지 꼼꼼하게 개발하는 습관을 길러야 할 것 같다.