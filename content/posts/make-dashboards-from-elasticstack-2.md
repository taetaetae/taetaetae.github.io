---
title: "Elastic Stack으로 코로나19 대시보드 만들기 - 2부 : 대시보드"
date: 2021-02-17T16:53:49+09:00
categories:
  - tech
tags: 
  - elasticsearch
  - logstash
  - kibana
  - filebeat
  - dashboard
  - archives-2021
featuredImage: /images/make-dashboards-from-elasticstack-2/logo.jpg
images :
  - /images/make-dashboards-from-elasticstack-2/logo.jpg
---

　[지난 포스팅](/posts/make-dashboards-from-elasticstack-1/)에서는 ﻿데이터를 수급하며 전처리 과정을 거쳤고, Filebeat와 Logstash를 거쳐 Elasticsearch에 인덱싱 하는 것까지 알아보았다. 앞선 포스팅에서 이야기했지만 단순하게 데이터를 시각화 도구를 이용해서 대시보드를 만드는 게 아니라 데이터가 추가되면 만들어둔 대시보드에 자동으로 반영되는 흐름을 만들고 싶었다. 마침 파이프라인을 이틀 전에 만들었기 때문에 그동안의 빠진 데이터를 추가해야 하는 상황이다. 이 경우 Filebeat-Logstash-Elasticsearch 가 실행 중이라면 앞서 작성했던 파이썬 스크립트만 한번 실행해 주면 이틀 치 데이터가 파이프라인을 거쳐 Elasticsearch로 인덱싱이 된다. 즉, 별도로 데이터를 가져와서 재 가공하고 추가하는 다소 까다로운 작업이 미리 만들어둔 파이프라인 덕분에 한 번의 스크립트 실행으로 손쉽게 처리가 됨을 알 수 있다. 

　이제는 쌓여있는 데이터를 가지고 시각화를 해볼 차례이다. ElasticStack에서는 Kibana라는 강력한 시각화 도구를 제공하는데 이번 포스팅에서는 Kibana를 이용해서 대시보드를 만드는 방법에 대해 알아보려 한다.

## Visualize
　﻿Elasticsearch에 인덱싱 되어있는 데이터들은 기본으로 제공되는 REST API를 통해서 조회할 수 있고 JSON 형태로 결과가 나오기 때문에 이를 가지고 다양하게 시각화를 할 수도 있다. 하지만 Kibana에서는 데이터를 조회하고 UI로 표현하는 일련의 모든 행위를 클릭 몇 번으로 할 수 있게 해주기 때문에 전문가가 아니더라도 조금만 만져보면 누구나 만들 수 있다.

{{< image src="/images/make-dashboards-from-elasticstack-2/visualize-main.jpg" width="80%" caption="New Visualizaion!!" >}}

　버전업이 되면서 비쥬얼라이즈를 만드는 첫 화면 또한 변화가 생겼다. 기존에는 어떤 유형의 비쥬얼라이즈를 선택할 것인지에 대해 선택하는 화면부터 나왔는데 만드는 걸 보다 편리하게 도와주는 `Lens`, `TSVB` 같은 기능들이 먼저 반겨준다. 이 기능을 통해서 만드는 방법도 괜찮지만 보다 명시적으로 만들고 싶으니 하단에 `Aggregation based`을 선택해서 원하는 비쥬얼라이즈의 타입을 선택해 보자. 이후 생성되어 있는 인덱스를 선택하면 본격적으로 비쥬얼라이즈를 그릴 수 있는 화면이 나오는데 대시보드 화면 기준으로 만들어야 할 항목별로 살펴보자.

### 전체 수
{{< image src="/images/make-dashboards-from-elasticstack-2/1.jpg" width="80%" caption=".">}}

　﻿확진자, 사망자, 격리 해제의 총합을 표현하려 한다. 이렇게 '숫자'를 표현하려 하는 경우 `Metric`을 활용하곤 한다. 우측에서 Aggregation 방법을 'sum'으로 설정하고 필드는 유형별로 각각 선택해 주자. 아래 'Add'버튼을 눌러 확진, 사망, 격리 해제 수를 모두 표시한 다음 저장을 눌러준다. Label을 지정하지 않으면 어떤 형태로 Aggregation을 했는지를 Label 영역에 보여주는데 그게 보기 싫다면 원하는 텍스트로 지정해 주는 것도 방법이다.

### 최근 수
{{< image src="/images/make-dashboards-from-elasticstack-2/2.jpg" width="80%" caption=".">}}

　﻿확진자, 사망자, 격리 해제의 '최근 데이터'를 보여주는 게 목적이다. 이 경우 Aggregation을 Top Hit으로 선택하면 필드를 선택할 수 있게 되는데 하루의 데이터가 총 18 row이기 때문에 (서울, 부산, ..., 제주, 검역) 18 row 을 전부 더한 값이 하루 기준의 합계가 된다. 여기서 정렬을 날짜 기준 내림차순으로 해줘야 가장 최근 데이터의 합계가 되는 점도 신경 써야 한다.

### 각 타입별 합계
{{< image src="/images/make-dashboards-from-elasticstack-2/3.jpg" width="80%" caption=".">}}

　﻿지역별로 타입별 수를 보기 위해 `Pie` 타입으로 선택하여 진행한다. 타입별(예로 들어 확진이면 confirmed)로 합계를 구하기 위해 Aggregation을 'sum'으로 설정하면 빈 원이 나오지만 각 지역별로 차트를 잘라서 봐야 하기에 하단의 Buckets의 Add를 누르고 regieon의 필드를 Terms Aggregation 한다. 18 row의 데이터가 전부 보여야 하기에 정렬 개수를 늘리고 option 탭에서 보는 취향에 알맞게 설정값들을 바꿔준다.

### 타입별 추이
{{< image src="/images/make-dashboards-from-elasticstack-2/4.jpg" width="80%" caption=".">}}

　﻿확진, 사망, 격리 해제 중에 사망을 제외하고 나머지 둘은 데이터의 크기가 크고 변화량이 비슷하기 때문에 x축은 시간으로 설정해두고 사망은 막대로, 나머지 둘은 라인으로 한 화면에서 표현하면 이 3가지 데이터를 한눈에 보기 좋을 것 같았다. `Vertical bar` 을 선택하고 x축(Buckets > X-axis)은 데이터 타입인 convert_date로 설정한다. 다음으로 사망은 매일 몇 명 사망했는지 뚜렷하게 보기 위해 그냥 sum으로, 나머지 둘은 누적 합계가 더 의미 있어 보일 것 같아 Cumulative Sum으로 Aggregation을 한다. 한 화면에서 3가지의 데이터를 보여주기엔 다소 y 축의 크기가 맞지 않으므로 확진, 격리 해제를 별도의 y 축(Y-axes)으로 'Metrics & axes'탭에서 설정해 준다. 다음으로 각 항목의 범례를 위로 표시하여 보다 보기 편하게 배치한다.

### 확진자 추이
{{< image src="/images/make-dashboards-from-elasticstack-2/5.jpg" width="80%" caption=".">}}

　﻿시간의 흐름에 따라 확진자가 얼마나 늘었는지, 더불어 그 안에서도 어떤 지역에서 많이 발생했는지를 한눈에 보고자 한다. 위와 동일하게 `Vertical bar`을 선택하고 x축에는 시간의 흐름을 보기 위해 convert_date을 가지고 Date Histogram Aggregation을 설정한다. 다음으로 y 축은 확진자 수를 sum 하고 (max를 해도 무방, 하루에 데이터가 하나기에 큰 의미가 없다.) 마지막으로 지역별로 하루의 막대를 쪼개야 하기에 Buckets를 추가로 만들어 regieon 기준으로 Terms Aggregation 을 한다. 그렇게 해놓고 보면 코로나19 바이러스가 처음 발생했을 때 대구에서 많이 발생한 것을 확인할 수 있고 그 이후에는 대부분 인구 분포가 많은 지역 순서로 확진자가 발생한 것을 알 수 있다.

## Dashboard

　﻿하나의 대시보드는 미리 만들어둔 비쥬얼라이즈를 조합하여 만들어진다. 각 비쥬얼라이즈의 크기나 위치 또한 자유롭게 설정할 수 있는 것이 가장 큰 장점이라 생각한다. 그럼 위에서 만들었던 비쥬얼라이즈를 가지고 배치를 해보자. 'Add from libray'을 누르면 위에서 만든 비쥬얼라이즈를 선택하는 화면이 뜨고 크기를 조절하고 적절하게 위치를 조정해서 입맛에 맞는 대시보드를 만들어 보자.

{{< image src="/images/make-dashboards-from-elasticstack-2/6.jpg" width="80%" caption="우측에서 비쥬얼라이즈를 선택!" >}}

{{< image src="/images/make-dashboards-from-elasticstack-2/8.jpg" width="80%" caption="대구는 처음에 확진자가 많이 발생했고 지금은 그에 비해 적다." >}}

{{< image src="/images/make-dashboards-from-elasticstack-2/9.jpg" width="80%" caption="조회 날짜를 조정해서 볼 수 있다." >}}

　Kibana로 만든 대시보드가 정말 강력한 부분이 이 상태에서 특정 지역의 상태를 보고 싶을 경우 범례에 있는 지역을 선택만 하면 손쉽게 화면이 바뀐다는 점이다. 또한 조회시간을 조절하여 집중으로 볼 수도 있다. 이러한 기능을 처음부터 다 만들어야지 생각을 하면 배보다 배꼽이 커지는 건 당연한 소리인 것 같다.

## 마치며
　﻿필자는 이러한 기능을 활용하여 실무에서 데이터의 전체 흐름을 한눈에 볼 수 있는 대시보드도 만들어 보았고, 날짜나 필드를 조합하여 데이터를 조회해야 하는 어드민을 직접 구현하지 않고 위와 비슷한 구성으로 만들어 개발 공수를 엄청나게 절약한 경험이 있다. 'ElasticStack이 짱이에요' 라는 말을 하고 싶은 건 아니다. 다양한 오픈소스를 적절하게 활용하면 적은 공수로 엄청난 효과를 누릴 수 있다는 장점이 있음을 말하고 싶다. 우리가 하고자 하는 건 데이터 분석이지 데이터를 어떻게 저장하고 시각화를 어떻게 해야 하는 건 둘째 문제 아니던가.

{{< image src="/images/make-dashboards-from-elasticstack-2/7.jpg" width="80%" caption="아직 위험하다!">}}

끝으로, 코로나19 바이러스가 하루빨리 종식되기를 간절히 바라며 어디 나가지 말고 집에서 필자처럼 대시보드를 만들고 추이를 지켜보는 건 어떨까...