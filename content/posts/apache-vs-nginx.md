---
title: Apache냐 Nginx냐, 그것이 알고싶다.
date: 2018-06-27 17:17:33
categories:
  - tech
tags: 
  - apache
  - nginx
  - web server
  - Event Driven
  - Prefork MPM
  - Worker MPM
  - archives-2018
url : /2018/06/27/apache-vs-nginx/
featuredImage: /images/apache-vs-nginx/want_to_know_that_apache_vs_nginx.png
---
웹서버는 HTTP 프로토콜을 통해 읽힐수 있는 문서를 처리를 하며 일반적으로 웹 어플리케이션의 앞단에 배치되곤 한다. 동적인 리소스는 WAS에게 처리하도록 하고 정적인 리소스를 보다 효율적으로 처리하기 위한 방법일수도 있다. 크게 Apache와 Nginx가 사용되곤 하는데 이 둘의 차이는 무엇일까?<!-- more --> 사실 필자는 사내에서 주로 Apache만 사용하다보니 Nginx는 그저 `Apache와는 다른 방식의 웹서버다` 또는 `보다 경량화 되었다` 정도로만 알고있었는데 이번기회를 통해 제대로 알고 비교를 해보면서 결국 어떤게 좋은지 알아보고자 한다.

> 구글링을 조금만 해보면 Apache와 Nginx를 비교하는 포스팅이 많이 나온다. 이번 포스팅의 목적 이러한 정보들을 단순히 요약/종합 하려는게 아니고, 최대한 실무 서비스를 운영하는 시각으로 정리하고자 함을 밝힌다.

## Apache ( https://httpd.apache.org/ )
우리나라에서 웹어플리케이션을 개발하는 사람들은 한번쯤은 들어봤을 `Apache`. 국내 일반적인 기업에서 웹서버의 표준으로 자리잡았다고 해도 과언이 아닐것 같다. Client에서 요청을 받으면 MPM (Multi Processing Module : 다중처리모듈) 이라는 방식으로 처리를 하는데 대표적으로는 Prefork와 Worker방식이 있다. 간단하게 어떤식으로 처리하는지 알고 넘어가자.
- Prefork MPM
{{< image src="/images/apache-vs-nginx/prefork.gif" src_l="/images/apache-vs-nginx/prefork.gif" caption="Prefork MPM, http://old.zope.org/Members/ike/Apache2/osx/configure_html" >}}

실행중인 프로세스를 복제되어 처리가 된다. 각 프로세스는 한번에 한 연결만 처리하고 요청량이 많아질수록 프로세스는 증가하지만 복제시 메모리영역까지 복제되어 동작하므로 프로세스간 메모리 공유가 없어 안정적이라 볼수 있다.
- Worker MPM
{{< image src="/images/apache-vs-nginx/worker.gif" src_l="/images/apache-vs-nginx/worker.gif" caption="Worker MPM, http://old.zope.org/Members/ike/Apache2/osx/configure_html" >}}

Prefork 동작방식이 1개의 프로세스가 1개의 스레드로 처리가 되었다면 Worker 동작방식은 1개의 프로세스가 각각 여러 쓰레드를 사용하게 된다. 쓰레드간의 메모리를 공유하며 PreFork방식보다 메모리를 덜 사용하는 장점이 있다.

참고로 WAS로 tomcat을 연동하는 경우라면 [mod_jk](https://tomcat.apache.org/download-connectors.cgi), [mod_proxy](https://httpd.apache.org/docs/2.4/mod/mod_proxy.html), [mod_proxy_ajp](https://httpd.apache.org/docs/2.4/mod/mod_proxy_ajp.html) 방식을 Apache 자체적으로 지원해주기 때문에 다양하고 효율적으로 tomcat을 연동할수 있다. [참고링크](https://www.lesstif.com/pages/viewpage.action?pageId=12943367)

## Nginx ( https://nginx.org/en/ )
Nginx에 대해 살펴보기 전에 [구글 트랜드](https://trends.google.co.kr)를 활용하여 Nginx에 대한 관심이 어느정도인지를 보고 넘어가자.

{{< image src="/images/apache-vs-nginx/google-trand.png" src_l="/images/apache-vs-nginx/google-trand.png" caption="최근 5년간 구글트랜드, 파란색이 Apache이고 빨간색이 Nginx" >}}

전세계는 Nginx보다는 Apache에 대한 관심이 많은것으로 보이는데 국내는 아주 조금씩 Nginx에 대한 관심이 오르는것을 볼수있었다. (그래도 아직은 Apache가 월등히 우세한 편이다.)
그럼 Nginx는 어떤식으로 돌아가는 것일까? 가장 유명한(?) 특징이라면 `Event Driven 방식`을 꼽을수 있을것 같다. Event Driven 방식에 대해 잠깐 언급을 하고 넘어가면 요청이 들어오면 어떤 동작을 해야하는지만 알려주고 다른요청을 처리하는 방식이다. ([Producer Consumer Pattern](https://dzone.com/articles/producer-consumer-pattern)과 유사하다.) 그러다보니 프로세스를 fork하거나 쓰레드를 사용하는 아파치와는 달리 CPU와 관계없이 모든 IO들을 전부 Event Listener로 미루기 때문에 흐름이 끊기지 않고 응답이 빠르게 진행이 되어 1개의 프로세스로 더 빠른 작업이 가능하게 될수 있다. 이때문에 메모리적인 측면에서 Nginx가 System Resource를 적게 처리한다는 장점이 있다고 한다.

{{< image src="/images/apache-vs-nginx/nginx_process_model.png" src_l="/images/apache-vs-nginx/nginx_process_model.png" caption="Nginx Process Model (https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale)" >}}

## 그래서 뭐가 좋은가?

이 포스팅을 적으면서 마지막엔 `Apache가 더좋다` 또는 `Nginx가 더좋다`로 마무리를 짓고 싶었는데 어느 시사/교양 프로그램처럼 어쩔수 없는 `열린결말`로 마무리를 지을수밖에 없을것 같다. (어찌보면 이게 정답일수도?)

기술의 선택에 있어서 정답은 없는것 같다.(물론 Spring 을 사용하느냐 서블릿을 직접 구현하는냐 와는 좀 다른 성격의 이야기;;) 운영하고 있는 서비스의 상황을 잘 알고 튜닝을 해가면서 가장 효율적인것을 선택하는게 정답이라고 말할수 밖에... 커뮤니티 파워를 무시 못하기 때문에 Apache를 선택할수도 있을테고, 점점 관심도가 올라간다는건 그만큼의 장점이 있고 또한 메모리 측면에서 동접자 처리시 효율적인 Nginx를 사용할수 있을것 같다. 

정리하면. 내가 사용하기에 어려움이 없는 도구를 잘 활용하는것, 그렇다고 오래된 기술이 편한다고 집착해서는 안되며, 새로운 기술을 두려워 하지말고 경험을 해본 뒤에 결정을 할것 이라고 내릴수 있을것 같다. (어렵다...ㅠㅠ)

---

- 참고 포스팅
http://www.mukgee.com/?p=293
http://knot.tistory.com/88
http://tmondev.blog.me/220737182315
http://tmondev.blog.me/220731906490
http://urin79.com/blog/20654191
http://jaweb.tistory.com/entry/apache-%EC%99%80-Nginx-%EB%AD%90%EA%B0%80-%EC%A2%8B%EC%9D%80%EA%B1%B0%EC%95%BC

