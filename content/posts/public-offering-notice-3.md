---
title: 공모주 알리미 개발 후기 - 3부
date: 2021-04-04T18:54:00+09:00
categories:
  - tech
tags: 
  - public-offering-notice
  - heroku
  - gcp
  - archives-2021
featuredImage: /images/public-offering-notice-3/logo.jpg
images :
  - /images/public-offering-notice-3/logo.jpg
---

　﻿[공모주 알리미](https://t.me/PublicOfferingNotice)라는 토이 프로젝트 개발기의 마지막 포스팅이다. 토이 프로젝트를 왜 시작하게 되었고 어떻게 설계하게 되었으며 데이터는 어떤 식으로 수집하고 그 데이터를 어떤 방법으로 사용자들에게 알림을 보내기까지 알아보았다. 이제는 이러한 일련의 '파이프라인'을 자동화해야 할 시간이다. 사람이 직접 수동으로 로컬 컴퓨터에서 위 파이프라인을 실행하는 것이 아니라 별도의 서버에 해당 애플리케이션이 등록되어 있고 이를 어떤 무언가에 의해 트리거링을 해주는 방식으로 말이다.﻿

- 1부 : [프로젝트 설계, 데이터 수집](/posts/public-offering-notice-1)
- 2부 : [수집한 데이터 알림 발송](/posts/public-offering-notice-2)
- 3부 : [서버 선정 및 릴리즈](/posts/public-offering-notice-3)

## 서버 선정
　﻿1부에서 이야기했던 것처럼 [heroku](https://www.heroku.com)라는 PaaS(Platform as a service)를 사용하면 될 것 같았다. 무료 플랜으로도 설계했던 서비스 내용을 모두 소화 가능했기 때문이다. 앞서 만든 Spring Boot Application 을 heroku에 배포를 해보자.

　heroku에서 새로운 'App'을 생성한다. 아래에서 보여주고 있는 화면대로 App name을 지정하고 만들기만 하면 끝. 그러면 배포 방법이 여러 가지가 나오는데 heroku에서 제공하는 CLI를 사용하는 방법, 그리고 Github 과 연동하거나, 컨테이너 레지스트리를 활용하는 방법 총 3가지가 있다. 여기서 필자는 Github을 활용해서 연동하는 방법을 소개해 보고자 한다.

{{< image src="/images/public-offering-notice-3/1.jpg" caption="heroku 에서 app을 생성하자." >}}

　﻿로컬에서 만든 애플리케이션을 Github에 push 하면 Github Repository 가 생기고 작업 파일들이 정상적으로 업로드된 것을 확인할 수 있다. 그다음 heroku에서 만들었던 App 페이지에서 Deploy 탭을 클릭하면 아래와 같이 3가지 방법으로 Deploy를 할 수 있다고 나오고, 이 중에 "Connect to Github"을 선택하면 Github 과 연동할 수 있는 버튼이 생기고 이를 누르면 자동으로 본인의 Github 내 Repository를 등록할 수 있도록 화면이 바뀐다.

{{< image src="/images/public-offering-notice-3/2.jpg" caption="heroku 와 github 연동" >}}

　﻿그다음 위에서 Github에 push 했던 Repository 이름을 적고 검색하면 조회가 되고 'Connect'를 누르면 자동으로 연결이 된 것을 확인할 수 있다. 연동된 Repository 브랜치에 코드가 푸시 되면 자동으로 heroku에 배포가 되도록 자동화 설정도 가능하고 그 아래에 보면 브랜치를 선택해서 배포를 수동으로 할 수 있기도 하다. 수동으로 푸시를 눌러보면 이런저런 빌드 로그가 나오고 최종적으로 배포가 되어 `{Appname}.herokuapp.com` 을 접속해보면 서버에 배포가 되어있는 것을 확인할 수 있다.
- sample Github Code : [taetaetae@heroku#HelloWorldController.java#L11](https://github.com/taetaetae/heroku/blob/master/src/main/java/com/taetaetae/helloworld/heroku/HelloWorldController.java#L11)
- sample heroku app : https://taetaetae-test.herokuapp.com/

{{< image src="/images/public-offering-notice-3/3.jpg" caption="수동으로 배포를 해보자." >}}

　﻿heroku에서는 이렇게 몇 번의 클릭만으로 간단하게 애플리케이션을 배포할 수 있는 기능을 제공하고 있고 서버 내 로그도 아래 화면처럼 보여주고 있기 때문에 쉽고 간단하게 서버를 구성하고 싶은 사용자들에게 매력적으로 보이는 것 같다. 단, 무료 플랜의 제한사항들을 자세히 살펴보고 사용할 것을 추천한다.

{{< image src="/images/public-offering-notice-3/4.jpg" caption="서버 로그도 볼 수 있다!" >}}

## 호출 테스트, 문제의 시작

　﻿앞서 만들었던 텔레그램 채널에는 아무도 가입을 하지 않았기에 배포한 heroku web endpoint를 호출하면 텔레그램 봇을 통해 알림이 오는 걸 테스트하고 싶었다. 그런데 아무리 호출을 해도 서버는 타임아웃이라는 에러 응답을 뱉기 일쑤였고, 로직이 문제인지 한참을 리팩토링하며 원인을 파악하는데 꽤 오랜 시간을 삽질하였다. 왜 타임아웃이 발생할까? heroku는 web에서 바로 실행할 수 있는 console 페이지를 제공하고 있었다. 그래서 '크롤링을 하기 위한 페이지'와 '구글'을 비교하기 위해 단순하게 curl 해서 가져오는 테스트를 해보니 아래처럼 확연히 결과가 달랐다. 결국은 heroku 와 크롤링 하는 서버 간의 네트워크 타임아웃이 문제였던 것.

{{< image src="/images/public-offering-notice-3/5.jpg" caption="오류가 나고 원인을 찾는 과정이 가장 어려운 것 같다. 어플리케이션의 문제가 아닌 네트워크 자체의 문제" >}}

## 그래서 어떻게 해결 했나?
　﻿heroku에서 타임아웃이 발생하는 문제를 해결하려 여러 구글링을 통해 찾아봤지만 방법을 찾을 수 없어서 결국 heroku를 사용하는 것을 포기하고 다른 방법을 찾아봐야만 했다. 앞서 이야기했지만 토이 프로젝트를 무료로 운영하고 싶었기 때문에 이런저런 방법을 찾다 보니 AWS 와 비슷하게 Google에서도 GCP라는 서비스를 알게 되었다. 이곳에서도 1년 동안은 무료이지만 이후 AWS처럼 과금이 드는 문제가 아쉬워서 좀 더 찾아보니 아주 작은 서버에 특정 조건을 맞춘다면 평생 무료로 사용이 가능하다는 걸 알게 되었고 이를 사용하기로 마음먹는다. (참고 사이트 : https://nhj12311.tistory.com/317)﻿

{{< image src="/images/public-offering-notice-3/6.jpg" caption="GCP에 가입하고 인스턴스를 생성하면 web에서도 접근 가능한 SSH 환경이 제공된다." >}}

　﻿GCP에 가입을 하고 무료 서버(?)를 발급받아서 각종 세팅 (java, crontab, timezone 등) 을 한 다음 실행을 시켜보니 아무런 문제 없이 잘 돌아갔고, GCP는 heroku 와는 다르게 가상 서버에 직접 접속이 가능했기에 web endpoint로의 호출이 아닌 crontab을 활용해서 로컬에서 직접 호출하도록 하였다. 앞서 이슈가 되었던 heroku처럼의 타임아웃 문제는 전혀 없었고 텔레그램으로 봇 알림도 잘 오는 것을 확인할 수 있었다.

## 마치며
{{< image src="/images/public-offering-notice-3/gadget.jpg" caption="나와라 만능 팔!<br>출처 : https://m.blog.naver.com/maifunnyday/220155296568" width="50%" >}}
　﻿머릿속으로 생각만 하는 것과 실제로 경험해보면서 얻은 돈 주고도 살수 없는 소중한 경험들. 거기에 heroku 나 bitly, gcp 등 들어만 봤지 해보진 않았던 경험들도 얻을 수 있어서 나만의 무기가 하나 둘 생겨난 기분이 들어 매우 좋았다. (heroku + telegram로 더 멋진 것을 만들어 볼 계획이다.) 더불어 지금 만든 '공모주 알리미'를 확장시켜서 단순히 한번 만들고 끝나는 게 아니라 다양한 기능들도 추가해서 토이 프로젝트 본연의 목적, 그리고 이런 걸 만들 수 있다는 자신감(?)을 점점 쌓아 올릴 생각이다.

　﻿끝으로, 공모주에 관심을 갖는 독자가 있다면 그동안 필자가 만들었던 [공모주 알리미](https://t.me/PublicOfferingNotice) 가입을 해보는 것도 좋을 것 같다.﻿