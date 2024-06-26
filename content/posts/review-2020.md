---
title: ‘중니어 개발자’의 2020 회고
date: 2020-12-31T13:31:01+09:00
categories:
  - review
tags:
  - archives-2020
featuredImage: /images/review-2020/main.jpg
images :
  - /images/review-2020/main.jpg
---

﻿　그 어느 때보다도 정신없이 달려온 2020년. 하고 싶은 것도 많았고 큰 꿈을 꾸기도 했지만 현실의 벽 앞에 크게 좌절도 해보기도 하고. 갑작스러운 세상의 변화에 적응하랴 정신적으로 육체적으로 너무 많이 힘들었던 올해. 돌아보면 참 후회가 되지만 한편으론 시련과 좌절 속에서 여러 가지를 배웠던 그런 한 해를 보낸 것 같다.

　﻿필자는 내년이 되면 이제 어느덧 개발자 생활을 한 지 9년 차가 된다. 보통 `주니어`라 함은 단순하게 이제 막 취업한 신입 또는 3\~5년 차를 말하고 `시니어`는 연봉이 X 원을 넘거나 n 연차를 넘을 경우를 말하는 것 같다. 물론 각 회사마다 이 둘을 정의하는 기준이 다르겠지만. 그런데 필자는 주니어도 시니어도 아닌 그 사이에서 애매\~한 연차. `중니어`. 과연 나는 무엇을 해야 할까? 무엇을 해야 연차에 맞는 역할(?)이라고 할 수 있을까? 그리고 그건 누구에게 배워야 하고 누가 가르쳐 주기나 할까?

﻿　매년 회고를 써왔다. 그럼에 연말이 되어서 연례행사처럼 작성하는 게 아닌 나에게 정말 필요한 방향으로 회고를 작성하려 한다. 단순하게 이런저런 일들이 있었고 '어쩔 수 없었네\~' 읊조리는 `무의미한 회고`보다 현실적으로 나 자신을 위해 변화해야 할 게 있으면 굵고 길게 계획을 세워보는 방향으로 해보고 싶다.

- [2019 회고](https://taetaetae.github.io/2019/12/29/review-2019/)
- [2018 회고](https://taetaetae.github.io/2018/12/31/review-2018/)
- 2017년엔 왜 없지..?
- [2016 회고](https://taetaetae.github.io/2016/12/31/adieu-2016/)

## 등장, 코로나-19
{{< image src="/images/review-2020/corona19.jpg" caption="나가지 말라면 나가지 **마!** 밥 먹지 **마!** 모이지 **마!** <br>출처 : [salihgonenli](https://www.instagram.com/p/B55nDnDAHS_/)" width="50%" >}}

﻿　세상이 변했다. 작년까지만 해도 미세먼지가 심하면 마스크를 쓰고 나가곤 했지만 코로나-19라는 전염병이 전 세계에 퍼지며 이제는 마스크 없이 살 수 없는 세상이 되었다. 늘 사무실에 나가 팀 동료분들과 이야기를 하며 밥도 먹고 회의도 하며 업무를 진행했지만 재택근무를 한지 어느덧 반년이 훌쩍 지났다.

﻿　처음엔 집에서 편하게 일을 할 수 있어서 좋았다. 그러나 IT 회사에 근무하고 있지만 아직도 버벅거리고 어색한 화상회의와 더딘 업무 진행으로 인해 점점 시간이 지날수록 답답함은 극을 달했다. 출/퇴근 시간 등 업무이외에 필요한 시간이 사라지며 오히려 업무에 집중하는 시간은 많아졌다. 그에 반해 피로도는 집중한 업무시간에 비례하며 늘어났기에 나무늘보처럼 늘어지는 시간들 또한 많았던 것 같다. 지나고 보면 그러한 시간들을 잘 계획하고 움직였더라면 뭐라도 배우거나 달성했을 시간들인 것 같아서 약간 아쉬움이 남는다. 내년엔 계획하는 시간의 비중을 좀 더 늘리는 것으로.

﻿　아무쪼록 코로나-19 바이러스가 없어지고 다시 예전으로 돌아갔으면 좋겠다. 그에 마스크 잘 쓰고 손 잘 씻고 사람 많이 모이는 곳은 피해야 하는 건 우리가 ~~할 수 있는~~ 해야 할 가장 큰일이겠지.

## 회사생활
### 서비스 전면 개편
　﻿팀에 투입한 이후 가장 큰 규모로 서비스 전면 개편을 진행하였다. 거의 올해 내내 했다고 봐도 무방할 정도. 업무의 양도 많았고 스펙 또한 복잡하였지만 가장 크게 배울 수 있었던 부분은 모놀리틱 서비스에서 마이크로 서비스로의 아키텍처 변화를 시도했다는 점. 그리고 일반적인 Request - Response 식의 1차원적인 흐름에서 `이벤트`라는 행위를 기준으로 모든 프로세스가 영향을 받는 구조를 적용하며 고민했다는 점에서 여러 가지 인사이트를 얻을 수 있었다. 아무래도 `중니어`다 보니 주어진 기능을 개발만 하는 것보단 좀 더 높은 곳의 설계 관점에서 고민하는 연습을 하려고 했던 것 같은데 아직 부족한 것 같다.

﻿　올해도 개발 문화를 개선하려는 노력도 하였다. CI를 재설치하고 다양한 개선을 통해 빌드 속도를 몇 배로 늘리기도 하였고, 단순/반복적인 업무들은 각종 봇들을 개발하여 업무 생산성을 올리기도 하였다. Sentry를 서버 레벨에 적용하여 무분별하게 발생하는 에러들을 그룹핑하여 우선순위에 따라 에러를 해결할 수 있는 구조를 만들기도 하였고, 소나큐브와 jacoco를 적용하여 코드 커버리지를 도식화하며 현재 모듈의 상태를 보여주기도 해보았다. 개발 문화의 개선은 겉으로 드러나진 않지만 잠재적인 문제들을 해결하는 데 가장 큰 효과를 준다고 믿기 때문에.

{{< image src="/images/blog-reorganization-by-hugo/NoThanksButWereBusy.png" caption="우리는 안바쁜 날이 없다.<br>출처 : https://www.astroarch.com/tvp_strategy/no-thanks-busy-pay-back-technical-debt-40188/" width="80%" >}}

　﻿다만 `바쁘다`는 이유로 이런저런 기술 부채를 해결하지 못한 채 한 해를 넘긴 게 가슴에 짐으로 남았다. 최근에 읽었던 [클린 애자일](/posts/review-the-book-clean-agile/) 에서 나오는 내용들로 내년에는 좀 더 안정적인 서비스를 위해 적용해볼 만한 부분들을 제안해 보려 한다. 개발자라면 안`바빴던` 날이 있던가.

### ~~애매한~~ 파트장
　﻿팀이 바빠지면서 몇 명씩 짝을 이루어 `파트`라는 단위로 나뉘어 개발을 진행하였다. 그에 필자는 두 분과 함께 하며 업무를 진행하고 업무현황을 체크하고 진행하는 역할을 맡았다. `파트장`이라기엔 뭔가 애매한 역할. 그래도 무언가를 해보겠다며 호기롭게 매일 스크럼도 진행하고 가끔 각각 면담도 하면서 일이 되게 할 수 있도록 여러 가지 고민과 실행을 해왔다. 조금이라도 경험이 그분들보단 많았기에 상황마다 어떤 식으로 업무를 진행해야 하는지에 대해 약간의 조언(팁)을 드리기도 하였고, 업무가 너무 바빠지자 내가 자처해서 그분들의 업무를 할당받아 진행해보기도 하였다.(지금 생각해 보면 엄청나게 바보 같은 행동..)

　﻿군 시절 그래도 `리더`라는 경험을 해보았기에 그들이 내게 원하는 것. 그리고 내가 해주어야 하는 것을 놓치지 않으려 노력했던 것 같다. 하지만 '나를 따르라'라며 진두지휘했던 그 시절과는 상황이 전혀 달랐고 필자 또한 주어진 업무의 양이 상당히 많았기에 썩 그렇게 만족할만한 파트 리더 역할은 못한 것 같다. (물론 함께 했던 분들 또한 그렇게 느끼실 것 같은...)

### 퇴사
　﻿결과적으로는 퇴사하지는 않았다. 하지만 입사 이래로 "퇴사 버튼" 위에 마우스 커서를 올려놓았기에. 정말 너무 힘들었다. 가끔 징징대며 힘듦을 토로하기는 하지만 이 정도까지는 아니었기에. 과연 `중니어`는 무엇을 해야 할까? 사실 올해 회고의 가장 핵심 키워드라 볼 수 있다. 초당 작성하는 코드의 양이 많으면 될까? 아니면 개발을 하지 않고 서비스 도메인을 잘 이해하며 함께 하는 팀원들이 잘 할 수 있도록 말 그대로 `리딩`만 잘 하면 될까? 그도 아니라면 무엇을 해야 한단 말인가? 그도 아니라면 퇴사를 마음먹은 주된 이유가 과연 코로나-19 때문일까?

{{< image src="/images/review-2020/goodbye.gif" caption="나도 언젠간 이 짤을 쓰는 날이 오겠지...<br>출처 : https://dingdo.tistory.com/146" width="80%" >}}

　﻿필자가 생각하는 `중니어`의 역할은, 우선 주어진 업무를 충실히 하는 건 기본이고 (이건 주니어에 대한 기대수준에도 포함) 팀원들이 바빠서 자칫 챙기지 못한 기술 부채나 시스템 관점에서 개선해야 할 부분을 '개인 시간'을 할애하며 묵묵히 개선해 나가는 것이라 생각한다. 근데 이것도 아니라면?

　﻿야근이라는 의미는 또 무엇일까? 주어진 시간에 주어진 업무를 해결하지 못해서 추가시간을 할애하며 하는 것일까? 그렇다면 야근을 하면 비효율적으로 일하는 것일까? 조금 더 꼼꼼하게 테스트하고 고민하며 안정적인 서비스를 위해 시키지도 않는 본인의 추가시간을 `투자`하는 것은 아닐까? (그렇다고 야근을 좋아하는 건 아니지만, 최근 몇 개월간 필자의 야근 그래프를 보면 거의 최근 비트코인 가격 그래프와 흡사하다.)

　﻿여러 가지 이유들로 퇴사를 결심하였지만 과연 퇴사가 정답일까 싶은 생각들이 지친 필자의 마음을 어르고 달래주어 퇴사를 하지는 않았다. 하지만 어느 조직에서 있던(어느 회사에서 있던) 본인의 가치관과 조직의 목표와 다르다면 깊게 고민을 해볼 필요가 있다. 만약 조직이 오로지 겉으로 드러나는 신규 기능에만 집중하고 그것이 기술 부채를 개선하는 활동보다 높이 평가될 경우 과연 문제가 많은 코드와 시스템을 보고도 눈 가린 채 신규 기능만 더하는 게 과연 맞는 일인 건지. 마침 필자와 비슷한 고민을 하는 누군가가 블라인드에 글을 올렸는데 답글이 너무 와닿아 이곳에 적어본다.

{{< admonition type=quote title="블라인드 내용" open=true >}}
절을 바꾸려 하지 말고, 다른 절로 옮기세요.

당장 옮길 수 없다면 무리 안 가는 선에서만 솔선수범하시고요.

개발에 대한 열정은 화력도 중요하지만 보온도 중요합니다.

타오르고 싶은 마음을 잠깐 보류해뒀다가 포기하지 않고 나중에 다시 꺼내드는 것도 능력이에요.

아니면 회사의 방식에 그대로 녹아드는 것도 방법입니다.

개발자 라기보다는 도메인 지식이 충만한 회사원이 되는 거죠.

안되는 걸 억지로 되게 하려고 해봐야 몸과 마음만 다치거든요.

차라리 되는 걸 더 잘하려고 노력하는 게 더 생산적일지도 모르겠습니다.
{{< /admonition >}}


## 자기계발
### 블로그
　﻿지금 쓰고 있는 회고까지 하면 18개. 딱 작년 보다 1개 덜 쓰게 되었다. 바빠서 못 썼다는 건 핑계니까 접어두고, 돌이켜 보면 '간절하지 못했다'라는 표현이 어울리는 것 같다. 이런저런 핑계들을 벗 삼아 자꾸 나 자신과 타협했던 시간들이 블로그 포스팅 개수만 봐도 뻔히 보인다. 그동안 배운 것들도 경험하며 삽질한 시간들도 많았는데... 아래 접속자 수 그래프를 보면 뭔가 정체기인 것 같아 마음이 아프다. 좀 더 분발하자 태태태.

{{< image src="/images/review-2020/ga.jpg" caption="월간 블로그 접속자 수" width="80%" >}}

　﻿그럼에도 불구하고 [그런 개발자로 괜찮은가](/tags/a-good-developer/) 라는 시리즈를 써왔던 것 같다. 아직도 담고 싶은 주제가 너무 많은데 `회사원`이 아닌 `개발자`로써 가져야 할 마음가짐을 좀 더 정리해보고 싶다. 시리즈를 연재 한다는 게 쉬운 일은 아니지만 개발자로써 매 순간 느끼는 부분들이 많기에 이것들을 머릿속에 남겨두긴 힘들어서라도 정리가 필요한 것 같다.

﻿　회사에서 운영하는 서비스도 개편하는 시국에 나는 뭐하고 있나 싶어 블로그를 개설한지 언 4년여 만에 블로그 [개편](/posts/blog-reorganization-by-hugo/)을 하였다. hexo라는 프레임 워크에서 hugo라는 프레임워크로 변경을 하였는데 성능이나 UI들이 아직까진 만족스럽다. github 블로그의 장점 중에 하나인 자유로운 커스터마이징 덕분에 내 입맛에 맞게 UI를 조정할 수 있다는 것 또한 매우 매력적이다.

### Daily Dev Blog
　﻿파이썬이라는 언어를 배우고 '한번 만들어 보자'하며 만들었던 토이 프로젝트인 [기술 블로그 구독 서비스](http://daily-devblog.com/). 벌써 2년이 지났고 구독자 수는 어느덧 4천 명을 향해 달려가고 있다.

　[gitter.im](https://gitter.im/daily-devblog/community)﻿서비스를 이용해서 준 실시간으로 서비스에 대한 피드백을 받고 대응을 해왔다. 대부분 자신의 블로그가 메일에 포함이 안된다는 내용이 많았는데 블로그 시스템마다 다 상이해서 상황에 따라 코드를 수정하거나 가이드를 드리는 방식으로 답변을 해왔다.

　﻿필자의 서비스를 Clone(?) 한 [서비스](http://today-post.com/)도 나타났다. 이게 엄청난 아이디어나 특허권이 있는 건 아니지만 메일의 내용이나 방식들이 너무 비슷해서 약간 당황했다. 그런데 얼마 전부터 서비스가 안 되는 것 같긴 하다. (무언가 문제가 생겼을까 오히려 궁금하기까지도...)﻿

　﻿이제는 서버의 한계가 찾아와 오전 10시에 메일 발송이 힘들어지는 경우가 가끔 발생하고 있고, 서버 유지 비용 또한 나름의 지출이라 무시 못 할 지경. 그럼에도 메일 상단에 있는 후원 버튼으로 몇몇 분들이 감사한 후원을 해주셨는데 물론 서버 비용을 대체할 정도의 금액은 아니었지만 정말 기분이 좋았다. 뭔가 특단의 조치가 필요한 시점 같다.

{{< image src="/images/review-2020/ddb.jpg" caption="기술블로그 구독서비스 사용자 추이" width="80%" >}}

　﻿다이어트를 하려면 나 다이어트한다고 SNS에 올리면서 동기부여를 받는 사람처럼, 필자도 서비스 개선을 하기 위해 이곳에 몇 개 적어두고자 한다. 안 그러면 계속 미룰 것 같아서.
- 언어/프레임워크 변경 : Python > Java
- 필터링 기능 추가
- 빌드/배포 자동화

### 새벽 5시
　﻿재택으로 인해 오전 9시에 일어나는 필자를 보고 `S`가 문득 [나의 하루는 4시 30분에 시작된다](https://book.naver.com/bookdb/book_detail.nhn?bid=16821937)라는 책을 읽어보라고 권유를 했다. 아무 생각 없이 '그래, 아침에 좀 일찍 일어나 볼까' 하는 마음으로 읽었는데 신세계를 경험하곤 매일 새벽 5시에 일어나기 시작한 지 벌써 한 달이 되어간다. 이 책에서는 무조건 아침 일찍 일어나는 것만을 이야기하진 않는다. 전날 어떻게 잠에 드는가, 아침에 일어나 어떤 행동들을 하고, 왜 아침에 일찍 일어나야 하는가에 대한 내용이 저자의 경험을 토대로 적나라하게 적혀있는 책이다. 꽤 신선한 충격을 받고 매번 눈팅만 하던 페이스북 그룹 [얼또 - Early 또라이 - 일찍 일어나는 또라이가 세상을 바꾼다](https://www.facebook.com/groups/earlyddorai)에도 일어나서 할 일과 해낸 일들을 댓글로 적어가며 하루를 시작하고 있다.﻿

　﻿어느덧 책도 일주일에 한 권은 읽게 되었고, 시간 없다는 핑계가 무색할 정도로 시간적인 여유가 많이 생겼다. 블로그도, 개발 공부도, 운동도. 보다 잘 계획해서 나만의 후회 없는 시간들을 가꾸어 나가야겠다고 다짐해본다.

## 활동
　﻿코로나로 인해 외부 활동을 거의 하지 못했다. 그 와중에 회사에서 필자를 보고 글을 써보지 않겠냐는 권유에 주말 이틀을 꼬박 지새우며 개발자로서 서비스를 개발/운영하는 경험과 느낌을 [회사 공식 블로그](https://blog.naver.com/naver_diary/222110224274)에 기고할 수 있는 좋은 기회도 있었다. 그래도 매번 블로그로 글 쓰는 연습을 해서인지 단 두 번의 퇴고 끝에 글을 작성할 수 있었다. ﻿

　﻿필자의 블로그를 봐서였을까. 어떻게 알고 강의와 집필 제안이 올해 꽤 들어왔다. 욕심 같아서는 "네! 할래요"라고 하고 싶었지만 회사 내규에 안 맞는 부분도 있었고 내가 강의를? 글을? 뭘 안다고? 하는 마음에 몇 차례 요청이 있었지만 아쉽게도 진행하지 못하였다. 나중에 기회가 되면 좀 긴 일정으로 도전해보고 싶은 마음은 있지만 상황이 허락할지는 잘 모르겠다.

{{< image src="/images/review-2020/mento.jpg" caption="다른 멘토분이 필자의 블로그를 언급해 주셔서 매우 놀랐다." width="80%" >}}

　﻿운이 좋아 2021년도 신입사원 멘토가 되었다. 아직 뚜렷하게 무언가를 진행을 하진 않았지만 언택트 시대이다 보니 신입사원분들께 전할 이야기들을 카메라 앞에서 촬영도 해보고 꽤 즐거운 경험이었다. 내년에 진행하게 되면 많은 것을 배울 수 있을 것 같다. 왜 누군가를 가르치려면 배우는 사람보다 가르치는 사람이 더 많이 배운다고 하지 않던가.

## 2021년, 🐮
　﻿회사 친한 형과 함께 코로나가 약간 잠잠해질 즈음 근처에서 술을 마시며 "그런 개발자로 괜찮은가" 시리즈와 비슷한 내용으로 이야기한 적이 있다. 그러다 그 형 입에서 나온 이야기는 아직까지도 많은 울림을 주었고, 매번 곱씹을 때마다 소화 안되는데 까스활명수 먹는 느낌이 든다.

{{< admonition type=quote title="회사 친한 형과의 대화" open=true >}}
﻿이렇게 불평불만 만 토로하는 거 보기 싫다고 작년에 네가 그랬는데 이젠 네가 그러고 있는 거 아냐?

불평불만만 하지 말고 행동을 해 행동을.

개발자에서 회사원 되는 거 한순간이다. 명심해.
﻿{{< /admonition >}}

﻿　내년에는 어떤 변화와 이벤트들이 있을지는 모르지만 우선 나부터 바뀌는 걸 노력해봐야겠다. 그래도 안된다면. 그다음을 준비하는 것으로.