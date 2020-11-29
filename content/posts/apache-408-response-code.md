---
title: 아파치 엑세스 로그에 408코드가?
date: 2018-04-29 17:39:36
categories:
  - tech
tags: 
  - apache
  - 408
  - archives-2018
url : /2018/04/29/apache-408-response-code/
featuredImage: /images/apache-408-response-code/network_flow.png
---
예전에 아파치 로그를 엘라스틱 스택을 활용하여 [내 서버에 누가 들어오는지를 확인할수 있도록 구성](https://taetaetae.github.io/2018/04/10/apache-access-log-user-agent)을 해두고 몇일간 지켜보니 다음과 같은 엑세스 로그가 발생하고 있었다.<!-- more -->
```markdown
1.2.3.4 - - [26/Apr/2018:01:27:33 +0900] "GET /aaa/ HTTP/1.1" 200 6001 30788 "http://www.naver.com" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
1.2.3.4 - - [26/Apr/2018:01:28:08 +0900] "-" 408 - 30 "-" "-"
1.2.3.4 - - [26/Apr/2018:01:28:08 +0900] "-" 408 - 28 "-" "-"
1.2.3.4 - - [26/Apr/2018:01:28:08 +0900] "-" 408 - 12 "-" "-"
1.2.3.4 - - [26/Apr/2018:01:28:08 +0900] "-" 408 - 30 "-" "-"
1.2.3.4 - - [26/Apr/2018:01:28:50 +0900] "GET /aaa/ HTTP/1.1" 200 5999 13521 "http://www.naver.com/" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
1.2.3.4 - - [26/Apr/2018:01:29:14 +0900] "GET /aaa/ HTTP/1.1" 200 5996 19437 "http://www.naver.com" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
1.2.3.4 - - [26/Apr/2018:01:29:15 +0900] "GET /aaa/ HTTP/1.1" 200 5997 17553 "http://www.naver.com" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
1.2.3.4 - - [26/Apr/2018:01:29:15 +0900] "GET /aaa/ HTTP/1.1" 200 5998 17429 "http://www.naver.com/" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
1.2.3.4 - - [26/Apr/2018:01:29:53 +0900] "-" 408 - 30 "-" "-"
1.2.3.4 - - [26/Apr/2018:01:29:53 +0900] "-" 408 - 30 "-" "-"
1.2.3.4 - - [26/Apr/2018:01:29:53 +0900] "-" 408 - 32 "-" "-"
1.2.3.4 - - [26/Apr/2018:01:29:53 +0900] "-" 408 - 38 "-" "-"
1.2.3.4 - - [26/Apr/2018:01:29:53 +0900] "-" 408 - 29 "-" "-"
1.2.3.4 - - [26/Apr/2018:01:30:54 +0900] "GET /aaa/ HTTP/1.1" 200 6000 17881 "http://www.naver.com" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
1.2.3.4 - - [26/Apr/2018:01:31:34 +0900] "-" 408 - 30 "-" "-"
1.2.3.4 - - [26/Apr/2018:01:31:34 +0900] "-" 408 - 30 "-" "-"
1.2.3.4 - - [26/Apr/2018:01:31:34 +0900] "-" 408 - 25 "-" "-"
```
한시간에 만건이상 `응답코드는 408`, `referrer도 없고`, `useragent도 없는`, `ip들도 매우 다양한` 이상한 녀석들이 요청되고 있었다. 

> 이렇게 엑세스 로그를 분석할수 있는 구성을 해두고 나니 보였지 안그랬음 그냥 지나갔을 터..

이러한 데이터를 키바나에서 보면 아래처럼 볼수있는데 한눈에 봐도 과연 의미있는 요청들일까? 하는 의구심이 들정도이다. (1시간 아파치 엑세스 로그)

{{< image src="/images/apache-408-response-code/200vs400.png" src_l="/images/apache-408-response-code/200vs400.png" caption="주황색이 408응답" >}}

그럼 이런 호출들은 도대체 뭘까? 천천히 생각좀 해보자.
  1. 정상적이지 않는 호출로 우리 서버의 취약점을 파악하려 하는것들일까?
  2. 응답코드 408은 요청시간초과 응답코드인데... 오히려 클라이언트 입장에서 문제가 있는건 아닐까?
  3. 어플리케이션 로직이 잘못되어 무한루프에 빠졌나;

[위키백과](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)에서는 아파치 응답코드 중 `408`에 대한 응답을 다음과 같이 알려주고 있다.

> The server timed out waiting for the request. According to HTTP specifications: "The client did not produce a request within the time that the server was prepared to wait. The client MAY repeat the request without modifications at any later time

즉, 아파치 단에서 타임아웃을 내버리는 상황. 여러 다양한 키워드들로 구글링을 해봐도 이렇다할 검색결과를 찾지 못하고 네트워크 관련상황인지 싶어 크롬 개발자도구를 열어 네트워크 지연 테스트를 해보았으나 별 효과가 없었다. 그렇게 범인찾는 형사의 심정으로 이것저것 알아보다 우연히 집에서 원격으로 회사 VPN 붙어서 테스트 하던도중 관련 증상을 재현 할수 있게 되었다.


### # 재현상황
우선 아파치버전은 2.2이고 `KeepAlive Off`가 되어있는 상황. 아래그림처럼 집PC - 공유기 - VPN - Apache - tomcat jenkins 상황이였는데 젠킨스에 한번 접속후에는 항상 408 응답이 주루룩(?) 발생하는것을 알수 있었다. (사실 맨위에 엑세스 로그가 재현한 엑세스 로그이다.)

{{< image src="/images/apache-408-response-code/network_flow.png" src_l="/images/apache-408-response-code/network_flow.png" caption="요청의 흐름" >}}


### # 결론
정확한 근거를 추론할수는 없었지만 재현한 바에 의해 결론을 내리자면, 특정 네트워크 환경에서 408응답이 발생하는것을 알수 있었다. (그중에 하나가 공유기 환경이라는 점)
해당 요청을 막기에는 너무 다양한 ip들이고 서비스 특성상 `keepAlive off`로 설정한점을 미루어 볼때 해당 요청을 deny 시키기에는 다소 무리가 있다고 보여진다. 또한 시스템에 영향을 줄 정도가 아니므로 무시하는것으로 결론을 내렸다.

결과적으로는 `그냥무시`로 끝났지만 그래도 알고 무시해야지~ 하는 마음으로 재현까지 해볼수 있었던 좋은 경험이였다.