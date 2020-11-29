---
title: 자바, 성능, 모니터링 테크세미나 정리 및 후기 (by 우아한 형제들)
date: 2019-05-12 20:04:01
categories:
  - review
tags: 
  - java
  - performance
  - monitoring
url : /2019/05/12/got-of-java-seminar/
featuredImage: /images/got-of-java-seminar/woo.jpeg

---

실무에서 자바 기반으로 개발을 하고 서비스를 운영을 하다보면 처음엔 아무런 문제가 없다가 사용자가 몰리는 등 이벤트성으로 트래픽이 많아질 경우 꼭 문제가 생기기 마련이다. 그럴때면 뒤늦게 부랴부랴 원인을 찾고 개선하기 바빠지게 된다.  <!-- more --> (아마 윗분들에게 혼나면서?ㅠㅠ) 
평소에 이런 성능문제를 개선하고 미리 모니터링 할수있는 부분에 대해 관심을 갖고 있었던 찰나, 우아한 형제들에서 [5월 우아한 테크 세미나](https://www.facebook.com/woowahanTech/photos/a.1925530564354206/2280664485507477)를 한다기에 부랴부랴 장문의 글로 신청을 하였고 운이 좋아 당첨이 되었다.
한창 회사에서 새로운 서비스 출시, 그리고 잠을 줄여가며 별도로 진행하고 있던 토이프로젝트 등 여러가지로 바쁜 시기였지만 특히 예전부터 뵙고싶던 이상민님께서 직접 강의를 해주신다기에 피곤한 심신을 이끌고 세미나에 참석하였고 그 후기를 적어보고자 한다.

> 두레이로 만드신 발표자료를 공유해 주셨지만 저작권 문제도 있고 해서 필자기준에서 이해한 부분에 대해서만 공유하고자 한다. 더불어 그냥 듣고 앵무새처럼 발표내용 그대로를 공유하는건 의미가 없다고 생각되어...

{{< image src="/images/got-of-java-seminar/1.jpg" caption="포스터만 봐도 벌써부터 가슴이 뛴다(?)." width="80%" >}}

## 성능

구글에서 작성한 [성능이 중요한 이유](https://developers.google.com/web/fundamentals/performance/why-performance-matters/) 라는 아티클을 공유해 주셨다. (시간이 된다면 한번 읽어보길 강추, 무려 한글!) 어플리케이션에서 성능은 사용자의 증가, 이탈율, 응답속도에 영향이 있고 이는 결국 추구하는 가치(이를 테면 수익)에 직면한다고 한다. 
사용자는 어느 관점에서 바라보는가에 따라 달라지고 각 관점에 따라 성능을 챙겨야 하는 부분이 달라진다. 수강신청을 하는 시점에서의 사용자와 뉴스 페이지를 읽는 시점에서의 사용자는 각 성격이 엄연히 다른것처럼. 
- 시스템 관리자
    - 등록된 / 등록되지 않은 사용자
- 서버 관점
    - 로그인된 / 로그인 하지 않은 사용자
- 성능 테스터 관점
    - Active User
        - 서버에 부하를 주는 사용자
        - 메뉴나 링크를 누르고 결과가 나오기를 기다리는 사용자
        - 성능테스트시 Vuser와 거의 동일 ( Vuser : 가상사용자(virtual user) )
    - Concurrent user
        - 서버에 부하를 주고 있거나, 줄 가능성이 매우높은 서비스에 접속중인 사용자
        - 웹 페이지를 띄워놓은 사용자

TPS(Transaction Per Seconds)는 초당 얼마나 많은 요청을 처리할수 있는지에 대한 시스템의 절대적인 수치로 볼수있다. (개발자는 어느상황에서든지 대충 감으로 이야기 하지말고 정확한 수치로 이야기 해야한다는 뼈를 때리는 조언과 함께...)  TPS는 Scale out/up을 통해 증가시킬수 있지만 Response Time 은 불가능하다. 물론 어플리케이션을 튜닝하면 두 수치 모두 개선이 가능하다. 이러한 TPS와 Response Time의 최대치는 출시전에 반드시 테스트를 통해 알고 있어야 이슈발생시 대응하는데 유용하다.
Bottleneck 즉 병목은 장비, 어플리케이션, 저장소, 설정 등 다양한 상황에서 발생할수 있다. 그중에 "아주 일반적"으로 가장 병목이 많이 발생하는 구간은 DB이고 그 다음으로 클라이언트(Web page, App), Network이 있을 수 있다. 
결론은 **Performance engineering is "Composite Art" of IT** 라는 하나의 문장으로 정리를 해주셨다. 아무리 이쁜 디자인과 어렵고 복잡한 기능이 있을지라도 성능이 뒷받침 안된다면 대용량 트래픽 상황에서는 무의미해지기 때문이라고 생각한다.

## 자바

자바의 역사에 대해 설명해 주셨다. ( 역사에 대한 보다 자세한 설명은 [https://www.whatap.io/blog/12/](https://www.whatap.io/blog/12/) 참고 ) 언제부터인가 JDK 라이센스 이슈가 많았었는데 실무에서 개발하는 입장에서는 java 8 에서는 문제가 안되고 java 11부터 라이센스 문제가 복잡하게 생길수 있다고 한다. 이부분은 공식문서(?)를 찾아보는게 좋을듯 하다. (개인 또는 회사에서 사용할 경우 상황에 따라 법적 이슈가 생길수도, 안생길수도 있는 복잡한 문제가 있어보여서... 필자도 제대로 이해하지는 못했다ㅠ)

그리고 각 자바 버전에서 발표한 새로운 기능에 대해 설명해주셨다.

- Java 8
    - lambda, stream, default method, LocalDate / LocalTime 추가
    - stream 과 foreach 의 성능은 거의 차이 없음 (오히려 가독성이 나빠질수도 있다.)
    - ParallelStream 은 해당 장비의 cpu 개수만큼 스레드 풀을 만들어 사용 (오히려 독이 될수 있으니 잘 알아보고 사용할것)
- Java 9
    - Compact Strings : char[] > byte[]
    - G1 default GC : [https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html](https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html)
    - Collections of (불변) : List.of, Set.of, Map.of
- Java 10
    - var 의 등장
    - Application Class-Data Sharing(AppCDS)
- Java 11
    - Oracle JDK의 유료화
    - Http Client. 기본 설정값들을 제대로 알고 써야한다. ( [https://golb.hplar.ch/2019/01/java-11-http-client.html](https://golb.hplar.ch/2019/01/java-11-http-client.html) )
- Java 12
    - Switch expressions
    - Shenandoah : [https://www.youtube.com/watch?v=E1M3hNlhQCg](https://www.youtube.com/watch?v=E1M3hNlhQCg)

## 모니터링

유명한 상용 APM들을 설명해 주셨다. 각각의 장점에 대해 설명해 주셨는데 정말 회사에 요청해 구매할수만 있다면 사서 해보고 싶을정도로 신기한 기능이 많았다. 그중 dynatrace 는 에이전트만 설치해두면 별도의 설정 필요없이 알아서 해준다고...

- dynatrace ([https://www.dynatrace.com/](https://www.dynatrace.com/))
- new relic ([https://newrelic.com/](https://newrelic.com/))
- AppDynamics ([https://www.appdynamics.com/](https://www.appdynamics.com/))
- WhaTap ([https://www.whatap.io/](https://www.whatap.io/))

오픈소스로는 스카우터와 핀포인트를 설명해 주셨다. 필자는 핀포인트로 회사 서비스를 모니터링 중에 있는데 스카우터에도 좋은 기능이 많아 보여 기회가 된다면 개발서버에 설치해서 핀포인트와 각각 장단점을 비교해 보고 싶어질 정도로 스카우터 자랑을 엄청 해주셨다. (NHN에서는 스카우터로 모니터링 하고 있다고 하니 더욱더 관심이 가게 되었다.)

- scouter ([https://github.com/scouter-project/scouter](https://github.com/scouter-project/scouter))
- pinpoint ([https://github.com/naver/pinpoint](https://github.com/naver/pinpoint))

APM 즉, Application Performance Management의 핵심은 바로 [java.lang.instrument](https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/package-summary.html) package 와 [Java ClassFileTransformer](https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/ClassFileTransformer.html) 에 있다고 하셨다. 마치 Spring의  AOP처럼.

이번 세션의 결론은 처음에 이야기 하신 부분과 비슷한 "절대로 단정짓지 마라 ! 데이터로 이야기 하자 !" 라는 문장으로 정리를 해주셨다. 그만큼 테스트를 많이 해보고 평소에 모니터링을 자주 해가며 서비스의 안정성을 높여야 한다는 뜻으로 이해했다.
끝으로 Q&A가 있어 평소에 궁금했던 질문을 드렸고 너무 친절하게 화이트보드에 그래프를 그려주시면서 (원래 강의시간보다 30분정도 더 하게 만든 장본인...ㅠㅠ) 답변을 해주셨다.

Q. Application의 상태를 확인하기 위해 각종 모니터링 툴을 활용하는데, 오히려 모니터링이 과하다 보면 Application 성능에 영향을 주게 된다. 어떻게 해야하는가?

A. 모니터링툴을 홍보하는 쪽에서는 당연히 성능에 영향이 없다고 한다. 하지만 먼저 개발서버에서 테스트를 해봐서 모니터링툴이 있고 없고의 서비 리소스의 차이를 확인해보고 조금씩 적용범위를 늘려가는 식으로 해보는것도 하나의 방법이 될 수 있다. 또한 샘플링을 통해 일부분만 확인하는 방법도 있다. (필자가 이용하는 pinpoint는 request의 20% 이런식으로 샘플링을 하고 있었는데 scouter에서는 response time기준으로 샘플링이 되나보다?ㄷㄷ)

{{< image src="/images/got-of-java-seminar/2.jpg" caption="너무 두서없이 적었나..." width="80%" >}}

그리고 필자의 질문 때문이였는지 실무에서 있었던 장애시 그래프 사례를 보여주시며 끔찍한(?) 상황까지 재밌게 표현해 주시며 약 3시간여 진행된 세미나가 마무리 되었다.

### # 마치며

자바로 Application개발을 하면서 성능과 모니터링은 마치 삼겹살엔 소주, 치킨에 맥주처럼 정말 떼려야 뗄 수 없는 사이인것 같다. 우아한 형제들에서 주최한 이번 기술 세미나는 필자에게 정말 많은것을 배우게 해준 좋은 행사였다. 그리고 회사에 가면 짬나는 시간을 활용해서 스카우터로 성능테스트를 해볼 계획이다. (라고 말하면 안되고 점심에 졸려서 성능테스트를 해봤다고 말해야 직장 상사가 좋아하신다 라고 말씀해주셨다 ㅎㅎ)

{{< image src="/images/got-of-java-seminar/3.jpg" caption="질문을 해야 내것이 된다는 나와의 약속을 이번에도 지킬수 있었다! (질문하고 받은 배달의 민족 쿠폰!)" width="80%" >}}

1,2 회 모두 탈락해서 못들었지만 다음에도 이런 기술관련 행사가 있으면 꼭 듣고 싶고 마지막으로 필자에 질문에 너무 성실하게 답변해 주시고 재밌고 귀에 쏙쏙 들어오는 강연을 해주신 이상민님께 이 포스팅으로나마 다시한번 감사의 말씀을 전하고 싶다.

