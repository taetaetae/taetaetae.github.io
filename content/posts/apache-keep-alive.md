---
title: Apache keepAlive
date: 2017-08-28 19:56:40
categories: [tech]
tags:
  - apache
  - keepAlive
  - archives-2017
url : /2017/08/28/apache-keep-alive/
featuredImage: /images/apache-keep-alive/keepalive_on_off.png
images :
  - /images/apache-keep-alive/keepalive_on_off.png
---

서버를 운영하다보면 간혹 문제가 발생하곤 한다. 이를테면 메모리가 다른이유없이 올라간다거나, 사용자 입장에서 응답속도가 간헐적으로 느린다거나. 그럴때마다 선배개발자분들께서 가장먼저 입에 오르내리는 단어. `keepAlive`. <!--more-->대충 검색을 해보면 접속을 유지하거나 그렇지 않거나 하는 설정이구나 만으로 생각했었는데 제대로 짚고 넘어가는 의미에서 정리를 해볼 필요가 있을것 같다.

## 정의
우선 2.4버전 기준 도큐먼트의 내용을 볼 필요가 있다.
https://httpd.apache.org/docs/2.4/mod/core.html#keepalive
> The Keep-Alive extension to HTTP/1.0 and the persistent connection feature of HTTP/1.1 provide long-lived HTTP sessions which allow multiple requests to be sent over the same TCP connection. In some cases this has been shown to result in an almost 50% speedup in latency times for HTML documents with many images. To enable Keep-Alive connections, set KeepAlive On.

발번역`(파파고의 힘!)`으로 이해한 내용으로는, keepAlive를 사용하면 50%까지 대기시간을 단축할수 있다는 설정이라 나와있다. 하지만 나같은 영어울렁중이 있는 사람들(?)은 영어 문서만을 가지고 완벽히 이해할수는 없다. 그래서 이것저것 찾아보고 다시 정리를 해본다.

## 정리
기본적으로 외부 사용자에 의해 요청(access)이 들어오게 되면 mpm방식이 어떤거든 간에 `(아파치 MPM에 대해서도 정리를 해야겠군..)` 아파치가 처리를 하든 WAS에게 넘겨주든 하나의 흐름이 들어오는데 동일한 사용자에 대해서 이 흐름을 끊고 다시 요청 받을것인가 아니면 연결을 유지하고 바로 처리를 할것인가에 대한 설정값으로 이해하면 좋을듯 싶다. ~~필자가 표현을 잘 못해서 그런것일수도 있으니~~ 그림으로 보는게 가장 빠를지도 모르겠다.

{{< image src="/images/apache-keep-alive/keepalive_on_off.png" src_l="/images/apache-keep-alive/keepalive_on_off.png" caption="출처 : https://www.svennd.be/keepalive-on-or-off-apache-tuning" >}}

위 그림은 IT전공자라면(?) 한번쯤은 봤을 `tcp 3-way handshake` 인데, keepAlive 를 on 하면 초기 연결하는 비용을 조금이나마 줄일수 있다는걸 보여준다. 즉, 다시말하면 keepAlive는 한번 연결된 상대에 대해서 연결을 잃지 않고(이런저런 설정값에 의존) 지속적으로 요청에 응답을 해줄수 있다는 옵션이다. 간단하게 생각하면 이 설정값을 이용하면 모든 요청에 의해서 지속적으로 응답을 해줄수 있으니 무조건 on하면 되는거 아닐까? 하는 생각이 먼저든다. default 값도 on 이니. 허나 자칫 잘못하면 메모리가  모든 접속자 마다 연결 유지를 해 놓아야 하기 때문에 아파치 프로세스수가 기하 급수적으로 늘어나 MaxClient값을 초과하게 된다. 또한 On상태일때 접속유지 하는 프로세스들 때문에 메모리를 그 만큼 많이 사용하게 된다.
이 양날의 검 `keepAlive`는 과연 어떻게 사용해야 하는걸까?

## 사용 시기
우선 앞서 말했던것과 같이 기본값은 `On`이다. 즉, 아파치 설치시 기본으로 연결유지를 하는것. 하지만 상황에 따라 `On`할지 `Off`할지를 결정해야 한다.
- `On` 해야할 경우 
  접속자의 수 상관없이 메모리가 충분할경우
  >메모리가 충분하다 : 접속자가 maxclient 값에 도달했을경우, swap메모리를 사용하지 않는 상태
- `Off` 해야할 경우 
  - 동시 접속자수가 많을경우
  - 메모리가 충분하지 못할경우
- 하지만 위의 경우는 지극히 `일반적인`경우고 운영하는 서버의 메모리의 상태나 접속자의 수를 확인하면서 조정이 필요하다. 꼭 keepAlive에 국한되는것은 아니지만 모든 개발에는 절대적인 답은 없는것 같다.

## keepAlive 를 `On`할 경우의 추가 셋팅부분
(참고 : https://abdussamad.com/archives/169-Apache-optimization:-KeepAlive-On-or-Off.html)
- MaxKeepAliveRequest [회수]
  하나의 지속적인 연결에서 서비스를 제공할 요청의 최대 값을 설정한다. 50 과 75 사이 정도면 충분하다고 한다. 
- KeepAliveTimeout [초]
  연결된 사용자로부터 새로운 요청을 받기까지 서버가 얼마나 기다릴 것인가를 설정한다. (default : 15초) 일반적으로 1~5초 정도로 설정하곤 한다.
- MaxClients [회수]
  자식프로세스들의 최대 값을 설정한다. 이는 메모리의 크기와 상관관계가 있다.
- MaxRequestsPerChild [회수]
  클라이언트들의 요청 개수를 제한. 만약 자식 프로세스가 이 값만큼의 클라이언트 요청을 받았다면 이 자식 프로세스는 자동으로 죽게 된다. 0 일 경우엔 무한대. 이 설정값으로 메모리누수를 방지할수 있다.

어쨋든, 정답은 없다. 상황에 맞춰서 설정할것! 그에따른 책임은 본인이 가져가야하는걸 명심!