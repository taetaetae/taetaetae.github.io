---
title: 벌써 2년 (feat. 토이프로젝트 회고,가치,수입)
date: 2020-07-12 19:55:02
categories:
  - review
tags:
  - archives-2020
url : /2020/07/12/toy-projects-second-year-review/
featuredImage: /images/toy-projects-second-year-review/ddb.jpg
---

　정확히 2018년 07월 12일 필자의 첫 토이 프로젝트인 ‘기술 블로그 구독 서비스’를 오픈하게 된다. 얼마나 많이 구독(가입) 하겠어 하는 생각이 부끄러울 만큼 6개월이 지나 구독자 수는 1,000명을 넘기고 1년이 지나 2,000명.<!--more --> 어느덧 달력을 보니 오늘이 정확하게 토이 프로젝트를 서비스한지 벌써 2년이 되는 날. 구독자 수는 어느덧 3,000명을 넘어선다. 뭔가 뿌듯하면서도 서비스를 좀 더 디벨롭 하지 못한 필자 자신을 돌아보니 괜히 마음이 무거워지고.

{{< image src="/images/toy-projects-second-year-review/dog.jpg" caption="뭔가 해야하는데... 괜히 눈치만 보이네...<br>출처 : http://egloos.zum.com/nievess/v/657827" width="80%" >}}

　지난 2년 동안을 돌이켜보며 서비스를 어떻게 운영해 왔는지, 그리고 토이 프로젝트가 필자에게 어떤 영향을 주었는지 되돌아보며 셀프 리뷰를 해 보고자 한다.

## 서비스 자체 평가
### 심플한 기능
　말 그대로 토이 프로젝트이기 때문에 기능 또한 아주 간단하다. [awesome-devblog](https://github.com/sarojaba/awesome-devblog)에서 제공하는 개인/단체 블로그들의 포스팅을 조회하여 어제 작성된 글들만 모아 발송한다. 거기에 주간 많이 클릭된 포스팅을 모아서 한 번 더 발송하는 기능까지. 추가적인 기능을 더 디벨롭 해야 하는데 아이디어가 없어서 인지 디벨롭 할 힘이 안 나서 인지 유지만 하고 있는 상태다.

### 서비스에 없어서는 안될 '로깅(Logging)'
　형식을 막론하고 컴퓨터로 돌아가는 모든 '프로그램'은 상황에 따라 미리 만들어 놓은 로직에 따라 움직이는 로봇에 불과하다. 물론 요즘에는 머신러닝이나 AI 같은 기술들로 컴퓨터가 스스로 학습하는 경우도 있지만 그 또한 미리 코딩을 통해 만들어진 부분들. 그렇기 때문에 2년이 지난 지금 이제까지 서비스가 어떻게 돌아갔는지를 확인하기 위해서는 사전에 준비해야 할 것이 있다. 그것은 바로 '로깅'. 서비스 투입 전부터 프론트부터 백엔드까지 다양한 로깅을 해서인지 2년이 지난 지금, 기록된 로그로 다양한 서비스 지표를 확인해 볼 수 있음에 다행이라 생각한다.

### 각종 지표
　먼저 봐야 할 지표는 당연히 가입/해지 추이. 드라마틱 한 선형 그래프는 아니지만 당연히(?) 해지 보다 가입이 더 많고 시간이 지날수록 어느 정도 꾸준하게 가입자가 들어오는 것을 보면 어떻게 알고 가입을 하러 오는지 신기할 따름이다. 하지만 마냥 신기해하지만 말고 해지하는 원인을 분석해야 할 필요가 있어 보인다. 아마도 수집하는 블로그들 중 간혹 개발과 관련되지 않는 글들이 종종 수집되어서 그런 것 같기도 하다.

{{< image src="/images/toy-projects-second-year-review/regist.jpg" caption="가입/해지 트랜드" width="80%" >}}

　다음으로는 클릭수. 눈치가 빠른 분들은 이미 알고 있겠지만 이메일에서 클릭 시 서버에서 각종 로깅을 하고 넘어가게 된다. 그러다 보니 클릭 성향(?)에 대해 집계도 가능한데 아래 지표를 보면 오전 일과를 시작하면서 메일로 종합된 기술 블로그 들을 읽기 시작하고 그중에서 특히 월요일 - 10시가 가장 많은 클릭수가 집계되었다.

{{< image src="/images/toy-projects-second-year-review/click.jpg" caption="클릭수 트랜드 | 시간+요일 별 클릭수 트랜드 | 시간+요일 별 클릭수 히트맵" width="80%" >}}

　이 포스팅을 작성하고 있는 지금까지 약 19,000여 개의 포스팅을 수집하고 발행하였는데 그중에서 가장 인기 있었던 포스팅 TOP 30 은 다음과 같다. 아무래도 단체 블로그의 포스팅을 메일 상단에 위치하고 노란색으로 테두리를 표시해서인지 대부분의 글들이 단체 블로그의 포스팅인 것을 알 수 있다.
- [이 회사, 이 세상 쿨함이 아니다](http://woowabros.github.io/techcourse/2019/12/05/techcourse-openrecruitingday.html)
- [대놓고 자랑하는 글](http://woowabros.github.io/experience/2019/11/12/bravo-baemin.html)
- [LINE 신입 SW 개발자 코딩 테스트, 이렇게 만들어졌습니다](https://engineering.linecorp.com/ko/blog/2020-line-sw-developer-recruit-coding-test/)
- [우테코에서 찾은 나만의 효과적인 공부법](https://woowabros.github.io/techcourse/2020/06/24/techcourse-level2-retrospection.html)
- [LINE 서버 개발자가 되기까지 내가 준비한 것들](https://engineering.linecorp.com/ko/blog/things-i-prepared-to-be-a-line-server-developer/)
- [연봉을 높이는 가장 쉬운 방법은?](http://blog.weirdx.io/post/61620)
- [학교에서 알려주지 않는 17가지 실무 개발 기술 리뷰](http://1ilsang.blog.me/221984558663)
- [간단하게 만드는 이상한 알람](https://woowabros.github.io/experience/2020/05/14/anomaly_alarm.html)
- [팀 문화의 탄생](https://woowabros.github.io/experience/2020/05/13/birth-of-team-culture.html)
- [LINE에서 전 직원이 재택 근무하면서 생산성을 유지하는 방법](https://engineering.linecorp.com/ko/blog/how-liners-keep-productive-while-working-at-home/)
- [Flutter, 왜 선택하지 못했나](https://engineering.linecorp.com/ko/blog/flutter-pros-and-cons/)
- [주석 달 시간에 프로그래밍을 제대로 하기](https://tech.peoplefund.co.kr/2020/02/18/code-rather-than-comment.html)
- [우아한테크코스 : 새로운 시작](https://woowabros.github.io/techcourse/2020/04/10/techcourse-level1.html)
- [기획자는 필요없다.](http://blog.weirdx.io/post/62410)
- [간단하게 구축해 보는 JavaScript 개발 환경](http://d2.naver.com/helloworld/2564557)
- [우아한테크코스 : 나만의 항로 찾기](http://woowabros.github.io/woowabros/2019/09/04/techcourse-level2-retrospection.html)
- [코드리뷰 모음 서비스를 소개합니다.](https://woowabros.github.io/techcourse/2020/06/05/techcourse-javable.html)
- [NAVER Tech Talk: Android 밋업(2018년 11월, 2019년 11월)](https://d2.naver.com/news/4699566)
- [2019년과 이후 JavaScript의 동향 - JavaScript(ECMAScript)](https://d2.naver.com/helloworld/4007447)
- [타다 웹 프론트엔드의 모든 것](http://engineering.vcnc.co.kr/2020/01/introduce-tada-web-frontend/)
- [개발 문화, 사무실 문화](https://tech.peoplefund.co.kr/2020/02/20/development-and-office-culture.html)
- [그런 개발자로 괜찮은가 - `자기개발` 편](https://taetaetae.github.io/2020/06/28/a-good-developer-in-terms-of-self-development/)
- [주니어 개발자의 클린 아키텍처 맛보기](http://woowabros.github.io/tools/2019/10/02/clean-architecture-experience.html)
- [우리도 채팅있으면 좋을 것 같아요.](https://woowabros.github.io/experience/2020/06/19/chat-app.html)
- [코드 가독성에 대해 – 1. 도입과 원칙](https://engineering.linecorp.com/ko/blog/code-readability-vol1/)
- [메인 데이터베이스 IDC 탈출 성공기](http://woowabros.github.io/experience/2019/12/19/ruby_database.html)
- [코드 가독성에 대해 – 3. 상태와 절차](https://engineering.linecorp.com/ko/blog/code-readability-vol3/)
- [그런 개발자로 괜찮은가 - `문화` 편](https://taetaetae.github.io/2020/06/21/a-good-developer-in-terms-of-culture/)
- [[2019. 9. 3] 카페24 개발자 세미나](https://www.imaso.co.kr/archives/5297)
- [진짜 카카오 티스토리 서비스 개판이다](https://jybaek.tistory.com/854)

그렇다면 수익은 얼마나 있었을까? 음?! 이 서비스의 수익이 있다고?? 우선 아래 차트를 보자.

{{< image src="/images/toy-projects-second-year-review/billing.jpg" caption="y축이 거꾸로. 그렇다. 가슴시린 차트. 수익은 당연히 마이너스." width="80%" >}}

　최초 무료 도메인을 받아서 사용했지만 메일 발송 시 스팸으로 발송되는 등 여러 가지 이슈들 때문에 어쩔 수 없이 도메인을 구입하게 된다. (년 단위 결제) 거기에 AWS 프리 티어도 기간이 만료되어 사용하는 만큼 비용이 발생하고. ( Route53(트래픽), EC2(서버 운영), SES(메일 발송) ) 월평균 5만 원 내외를 AWS에 지불하며 서비스를 운영하고 있다. (2년 동안 서버 운영비만 벌써 약 42만원...ㅠㅠ) 후원 버튼을 달아놓았지만 전혀 효과가 없어 아쉬울 따름이다...

## 토이프로젝트가 내게 준것들
　토이 프로젝트를 2년 동안 진행하며 진행하기 전과 후를 비교했을 때 가장 좋았던 점은 실무에서도 찾기 어려운 '책임감'과 '또 다른 지식'을 자연스럽게 얻을 수 있었다는 점이다. (지금은 안정화되어 그렇진 않지만) 매일 오전 9시 50분이 되면 항상 긴장상태로 메일 앱을 키고 기다린다. 내가 만든 서비스에서 메일이 안 오기라도 하면 바로 서버에 들어가 에러 원인을 찾고, 밤새 원인을 찾아 헤매다, 복구하고 다시 또 반복. 지금 생각하면 너무 힘들었던, 포기하고 싶었던 순간들이지만 그 시절이 필자에게는 무엇보다도 소중하고 값진 경험들로 가득 채워질 수 있었음에 감사함을 느낀다.
　실무에서는 java로 웹서비스를 개발하다 보니 다른 언어를 배울 기회가 없었는데 이 토이 프로젝트를 하면서 처음으로 파이썬이라는 것을 알게 되었고, 물리 장비(혹은 VM) 위에서 운영되는 정통적인 웹서비스만 운영하다 이 기회를 통해 AWS를 접해볼 수 있었고, 사용자가 하나둘 늘어남에 따라 제한적인 자원 내에서 효율적인 성능을 내기 위한 고민을 하게 되고. 실무에서 배우지 못한 값진 경험들을 할 수 있어서 너무 좋았다.
※ 참고
- [개발 후기 1부](https://taetaetae.github.io/2018/08/05/daily-dev-blog-1/)
- [개발 후기 2부](https://taetaetae.github.io/2018/08/09/daily-dev-blog-2/)
- [개발 후기 3부](https://taetaetae.github.io/2019/02/17/daily-dev-blog-3/) 

## 마치며
　핑계지만 서비스를 좀 더 디벨롭 해야 하는데 못하고 있다. 회사 바쁜 일만 어느 정도 끝나면 처음부터 다시 설계하여 보다 안정적인 서비스를 만들어 보고 싶다. 나아가 지금은 '돈 잃는' 서비스를 운영 중인데 비즈니스 모델을 고민해서 '돈 버는' 서비스로 바꿀 수 있을지... 가 가장 고민 포인트다. 조언이나 제안은 언제든지 환영이다.