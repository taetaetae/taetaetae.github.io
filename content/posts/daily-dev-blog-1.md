---
title: 기술블로그 구독서비스 개발 후기 - 1부
date: 2018-08-05 10:27:21
categories:
  - tech
tags: 
  - aws
  - python
  - flask
  - archives-2018
url : /2018/08/05/daily-dev-blog-1/
featuredImage: /images/daily-dev-blog-1/ddb-thumnail.png
images :
  - /images/daily-dev-blog-1/ddb-thumnail.png
---
이번 포스팅은 약간의 자투리 시간을 활용하여 이것저것 만져보다 만들게 된 `Daily DevBlog`(기술블로그 구독서비스)에 대해 이야기 하려고 한다. <!-- more -->
하나의 글에 관련 내용을 모두 담기에는 양이 많아서 읽는사람도 지루하고, 글을 쓰는 필자 또한 `어불성설` 할것같아 크게 3개의 시리즈로 나눠서 최대한 자세하고 현장감(?)있게 글을 써보려고 노력했다.

- 1부 : [왜 만들게 되었는가 그리고 어떤 구조로 만들었는가](https://taetaetae.github.io/2018/08/05/daily-dev-blog-1/)
- 2부 : [문제발생 및 Trouble Shooting](https://taetaetae.github.io/2018/08/09/daily-dev-blog-2/)
- 3부 : [앞으로의 계획과 방향성](https://taetaetae.github.io/2019/02/17/daily-dev-blog-3/)

글에 들어가기 앞서 최종 결과는 http://daily-devblog.com 에서 확인할수 있다.

---
## 무엇이 나를 움직이게 했는가
얼마전까지 오픈소스는 정말 실력있는 개발자나 유명한 사람들 말고는 금기의 영역(?)이라고 생각했었지만 최근 [오픈소스 개발자 이야기 세미나](https://taetaetae.github.io/2018/07/01/open-source-software-develpoer-story-review/)를 다녀온뒤 마음속에 있었던 벽이 사라지는듯 했다. 세미나를 들으면서 '나도 무언가를 만들어 볼수는 없을까?', '회사라는 명찰을 떼면 난 어느 수준에서 개발을 하고 있는 것일까?' 등 여러 생각들이 머리를 멤돌다 [개발자를 위한 글쓰기](https://www.slideshare.net/zzsza/intro-102870757)라는 글에서 기술블로그들을 모아놓은 [awesome-devblog](https://awesome-devblog.herokuapp.com)를 소개하는 글을 보게 되었고 내 머릿속에 정리안되던 그 생각들은 "이 데이터를 활용해서 무언가를 만들어보자!"로 귀결되었다. 

> 다른 이야기 이지만, awesome-devblog 을 보고 당장 내 블로그도 등록해야지 했었는데 이미 등록이 되어 있었다;; 등록해주신 분께 감사하다는 생각이 들기전에 내 블로그가 누군가에게 보여지고 있구나 하며 새삼 놀라움이 더 컸다.

## 요구사항과 도구 그리고 설계
만들려고 생각해봤던 요구사항은 다음과 같다. 마치 회사에서 개발전 스펙을 정리하듯...
1. 웹페이지를 활용해서 구독하고자 하는 사람들의 이메일을 수집할수 있어야 한다.
2. 매일 전날 작성된 글을 수집하고 조합하여 구독하고자 하는 사람들에게 메일을 보낼수 있어야 한다.

위 두가지만 보면 너무 간단했다. 또한 기존에 사용하지 않았던 기술들을 사용해보면서 `최대한 심플하게` 개발하는것을 첫 개인 프로젝트의 목표로 하고 싶었다. 하여 생각한 아키텍처는 다음과 같다.

{{< image src="/images/daily-dev-blog-1/architecture.png" src_l="/images/daily-dev-blog-1/architecture.png" caption="최대한 심플하게 설계해보자." >}}

데이터는 해당 github에 있길래 그냥 가져다 쓰려고 했으나 그래도 데이터를 관리하시는 분께 허락을 받고 사용하는게 상도덕(?)인것 같아 수소문끝에 연락을 해서 허락받는데 성공하였다.

{{< image src="/images/daily-dev-blog-1/message.png" src_l="/images/daily-dev-blog-1/message.png" caption="데이터 사용을 허락해주신 천사같으신분..." >}}

> 이 자리를 빌어 데이터를 사용할수 있도록 [허락해주신분](https://www.facebook.com/sarojaba) 께 감사인사를 표합니다.

홈페이지를 만들기 위해서는 이제껏 삼겹살에 소주처럼(응?) Java에 Spring을 사용해 왔었지만 이번엔 좀 다른 방식을 사용하고 싶었다. 

{{< image src="/images/daily-dev-blog-1/language_framework.png" src_l="/images/daily-dev-blog-1/language_framework.png" caption="물론 삼겹살에 맥주, 치킨에 소주를 먹어도 되긴 하지만..." >}}

최근에 Flask라는 python기반 웹 프레임워크를 만져본 [경험](https://taetaetae.github.io/2018/06/29/simple-web-server-flask-apache/)이 있어서 이렇다할 고민없이 빠른 결정을 할수 있었다. 또한 DB는 mysql 이나 기타 memory DB를 사용할까 했지만 이또한 심플하게 파일을 활용하는 sqlite3 을 사용하고자 하였다.

## 웹서버\_최종\_수정\_파이널\_진짜\_확정
Flask를 활용하기 위해서는 당연히 웹서버가 필요했다. 처음엔 `awesome-devblog`에서도 사용하고 있던 https://www.heroku.com/ 를 이용해서 해보려 했으나 매일 구독자들에게 메일을 보내는 등 스케쥴러 기능같은건 구현하기 힘들었고 인스턴트 어플리케이션을 등록하는 형태라 사용자의 메일을 입력받고 저장하는 로직을 만들기는 어려워 보였다. (필자가 heroku를 너무 수박 겉핥기식으로 봐서 일수도 있다...) 
좀더 찾아보니 https://www.pythonanywhere.com/ 라는 제한적이지만 무료 서비스가 있었는데 웹콘솔도 지원하고 상당히 매력있어 보여서 `이거다!` 하며 개발을 시작을 했으나 (나름 도메인까지 그럴싸하게 만들었지만... http://dailydevblog.pythonanywhere.com/ ) 세상에 공짜는 없다는 말을 실감하며 앞서 말했던 요구사항을 완벽하게 구현할 수 없는 상황이였다.(request 제한, 스케쥴러 등록 개수 제한 등 보다 여러기능을 사용하기 위해서는 돈을 내고 써야...)
마지막 희망으로 언제샀는지 서랍속 깊이 자고있던 라즈베리 파이를 꺼내서 공유기 DDNS설정을 하고 라즈베리안을 설치하며 웹서버를 위한 셋팅을 시도해보았으나 언제나 그렇듯 (시험공부 하기전에 책상 정리하고 괜히 방청소까지 하다가 피곤해서 자버리는듯한 느낌) 배보다 배꼽이 클것같아 이또한 진행하다가 중단하게 된다.
결국 AWS에서 1년동안은 무료로 사용할수 있는 [Free Tier](https://aws.amazon.com/ko/free/) 라는걸 발견하고 이참에 나도한번 사용해보자라는 마음을 가지고 과금되지 않게 조심조심 셋팅을 할수 있었다. 물론 뒤에서 이야기 하겠지만 약간의 과금은 필요했다ㅠ (나름 심도깊었던 고민을 한방에 해결해버리는 AWS 짱;; 이래서 AWS~ AWS~ 하는가 싶었다.)

{{< image src="/images/daily-dev-blog-1/select_server.png" src_l="/images/daily-dev-blog-1/select_server.png" caption="돌고 돌다 킹갓엠페러제네럴 AWS를 만나게 된다." >}}

## AWS와 DNS, 그리고 Elastic Stack까지

AWS Free Tier 에서는 참 고맙게도 다른사람들이 접속할수있도록 공인 IP를 제공해 줬다. 그래서 아파치에 Flask를 연동까지 하고 최대한 심플하게 웹페이지를 작성하여 접속이 가능하도록 했는데 문제는 IP주소로 서비스를 하기에는 뭔가 2% 부족해보였다. 그래서 무료 도메인을 찾아다니던 도중 https://freedns.afraid.org/ 이라는 서비스를 찾고 결국 http://daily-devblog.mooo.com 라는 도메인으로 AWS에 설정된 공인IP를 연동시킬수 있었다. (지금은 redirect 시켜둔 상태, 2부에서 왜 도메인을 구입했는지에 설명할 예정이다.)
또한 예전 조직장님의 말씀이 떠올라 서비스에서의 또다른 인사이트를 찾기 위해 EC2 서버 한대 더 발급받아 (한대에 전부 올리기에는 메모리가 부족했다;;) Elastic Stack 을 셋팅하여 로깅을 할수 있었고 이를 아래 그림처럼 키바나로 시각화 할수 있었다. (이 얼마나 아름다운가... 다시한번 Elastic Stack 에 무한 감동을;;)

{{< image src="/images/daily-dev-blog-1/kibana.png" src_l="/images/daily-dev-blog-1/kibana.png" caption="각종 로깅을 통해 만들어낸 우아한 대시보드" >}}


하지만 탄탄대로일것만 같았던 첫번째 개인 프로젝트는 여러가지 문제점들이 발생하였고 한 일주일동안 두세시간 자며 트러블 슈팅을 해야만 했었다. (업무시간을 피해가며 하다보니 시간이 나질 않았다...)

--- 

우선 여기까지, 서비스를 만들게 된 계기와 전체 구조에 대해 이야기를 해보았고 다음 2부에서 하게될 이야기는 이보다 훨씬더 복잡하고 피곤한(?) 이야기로 포스팅 될것같다. 이야기의 흐름이 `문제 - 해결`, `문제 - 해결` 식의 java.util.Map 구조라고 해야하나...

2부를 기대해 본다.