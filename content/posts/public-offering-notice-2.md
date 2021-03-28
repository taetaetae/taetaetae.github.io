---
title: 공모주 알리미 개발 후기 - 2부
date: 2021-03-28T11:41:33+09:00
categories:
  - tech
tags: 
  - public-offering-notice
  - telegram
  - bitly
featuredImage: /images/public-offering-notice-2/logo.jpg
images :
  - /images/public-offering-notice-2/logo.jpg
---

　﻿혹시 이 포스트를 처음 읽는 독자라면 [지난 포스팅](/posts/public-offering-notice-1)을 읽고 오는 것을 추천한다. 정리하자면, 지난 포스트에서는 [토이 프로젝트](https://t.me/PublicOfferingNotice)를 시작하게 된 계기와, 어떤 식으로 만들지에 대한 설계. 그리고 데이터를 수집하는 과정에 대해 이야기했었다. 지난 포스팅에서 수집한 데이터를 이제 사용자들에게 알려주는 부분에 대해 정리하고자 한다.

- 1부 : [프로젝트 설계, 데이터 수집](/posts/public-offering-notice-1)
- 2부 : [수집한 데이터 알림 발송](/posts/public-offering-notice-2)
- 3부 : 서버 선정 및 릴리즈

## 데이터 정의
　﻿java 라이브러리 중에 jsoup라는 것을 사용하여 웹사이트를 크롤링 하였고, 필요한 데이터를 파싱을 하였다. 아래는 '공모주'라는 자바 모델을 정의해 보았다. 이렇게 자바 '모델'로 정의를 하는 이유는 필요한 데이터가 무엇인지 다시 한번 정리를 하기 위함이기도 하고 map 같은 형태의 임시 변수(?)보다 더 직관적이기에 이후 코드를 작성하는데 가이드 역할의 효과도 얻을 수 있을 것 같았기 때문이다.
```java
public class PublicOffering {

  private String name; // 종목명
  private LocalDate startDate; // 일정 시작일
  private LocalDate endDate; // 일정 마감일
  private LocalDate listingDate; // 상장일
  private String publicOfferingPrice; // 확정 공모가
  private String expectedOfferingPrice; // 희망 공모가
  private List<String> Underwriter; // 주간사
  private String detailUrl; // 상세URL
  private String competitionRate; // 청약경쟁률
}
```

　﻿초기에는 위에서 정의한 모델처럼 공모주의 기본 정보만을 서비스해야겠다 생각했고, 관련 뉴스라든지 기타 추가적인 정보나 다른 분들의 요구 사항(?)들이 추가될 경우 점진적으로 설계를 하고서 확장시켜 나가는 방향으로 계획했다. 우선은 기능들이 부족하더라고 돌아가는 서비스를 만들고 싶었기에.

{{< image src="/images/public-offering-notice-2/agile.png" caption="애자일 방법론!<br> 출처 : https://m.blog.naver.com/keycosmos3/221267522930" >}}

## 텔레그램 봇/채널 생성
　﻿텔레그램 봇을 만드는 과정은 가볍게 검색을 해보면 너무나 쉽게 찾을 수 있지만, 보다 하나의 글 안에 모든 내용을 담고 싶어 텔레그램 봇을 만들고 → 텔레그램 채널을 만든 다음 → 텔레그램 봇을 이용해서 텔레그램 채널에 메시지를 보내는 걸 이야기해 보고자 한다.

> ﻿아, 여기서 왜 꼭 '텔레그램'을 선택했는가에 대한 이유는 개인적으로 다른 메신저 (카카오톡, 라인 등)보다도 api를 활용하여 메시지를 보내는 과정이 단순하면서도 빠르고 쉽게 느껴졌기 때문이다. 혹시 텔레그램 이 아닌 다른 메신저로 보내달라는 요청이 있을 경우 그때 가서 고민해 보려 한다.

- 텔레그램 봇 생성

　﻿먼저 텔레그램 메신저에서 'BotFather'라는 사용자를 찾고 '/start'를 누르면 아래와 같이 사용할 수 있는 명령어가 나온다.

　﻿그다음 우리는 봇을 만들 것이기 때문에 '/newbot'을 누르고 봇의 이름을 작성하고 그 봇의 사용자 이름을 지정한다. '\_bot'으로 끝나야 한다고 하기에 이름 뒤에 붙여서 만들면 그걸로 끝. 다음으로 친절하게 HTTP API를 사용할 수 있는 토큰이 발급되는데 이 토큰으로 봇을 컨트롤 가능하기 때문에 잘 간직하고(?) 있어야 한다.

{{< image src="/images/public-offering-notice-2/newbot.jpg" caption="친절한 봇 아버지" width="70%" >}}

　﻿이후 해당 토큰을 이용해서 봇의 상태를 확인해보자. 아래의 url에 토큰 경로만 변경하여 입력하면 json 응답을 받을 수 있다.
```markdown
https://api.telegram.org/bot{token}/getUpdates
e.g. https://api.telegram.org/bot17...42:AAH...cQU/getUpdates
```

- 텔레그램 채널 생성

　﻿1:N으로 채널에 가입한 사람들에게 메시지를 일방적으로 보내야 하기 때문에 사용할 채널을 만들어 보자. 텔레그램 UI만 봐도 간단하게 생성하기 쉽게 되어있다. 더불어 이 채널에 메시지를 보내야 하기 때문에 위에서 만들었던 봇을 추가하고 관리자로 승격 시키자.

{{< image src="/images/public-offering-notice-2/newchannel.jpg" caption="채널 생성 > 봇을 관리자로" width="70%" >}}

- ﻿텔레그램 봇으로 텔레그램 채널에 메시지 보내기

　이 부분에서 약간 헤맸는데 결국 위에서 얻은 토큰과 채널의 특정 id를 알아야 메시지를 보낼 수 있다. 앞서 만들었던 채널에 아무 메시지나 작성을 하고 위에서 호출했던 'getUpdates' api를 다시 호출해보면 아래처럼 채널의 id를 구할 수 있게 된다.

{{< image src="/images/public-offering-notice-2/channelid.jpg" caption="메세지를 작성하지 않으면 데이터가 안나는 경우가 있었다." >}}

　﻿마지막으로 아래처럼 'sendMessage' api를 활용해서 메시지를 보내보자. 이 외에 텔레그램의 공식 API 문서를 확인해 보는 게 가장 정확하고 다른 기능들도 둘러보는 것을 추천한다.
```markdown
https://api.telegram.org/bot{token}/sendMessage?chat_id={channelId}&text={text}
e.g. https://api.telegram.org/bot17...42:AAH...cQU/sendMessage?chat_id=-10...33&text=helloWorld
```

{{< image src="/images/public-offering-notice-2/helloworld.jpg" caption="개발의 첫 시작은 언제나 hello world" width="70%" >}}

## URL 인코딩 이슈 > bitly 로 활용

　간단하게 될 줄 알았던 이 토이 프로젝트는 꽤 간단한 문제인 것 같은데 나름의 시간을 할애해야만 했던 '인코딩 이슈'라는 벽이 나타났다. 친절하게 '메시지 인코딩이 잘못되었어~'라고 응답이 내려왔으면 인코딩 문제인 줄 알았겠거늘 그냥 400에러... 한참을 삽질 끝에 url 을 text에 넘기는 과정에서 인코딩 이슈로 api 호출이 잘못되고 있던 것이었다.

　인코딩을 처리해서 넘기려 했지만 이러한 특수문자 같은 인코딩 이슈가 또 나올 수도 있었기에. 그리고 이 링크를 얼마나 클릭했는지도 보고 싶었기에 방법을 찾다 단축 url 서비스로 잘 알려진 bitly를 사용하기로 하였다. 즉, 공모주 관련 알림을 보낼 때 그 공모주의 상세 URL 을 단축 url로 통해서 (인코딩이 필요 없는) url로 만들어서 보내는 형식이다.

　bitly를 가입하고 인증 토큰을 발급받은 다음 아래와 같이 작성을 하면 단축 url 이 생성된 후 반환을 받을 수 있다.
﻿
```java
private String getDetailLink(PublicOffering publicOffering) {
	String jsonBody = "{\"long_url\" : \"" + publicOffering.getDetailUrl() + "\"}";
	JsonNode responseNode = WebClient.create().post()
	    .uri("https://api-ssl.bitly.com/v4/shorten")
	    .header("Authorization", "Bearer f02...71c")
	    .contentType(MediaType.APPLICATION_JSON)
	    .body(BodyInserters.fromValue(jsonBody))
	    .retrieve()
	    .bodyToMono(JsonNode.class)
	    .block();
	return responseNode.get("link").asText();
}
```

　실제로 사용자들이 알림을 받고 클릭을 한 내역을 보면 어떤 공모주에 더 관심이 가는지 혹은 무슨 요일에 더 알림을 보는지 등 서비스를 만들려고 했던 목적 이상의 데이터를 얻을 수 있게 된다.

{{< image src="/images/public-offering-notice-2/bitly.jpg" caption="bitly 대시보드 화면. 실제 사용자들의 액션(?) 데이터" >}}


## 마치며
　﻿결국 (아직 서버에 올리지 않았지만) 최종 결과물은 아래 화면처럼 데이터를 획득하고 (크롤링) 텔레그램의 봇 api와 bitly의 API를 활용해서 메시지를 채널로 전송하는 데까지 성공하였다. 이제 남은 건 해당 애플리케이션을 서버에 업로드하고 이를 주기적으로 호출하는 방법만 찾으면 끝.

　여기까지만 보면 큰 어려움이 없었고 만들기 전에 생각했던 흐름대로 만들었기 때문에 이제 다음 스탭만 진행하면 모든 게 끝날 거라 기대를 했다. 하지만 서버를 선정(?) 하고 그것을 주기적으로 호출하는 방법을 찾는데 예상치 못한 어려움을 만났고 나름 꽤 많은 시간을 삽질해야만 했다.

　To Be Contunue...

﻿