---
title: Elastic{ON}Tour
date: 2017-12-14 12:02:45
tags: 
  - elasticsearch
  - archives-2017
categories:
  - review
url : /2017/12/14/elastic-on-tour/
featuredImage: /images/elastic-on-tour/ElasticStackWorkshop.jpg
images :
  - /images/elastic-on-tour/ElasticStackWorkshop.jpg
  
---
작년에 팀을 옮기면서 `로깅`에 대해서 관심을 갖기 시작 하였고 찾아보다 ElasticStack 이 적합하다고 판단, 팀 내에서 나홀로 삽질해가며 지금의 로그 모니터링 시스템을 구축하였다. 그에 ElasticStack 에 관심을 갖던 찰나 지난 화요일(12월 12일)에 있었던 Elastic On Tour에 참석을 하였고 다양한 기술적 인사이트를 얻을수 있었는데 그 감동(?)을 잃기 싫어 정리해보고자 한다.
<!-- more -->
## Registration + Partner Showcase
코엑스 인터컨티넨탈 호텔에서 진행되었다. 역시 외국계 기업이여서 그런지 행사 규모가 어마어마 했다. 이정표를 따라 지하로 가서 등록을 하고, ElasticStack 을 이용해서 서비스를 하고 있는 파트너사들의 부스를 기웃거리며 ElasticStack의 저력(?)을 다시한번 실감을 할수 있었다. 특히 Elatic 본사에서 나온듯한 외국인들이 Q&A 같은걸 해줬는데 답변을 해주는 외국인도 대단해 보였는데 질문을 하는 한국사람(?)들이 더 대단하게 보였다. 과연 난 저렇게 아무렇지 않고 프로페셔널(?)하게 질문을 할수 있을까?
{{< image src="/images/elastic-on-tour/people.jpg" caption="Registration + Partner Showcase" src_l="/images/elastic-on-tour/people.jpg" >}}

## Track 1 : Partner Sessions
Track 1 과 2로 나뉘였는데 2는 Elastic Stack 을 경험하지 못해봤거나 소개하는 자리같아서 Track 1를 듣기로 하였다. 내가 도입을 할때만 해도 관련 자료가 잘 없었고, 정말 특이 케이스가 아닌 이상엔 잘 사용하지 않겠구나 하는 느낌이였는데 발표하시는 분들을 보고서는 생각이 180도 바뀌었다. 너무 활용들을 잘 하며 서비스를 하고 있었고 단순하게 `검색엔진`이 아닌 상황에 맞는 커스터 마이징이나 다른 기술 스택을 함께 사용함으로써 시너지 효과를 내고 있었다.
- Microsoft
  - OpenSource 에 안좋은 이미지가 있으나 오래전부터 투자를 많이 해왔다고 설명을 하며 Azure라는 서비스에서 Elastic Stack 을 어떤식으로 활용하는지 발표를 하였다.
  - 상당히 심플하고 처음 접하는 사람도 클릭 몇번으로 ES Cluster를 구성할수 있다는게 장점이였으나, 유료 + 커스터마이징 제한 이 아쉬웠다.
- S-Core : 에스코어 경험에 기반한 Elastic 활용법 
- EZFarm
  - (외모 비하는 아니지만)농부 처럼 생기신 분이 나와서 기술에 대해 말씀하시는게 신기한 발표였다.
  - 간단히 말하면 돼지가 물 먹는 량 등 농업/축산업의 데이터를 ES에 담고 머신러닝을 통하여 효율화 하는 방안 이였던것 같다.
- MEGAZONE
  - 파트너 부스에서 티셔츠를 준(?) 곳이였는데 Elastic Cloud Seoul 을 발표하였다. (드디어 한국에도 이런 서비스가!)
- OpenBase
  - 키바나 플러그인을 직접 개발하고 커스텀 UI의 사례를 보여주었다. 
  - 키바나 소스중에 엑셀 다운로드가 한글로 안되어 고쳐본것 말곤 플러그인을 개발할 생각은 없었는데 정말 개발자 스러운 발표였다.
- DIREA
  - 결제 관련 장애추적 및 예측 시스템을 발표하였다. 
  - 마침 내가 하고있는 서비스와 비슷하고, 내가 구현해보려고 했던 부분과 거의 일맥상통한 부분이 있어서 소름이였다.

{{< image src="/images/elastic-on-tour/parter_Session.jpg" caption="Track 1 : Partner Sessions" src_l="/images/elastic-on-tour/parter_Session.jpg" >}}

## Opening Keynote
앞서 어떤 발표에서 ElasticSearch가 탄생하게된 계기가 어떤 분이 요리사가 되려는 아내를 위해 조리법을 더 빨리 검색할수 있는 엔진을 만들었다고 하는데 그 어떤분이 내눈앞에 나타나 발표를 하셨다. 현 Elastic CEO 이신 Shay Banon 이였다. (어색한 동시 통역으로 이해하였지만) 그분이 강조하신 Elastic 회사 정신인 "간단한건 간단하게 만들어야 하며 쉬워야 한다." 가 연설중에 가장 인상적이였고, 통역하신 아주머님(?)때문이였는지 전달하시는 의도를 정확히 파악하긴 어려웠으나 일단 CEO를 포함한 전체 회사 분위기가 젊어보인다는걸 느낄수 있었다.

{{< image src="/images/elastic-on-tour/key_note.jpg" caption="Shay Banon key note" src_l="/images/elastic-on-tour/key_note.jpg" >}}

## Break (식사시간)
이런 세미나? 컨퍼런스? 를 많이 다녀본건 아니지만 역대급으로 좋았던 점심식사였다. ;)
사실 혼자와서 밥을 어떻게 해결하나 했는데 우르르르 호텔 직원분들이 각 자리에 도시락을 대령(?)해주셔서 맛있게 먹을수 있었다. 
참 사람이 간사한게, 아침에 졸린눈 비벼가며 지옥철 고생을 뚫고 와서 힘들었지만 밥을 먹으면서 아침의 그 고생은 눈녹듯 사라졌다.

{{< image src="/images/elastic-on-tour/lunch.jpg" caption="호텔 도시락!!" src_l="/images/elastic-on-tour/lunch.jpg" >}}

## Deep Dive (Elasticsearch, Ingest, Kibana, Machine Learning)
각 스택(?)에 대해서 변화된 부분, 그리고 활용가능성과 최근 출시한 6.X 버전에 대해서 소개하는 시간이였다.  사실 아직 2.4 버전을 운영중이라서인지 6.X의 변화된 부분, 그리고 5.x 버전에서의 차이를 설명해주는데 확 와닿지는 못하였다. 아직 5.x 버전의 필요성을 못느껴서 버전업을 안하고 있었는데 발표를 듣고 꼭 유료 라이센스를 구매하지 않아도 용량 단축이나 안정성, 추가 기능등 5.X로의 버전업은 해서 나쁠것도 없을듯 하다는 생각이 들었다. (단, 무조건 신버전이 좋은것은 아닌듯 잘 알아보고 해야겠지...)
특히 키바나를 활용해 머신러닝을 눈앞에서 보여준건 너무 좋았다. 성능이 좋아서인지 데모시연 하는데 전혀 막힘이 없었으니...

{{< image src="/images/elastic-on-tour/machine_Learning.jpg" caption="노란색 음영이 머신러닝으로 계산한 그래프의 위치값들" src_l="/images/elastic-on-tour/machine_Learning.jpg" >}}

## Customer Presentations
Elastic Stack 및 X-Pack 을 활용하여 어떤 문제를 어떻게 해결하고 있는지에 대한 세션이였다. 삼성, NSHC, Naver 에서 약 20분씩 발표를 해주셨는데 약간 시간에 쫒기듯(?) 발표가 진행되어 집중이 잘 안되었다. 마지막에 우리 회사분이 발표하신 `Application Logging with Elasticsearch at Naver` 라는 주제는 실제로 내가 사용하고 있는 모듈인지라 관심갖고 들었다. 최근들어 자주 버벅이고 에러알림지연(기능중 하나)등 문제가 있었는데 이러한 부분들을 Multi-Cluster 등 튜닝을 통하여 해결했다는 부분을 설명해주셨다. (워낙에 양이 많으니...)

{{< image src="/images/elastic-on-tour/nelo.jpg" caption="NELO" src_l="/images/elastic-on-tour/nelo.jpg" >}}

## 마치며
밥도 맛있고 발표 내용들도 좋은 인사이트를 얻을수 있었고, 마지막에 경품추첨등 딱딱하지 않는 세미나 였다고 생각한다. 올해는 무료로 하는 행사였지만 내년부터는 유료라고 하는데 난 내년에도 사비를 내서라도 갈 생각이다. (회사지원이 된다면 좋겠지만 ^^;) 한가지 아쉬운건 너무 준비 없이 갔다는게 아쉽다. 사실 구버전(?) 2.X를 사용중이고 5.x 6.x 를 전혀 경험해보지 못해서 공감가는 부분이 없었는데, 나름 높은 버전으로 올려야 겠다는 생각을 할수있었던 좋은 시간이였던것 같다.