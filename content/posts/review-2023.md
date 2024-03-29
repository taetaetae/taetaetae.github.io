---
title: "초보 시니어 개발자의 2023 리뷰"
date: 2023-12-31T15:26:10+09:00
categories:
  - review
tags:
  - archives-2023
featuredImage: /images/review-2023/mentor.jpg
images :
  - /images/review-2023/mentor.jpg
---


　언제부터인가 새해가 되면 그 해의 키워드를 선정하고 해시태그처럼 달고 다니며 한 해를 보내온 것 같다. 작년의 키워드는 "한계". 한정된 시간 속에서 하고 싶은 것도 많고 해야 할 것도 많은 나로서는 중요도에 따라 어쩔 수 없이 무언가를 포기하게 되는 순간들이 찾아왔다. 그럴 때마다 늘 어쩔 수 없다는 자기 가면을 쓴 채 정말 하고 싶던, 꼭 해야 할 것들임에도 불구하고 다음에 해야지 하고 넘어갔던 적이 많았다. 

　그렇게 시간을 보내니 아쉬움이 남게 되었고 잠을 줄여서라도 할 것 들을 하자며 나를 극한으로 몰아붙여보자는 의미로 작년의 키워드를 "한계"라고 정했고 정말 많은 것들을 경험할 수 있었다. 그렇게 나를 몰아붙이는 삶을 살다 보니 말 그대로 그저 "여러 가지만 했던" 한 해로 기억된다. (아마 그래서 작년 리뷰가 없던 이유일지도...?)

　작년의 "한계"라는 키워드를 통해 잠은 죽어서 자야지 하는 마음으로 불타는 열정을 연습했다면 올해는 같은 시간을 쓰더라도 제대로 쓰고 싶은 마음에 많은 것들을 배우자는 의미로 "배움"이라는 키워드를 선정하게 되었다. 개발자로 살아온 지 올해로 11년 차가 되는 해 이기도 하고 이제는 "시니어 개발자"라는 수식어가 붙다 보니 더욱 시간을 허투루 보낼 수가 없다는 생각이 들었다. 그렇게 "배움"이라는 키워드를 가지고 한 해를 지나와보니 정말 많은 것들을 경험 그 이상으로 배울 수 있었고 그에 대한 한 해의 리뷰를 해보고자 한다. 

## 회사원으로써의 노력
　부여받은 일은 기본이고 그 이상을 스스로 찾아서 해야 하며, 구성원 모두가 함께 성장할 수 있는 분위기를 이끌어 나가야 하는 일당백 '시니어 개발자'로써 회사 생활을 해왔던 것 같다. (쓰고 보니 이력서에서나 볼법한 문장이지만;;) 특히 후배 개발자분들이 잘 성장할 수 있는 환경을 조성하고 그러한 과정들이 결국 서비스가 나아가고자 하는 방향에 보탬이 될 수 있도록 서포트 하는 것에 집중을 해왔다.

　가끔은 팀 내에 쌈닭(?)이 되어 돌아만 가게 하던 일을 개발자로써 확장성과 유지 보수성을 위해 개선해 보자는 자세를 취해 보기도 했고 함께 일하는 주니어 분들께 하기 싫었지만 (그 시절 나를 보는 것만 같았던) 좀 더 올바른 개발자로서의 성장을 하는 바람으로 쓴소리를 몇 번 건넨 것 같다. 지나고 보면 좋은 게 좋다는 식으로 넘어가도 될법했나 싶지만 우리는 그저 코딩만 하는 기계가 아니기에. 누군가는 이런 생각과 말을 해야 하지 않을까 하는 이상한 책임감의 모자를 써보기도 했었다.

{{< image src="/images/review-2023/jenkins-idc.jpg" caption="IDC장애 대비 Jenkins 이중화 구성<br>Active IDC 장애시 Standby IDC 에서 Jenkins 운용이 가능하다. " width="80%" >}}

{{< image src="/images/review-2023/engineeringday.png" caption="사내 기술공유 행사 발표" width="80%" >}}

　기술적 기억으로는 젠킨스 IDC 이중화를 위해(master-slave가 아닌) 스스로 꽤 장기간에 걸친 시행착오를 통해 젠킨스 클러스터를 IDC간 이중화 구성하기도 하였고 인원 대비 업무량이 많다 보니 늘어만 가는 기술 부채를 개선하고자 자체적으로 '기술/프로세스 개선 TF'를 구성해서 개발팀에서 챙겨야 할 부분들을 놓치지 않기 위한 장치들을 만들었다. 여러 output 중에 하나로 팀 내 주니어 분과 함께 하반기 사내 기술 공유 행사에서 "그런 배포 프로세스로 괜찮은가(feat. Github Action)"라는 제목으로 배포 자동화 사례를 사내 오픈소스화(Github Action Marketplace) 하여 발표하기도 하였다.

{{< image src="/images/review-2023/reading-club.png" caption="나름 열정스러운 모임 이름" width="30%" >}}

　사내 독서모임에서 모임장을 자처하여 인문학 독서 소모임을 만들고 10여명 정도의 사람들과 함께 진행을 해보기도 하였고, 같은 서비스를 만들고 있는 다양한 사람들끼리 한 달에 한 번씩 모여 식사 자리를 가졌던 미식회 모임을 운영해 보기도 하였다. 개발과는 관련이 없을 수도 있지만 여러 모임을 운영해 보면서 "관계" 그리고 "조직 운영"에 대한 부분을 간접적으로나마 느껴볼 수 있었던 것 좋은 경험으로 기억될 것 같다.

## '태태태'라는 부케 만들기
{{< image src="/images/review-2023/deview-campus.jpeg" caption="학생 개발자 분들과 함께" width="100%" >}}

　오래전부터 회사 밖에서의 "나"를 찾기 위한 노력들을 해오고 있다. 개발자로써 회사라는 컴포트 존을 벗어나도 스스로 성장할 수 있는 단단한 내가 되기 위한 노력들. 올해 초 회사 주최로 진행된 DEVIEW 2023의 곁다리(?) 행사인 [DEVIEW CAMPUS 2023](https://d2.naver.com/news/8012036)에서 대학생분들에게 개발자의 성장에 대한 내용으로 발표를 하기도 하였고 다양한 경로를 통해 개발자를 꿈꾸는 분들과 함께 멘토링을 진행하기도 하였다. 멘토링을 할 때마다 느끼지만 내가 취업을 했을 때와의 개발에 대한 온도는 사뭇 달라졌다. 전공이나 나이를 불문하고 신입 개발자분들의 수준이 어마 무시하게 올라왔고 가끔은 나도 모르는 부분을 질문받을 때가 있을 정도로 지식의 수준이 꽤 많이 높아졌다는 걸 몸소 느끼는 중이다.

　이렇게 멘토링을 자처해서 하는 이유는 여러 가지가 있지만 멘토링을 할 때면 멘티보다 멘토가 더 도움을 받는 게 크기 때문이다. 어떤 조언을 해야 멘티분이 도움을 얻을지에 대한 생각, 질문을 받았을 때 그저 답을 알려주기 보다 스스로 성장할 수 있도록 힌트를 잘 주는 대화를 하는 연습. 무엇보다 채용시장이 얼어붙은 요즘 기술 수준이 꽤 많이 높은 신입 개발자분들과 함게 고민하며 멘토링을 하는 그 자체가 고인물 개발자(?)가 되지 않기 위한 나 스스로의 장치로 활용 중이다.

　좋아 보이는 콘텐츠에 나의 의견을 추가하여 큐레이팅 하는 서비스인 [커리어리](https://careerly.co.kr/@taetaetae)라는 곳에 올해는 약 90여 개의 글을 작성했고 신기하게도 이 글을 작성하는 시점으로 어느덧 1만 3천여 명이 팔로우를 해주고 계신다. 글 쓰는 것에 진심이다 보니 글을 쓰다 보면 가끔 주간 조회 수 상위에 랭크되는 경우도 있는데 그럴 때마다 하나의 글을 쓰더라도 모두에게 도움이 될 수 있을 글을 적어야겠다는 책임감이 들기도 한다. 그냥 한 번 해볼까 하며 시작했던 글쓰기가 인생의 변곡점을 줄 정도로 강력한 효과를 주고 있다. 글쓰기는 앞으로도 쭉 킵고잉할 예정이다.

## 환경 바꾸기
　내 기술 이력서의 제목인 "Software Engineer Crazy for Growth"에 걸맞게 나는 성장하는 것에 미친 것이 틀림없다. 누군가는 성장 라이팅 하지 말라고 하지만 아무리 생각해도 개발자의 삶에 성장을 뺀다면 그저 API 찍어내는 기계가 아닐까 조심스레 생각도 해본다. 심지어 요즘엔 ai가 너무 발달되어 있다 보니 그들(?) 과의 경쟁에서 살아남으려면 개발자는 항상 성장을 생각해야 한다. 그러다 보니 정체되어 있다는 느낌을 받을 때면 환경부터 바꿔보려고 노력하는 편이다. 바뀐 환경이 결국 나를 바꿔줄 거라 믿기 때문에.

{{< image src="/images/review-2023/json-study.png" caption="JSON 상하차 스터디 멤버들과<br>세번째 스터디는 양꼬치가 너무 맛있어서 먹느라 사진을 못찍었다." width="100%" >}}

　올해 가장 잘했다고 생각하는 환경의 변화 중에 스터디 모임을 개설한 것을 빼먹을 수 없다. 회사 분들과 종종 자의반 타의 반으로 스터디를 진행했었지만 늘 함께하던 사람들이라 회사일이 바쁘다 보면 이런저런 이유들로 제대로 된 스터디 진행이 안되던 경우가 허다했다. (그렇게 책장에는 새로운 책들만 늘어만 가고...) 그런 갈증으로 블라인드에 익명으로 자바/백엔드 개발자 '성장' 스터디를 하자는 글을 쓰게 되었고 40여 명 넘게 지원을 하셔서 놀랬다. 비슷한 연차 + 비슷한 지역 + 다양한 회사 + 6명 내외라는 나만의 기준으로 선발(?)을 하게 되었고 서먹서먹했던 첫 온라인 미팅을 시작하여 벌써 3개의 책으로 스터디를 진행할 수 있었다. 온라인과 오프라인 모임을 적절하게 분배하고 중간마다 쉬어가는 주간도 만들었고 예치금과 벌금제도로 적절한 긴장감이 있는 상태로 스터디를 운영했다. 모인 금액으로는 스터디가 끝날 때 가벼운 식사 자리를 갖기도 하고. 아무래도 6명 모두 다른 회사라 각 회사의 대표(?)가 된 느낌의 책임감도 있었고 무엇보다 각 회사마다의 다른 기준을 간접적으로 경험할 수 있는 게 가장 좋았다. 내년에도 이 멤버와 함께 더 체계적이고 더 단단한 스터디로 업그레이드되어 그 스터디 결과물을 공유할 것 같다. 이 자리를 빌려 감사하다는 이야기를 전하고 싶다.

{{< image src="/images/review-2023/study-cafe.jpg" caption="새벽 6시에 가면 아무도 없어서 조용해서 좋다." width="40%" >}}

　환경을 바꾼 것 중에 또 한 가지는 새벽에 스터디 카페를 가는 것이다. 몇 년 전부터 시간이 부족함을 인지하고 미라클 모닝을 시도하고 있었다. 그러나 아침잠이 많은 나로서는 새벽 5시에 일어나긴 하지만 소파에 눕는다든지 다시 이불 속으로 들어가는 삶이 반복되었다. 일어나서 무언가를 해야 하는데 아무래도 '집'이라는 공간이 주는 편안함 때문에 무너지기 너무 쉬웠다. 늘 시간이 없다는 생각의 결론은 아침에 일찍 일어나 24시간 운영하는 스터디 카페에 가는 것으로 이어졌고 이때는 회사일이나 다른 것을 하는 게 아닌 온전히 "나"를 위한 시간으로 만들기로 하였다. 삼각대 기능이 있는 셀카봉을 일부러 가져가 타임랩스로 책상을 찍으며 강제적으로 휴대폰을 만지지 않게 되는 효과와 누군가 나를 보고 있다는 일석이조의 효과를 누리며 평소에 하지 못했던 책 읽기라든지 머릿속에 복잡한 생각들을 정리하는 시간을 갖기도 했다. (이렇게 찍은 영상들로 유튜브를 만들어 볼까 싶었지만... 과연...)

## 실패해서 배웠던 것들
　올해는 아쉽지만 두 가지의 실패의 기억들이 있다. 신기하게도 작년에 내 커리어리와 블로그를 통해 책을 써보지 않겠냐며 연락 온 출판사들이 꽤 많았다. 그렇지만 마음의 여유도 없었고 물리적인 시간이 없다는 핑계로 지금은 어렵다는 답장을 보내곤 했었는데 어떤 한 출판사 담당 직원분께서 짜임새 있게 기획서를 작성하셔서 책을 써보지 않겠냐고 제안을 주셨던 적이 있다. 다른 출판사와는 느낌이 달라서 힘든 상황이었지만 써보겠다고 여러 번의 미팅을 끝에 책을 쓰기 시작하였다. 그런데 막상 시작을 해보니 우려했던 것처럼 물리적인 시간이 정말 나지 않았다. (지금 생각해 보면 지금처럼 새벽에 쓰면 되었겠지 하는 아쉬움이 남는다.) 그래서 계약한 마감 날짜를 여러 차례 미루게 되었고 출판사 입장에서도 계속 미루는 수차례 배려해 주셨지만 결국 일정을 못 지켜 계약 파기가 되었다. (물론 일정을 맞춰야 한다는 게 스트레스가 되어 처음으로 탈모까지 왔지만;; 지금은 다행히 치료가 되었다.) 글쓰기를 좋아하는 나로서는 내 이름으로 책을 써보고 싶었던 상황이라 기회였는데 결국 내 복을 내가 차버리는 상황이라 지금 생각해도 나 자신에게 너무 밉다. 그 이후로도 또 다른 출판사에서 비슷한 형태로 제안이 와서 이번엔 제대로 해봐야지 하며 글의 방향성에 대해 몇 차례 미팅을 했지만 내가 생각하는 방향과 출판사에서 생각하는 간극을 좁히지 못하여 아쉽게도 무산되었다.

{{< image src="/images/review-2023/run.png" caption="들쭉날쭉 어설픈 기록들" width="80%" >}}

　올해 초부터 달리기라는 운동을 제대로 하게 되었다. 아내가 풀 마라톤을 두 번이나 완주했고 그때마다 늘 따라다니며 열정을 느껴보니 건강한 열정을 나도 느껴보고 싶었기 때문이다. 처음 뛸 땐 10km를 1시간 내로 달리긴 했지만 올해 총 4번의 10k 마라톤에 뛸수록 시간이 단축되기는커녕 점점 더 느려졌다. 나 달리기한다고 SNS나 주변 사람들에게 자신 있게 이야기했지만 정작 연습도 잘 안 했고 이런저런 핑계를 대며 꾸준히 못한 게 목표 달성 실패의 원인이지 않나 생각을 해본다.

## 내년의 키워드 : 꾸준함
　"지금이 아니면 안 돼"라는 내 좌우명처럼 언제나 최선을 다하며 살아간다. 그렇기에 올해 열심히 살았나?라는 질문엔 언제나 "YES!"라고 답할 수 있지만 앞서 이야기 한두 번의 실패를 통해 나의 부족한 점을 알게 되었던 나름의 의미 있던 시간이 아니었나 생각을 해본다. 그래서 내년의 키워드는 "꾸준함" 이 될 것 같다. 10여 년 넘게 개발자로 살아보니 홍길동처럼 이리저리 트렌드에 따라 움직이는 것도 방법일수 있겠지만 그보다는 나의 색깔을 드러내며 꾸준히 버티는 것도 중요하겠다는 생각이 요즘 들어 부쩍 든다.

{{< image src="/images/review-2023/water.JPG" caption="주차하다 발견한 작은 꾸준함의 결과물" width="60%" >}}

　여러 가지 유혹이나 변화에도 흔들리지 않고 온전히 내가 할 수 있는 영역들에 대해 회사에서 필요로 하는 사람이 되고 회사 밖에서도 영향력을 펼칠 수 있는 내가 되었으면 하는 상상을 하며 올해 2023 리뷰를 마친다.