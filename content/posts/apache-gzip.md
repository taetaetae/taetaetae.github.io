---
title: gzip 설정으로 속도를 더 빠르게!
date: 2018-04-01 17:58:11
categories:
  - tech
tags: 
  - gzip
  - mod_deflate
  - archives-2018
url : /2018/04/01/apache-gzip/
featuredImage: /images/apache-gzip/apache_gzip.png
images :
  - /images/apache-gzip/apache_gzip.png
  
---
내가 운영중인 웹서비스의 응답속도를 보다 더 빠르게 하기 위해서는 어떤 방법이 있을까? 
웹 서비스를 위해 서버를 구성할 경우 일반적으로 앞단에 `웹서버`를 두고 그뒤에 `WAS`를 두는 설계를 하곤 한다. <!-- more -->여기서 웹서버는 대표적으로 Apache나 Nginx가 있고 WAS는 tomcat이나 기타 다른 모듈을 사용하는데 이렇게 두단계로 나누는 이유는 여러가지가 있겠지만 여기서는 앞단의 웹서버(Apache)의 설정으로 응답속도를 줄일수 있는 방법을 알아 보고자 한다.

## 웹페이지의 응답속도를 줄일수 있는 '일반적인'방법들
꼭 서버의 설정들을 건드리지 않고도 웹페이지의 응답속도를 줄일수 있는 방법은 다양하다. 가장 간단하게 코드 레벨에서 설정할수 있는 방법으로는 스타일시트를 위에 선언하거나 java script는 코드 아래부분에 넣는것만으로도 어느정도 응답속도를 줄일수 있다고 한다. 
> (사족) 신입시절 회사 대표님이 필수로 읽어보라고 전 직원들에게 선물해주셨던 [웹사이트 최적화기법 (스티브 사우더스 저)](http://book.naver.com/bookdb/book_detail.nhn?bid=4587095)이 생각이 난다. ~~모두 사주려면 돈이 얼마야...~~ 그만큼 웹개발자들에게 중요하면서도 한편으로는 기본이 되는 부분들이니 한번쯤 목차라도 읽어보는게 좋을듯 하다.

사실 이 포스팅을 작성하게된 가장 큰 계기는 얼마전 사내 해커톤을 하면서 경험한 부분 때문이다. ( + 들어만 봤지 실제로 해보지는 않아서... ) 서버에서 node(React)를 띄우고 그 앞단에 Apache로 단순 Port Redirect ( 80 → 3000 ) 시켜주고 있었는데 react 에서 사용하는 `bundle.js`의 용량이 크다보니 최초 페이지 접근시 로딩시간이 5초 이상되어버린 것이다. bundle.js를 줄여보는등 다양한 방법을 사용했다가 결국 Apache 설정을 통해 1초 이내로 줄일수 있었다.

## gzip 
우선 `gzip`이란 파일 압축에 쓰이는 응용 소프트웨어로 GNU zip의 준말이라고 한다. (참고 : [위키백과 Gzip](https://ko.wikipedia.org/wiki/Gzip)) 이를 사용하기 위해서는 브라우저가 지원을 해야하는데 https://caniuse.com/#search=gzip 을 보면 대부분의 브라우저에서 지원하는것을 볼수 있다. 

## 데이터 흐름
그럼 gzip 을 사용했을때와 사용하지 않았을때의 차이는 어떻게 다를까? 우선 Request/Response Flow 를 잠깐 살펴보면 다음과 같다.
- gzip 사용 전

{{< image src="/images/apache-gzip/before_gzip.png" src_l="/images/apache-gzip/before_gzip.png" caption="출처 : betterexplained.com" >}}

1. 브라우저가 서버측에 `/index.html`을 요청한다.
2. 서버는 Request를 해석한다.
3. Response에 요청한 내용을 담아 보낸다.
4. Response를 기다렸다가 브라우저에 보여준다. (100kb)

- gzip 사용 후

{{< image src="/images/apache-gzip/after_gzip.png" src_l="/images/apache-gzip/after_gzip.png" caption="출처 : betterexplained.com" >}}

1. 브라우저가 서버측에 `/index.html`을 요청한다.
2. 서버는 Request를 해석한다.
3. Response에 요청한 내용을 담아 보낸다. 여기서 해당 내용을 압축하는 과정이 추가가 된다.
4. Response header에 압축이 되어있다는 정보를 확인후 브라우저는 해당 내용을 받고(10kb), 압축을 해제한 후 사용자에게 보여준다.

정리하면, gzip을 사용하면 서버는 Client에게 보낼 Response를 압축하기 때문에 네트워크 비용을 줄일수 있어 응답속도가 빠른 장점이 있다. 

## 무조건 사용해야 하는가?
물론 무조건 좋은 `(마치 show me the money 같은)`정답은 없다. 서버에서 압축을 하여 Client에게 보내면 그대로 사용자에게 보여주는것이 아니라 `압축을 해제하는 과정`이 추가적으로 필요하다. 이러한 과정에서 브라우저는 cpu를 사용하게 되어 오히려 랜더링 하는 과정이 느려질수 있어 자칫 응답속도는 빨라졌다 하더라도 사용자 체감상 더 느려진것처럼 보여질수 있다. 따라서 상황에 맞춰 gzip을 사용해야 할것인지 말것인지에 대해 테스트가 필요하다.

## 마치며
학부시절 또는 신입시절, 아파치는 정적인 리소스를  담당하고 톰켓은 서블릿 처럼 데이터 가공이 필요한 페이지를 담당한다고 주문을 외우듯 하였지만, 웹서버에서 압축을 하면 어떤 효과가 있는지 실제로 경험해보는게 가장 중요한것 같다.
적용 방법은 복붙하는 느낌이라 아파치 공식 문서를 링크 하는것으로 해당 포스팅을 마무리 하겠다.
- Apache Document (사용법) : https://httpd.apache.org/docs/2.2/ko/mod/mod_deflate.html
- 적용 테스트 사이트
  - https://developers.google.com/speed/pagespeed/insights
  - http://www.whatsmyip.org/http-compression-test