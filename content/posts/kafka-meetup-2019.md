---
title: KafkaKRU(Kafka 한국사용자 모임) 밋업 후기
date: 2019-03-31 01:49:30
categories:
  - review
tags: 
  - kafka
url : /2019/03/31/kafka-meetup-2019/
featuredImage: /images/kafka-meetup-2019/kafka.jpg
images :
  - /images/kafka-meetup-2019/kafka.jpg
---

필자는 ElasticStack을 사용하면서 처음 카프카를 접하게 되었다. 메세징 큐 라는 개념도 전혀 모르는 상태에서 설치부터 ElasticStack 연동까지 사용하며 정말 `강제로` 카프카에 대해 공부를 하게 되었다. 카프카를 자주 다루고 메커니즘에 대해 자세히 살펴보다 잠깐 해이해질 무렵 카프카 한국 사용자 모임에서 밋업을 한다고 하길래 빛의 속도로 신청, 아마도 1등으로 신청했지 않았을까 싶다.<!-- more -->
사실 작년 카프카 밋업을 못간게 너무 한(?)이 되어 이번엔 회사 업무 등 여러가지로 한창 바쁘지만 "지금이 아니면 안돼" 라는 생각으로 밋업을 다녀왔고, 짧지만 후기를 작성해 보고자 한다.
> (요즘 왜 이렇게 바쁜지 모르겠지만... 신기하게도 그 바쁜 일정들이 하나도 겹치지 않는게 더 신기하다... )

{{< image src="/images/kafka-meetup-2019/first.jpg" caption="삼성 SDS 건물에서 진행된 카프카 밋업" width="80%" >}}

참고로 필자는 카프카에 대해 아주 조금 건드려본 수준이라 발표하시는 분들의 전부를 습득하기엔 다소 그릇이 작아서 일부 세션은 거의 "그런가보다~" 하고 들을 수 밖에 없었다. 후기도 아마 그런 맥락으로 작성할듯 싶다.
- Kafka 한국 사용자 모임 링크 : https://www.facebook.com/groups/kafka.kru

## 카프카를 활용한 캐시 로그 처리 - 김현준(카카오)
- 이미지 등 캐시서버의 로그를 분석하기 위한 시스템을 구축하는데 ElasticStack 을 활용
- Elasticsearch 로 늦게 들어와서 사례를 찾아보니 대용량 로깅 처리시 앞단에 메세징 큐를 둬야 한다고 했고 그게 카프카
- 카프카 모니터링은 그라파나로 활용
- lag이 자꾸 생김
    - 파티션을 쪼개거나, 컨슈머를 늘리는 방법이 있음
    - auto.commit.interval.ms 와 enable.auto.commit=true 로 조정
    - interval을 줄이니 lag이 줄어듬
- 현재는 수백대 캐시서버의 로그를 초당 15만건 이상 처리중

질문을 했다. 필자도 lag이 높아지면 어쩌지 하는 불안감과 높아지면 컨슈머를 늘리면 되겠지 하는 막연함이 있었는데 commit interval을 줄이면 lag이 줄어든다고 해서 무조건 줄이면 좋은가에 답변은 카프카를 관리하는 주키퍼쪽에 무리가 간다고 설명해 주셨다. 역시 만병통치약은 없고 상황에 따라 적절하게 시스템 관리자가 조정해가며 운영해야 하는점을 느꼈다.

- 참고 URL : [https://kafka.apache.org/documentation/#adminclientconfigs](https://kafka.apache.org/documentation/#adminclientconfigs)

## 카프카를 활용한 엘라스틱서치 실무프로젝트 소개 - 이은학(메가존)

- 카드사의 프로젝트를 약 3개월간 개발하였고 전체 아키텍쳐 중에 일부분을 kakfa를 활용
- Elasticsearch 데이터를 hadoop에 백업 형태로 옮기며 관리
- filebeat > kafka > spark streaming 을 활용하여 데이터의 검증처리가 가능 (특정 상황에서의 관리자에게 알림 등)
- logstash 의 ruby 필터를 활용하여 일정의 작업을 해주는 데이터 파이프라인 구성 가능 (개인정보 식별 등)
- logstash 는 cron형태의 배치로도 가능

또 질문을 하였다. (카프카 밋업과는 무관했지만...) logastsh 를 사용하면서 필터쪽에 로직이 들어가면 성능상 괜찮냐는 질문에 하루에 15억건을 처리하고있고 문제가 없었다고 한다. 필자는 아파치 엑세스 로그를 logstash로 처리하면서 간혹 뻗거나 에러가 발생했는데 아마 파일을 logstash가 직접 바라보고 처리도 하게해서 그런것 같다. (지금은 filebeat가 shipper 역활을 수행하고 있고 큰 무리 없이 운영중)

## 카프카를 활용한 rabbitMQ 로그처리 - 정원빈 (카카오)

- 레빗엠큐는 erlang으로 구현된 AMQP 메시지 브로커이고 TCP기반으로 구성
- Kafka 는 게으르지만 메우 효율성이 뛰어남, 반면 RabbitMQ 는 똑똑하지만 보다 느림
- Kafka 에서 Elasticsearch 로의 ingset 는 NIFI를 활용
- 레빗엠큐와 카프카의 차이


  | | Kafka | RabbitMQ |
  | --- | --- | --- |
  | 컨슈머 추가 | 여러 컨슈머가 하나의 메세지를 동시에 할수 있어 확장에 용이함 | 확장할때마다 큐를 추가 생성해야함 |
  | 메세지 저장 | 로그기반으로 디스크에 저장, 리텐션 이후 삭제 | 큐 기반으로 메모리에 저장 컨슈머가 메세지 수신시 즉시 삭제 |
  | 메세지 처리 | 발송확인 가능 / 수신확인 불가능 | 발송확인/수신확인 가능 |

## 카프카를 마이크로서비스 아키텍쳐에 활용하기 - 이동진 (아파치 소프트웨어 파운데이션)

- 카프카 스트림즈 소개 (Interactive Query)
- 카프카를 활용하여 마이크로서비스에서 사용하려면 데이터를 임시 공간에 넣어두고 (redis 같은?) 빼서 사용하는 형태가 아니라 Interactive Query 또는 Queryable Store 로 활용 가능

사실 이부분은 필자가 제대로 못따라간 세션중에 하나이다. 용어나 메커니즘도 다소 생소했고 대략 어떤 부분을 발표해주시는지 느낌은 있었으나 제대로 이해를 못해서 ...  부끄럽지만 카프카 스트림즈의 공식링크로 대체한다. 

[https://kafka.apache.org/documentation/streams/](https://kafka.apache.org/documentation/streams/)

## 카프카 프로듀서 & 컨슈머 - 강한구 (카카오 모빌리티)

- 프로듀서
    - 메세지를 생산 및 전송
    - Accumulator : 사용자가 send한 record를 메모리 쌓는 역활
    - Network thread : 전송
    - 각 옵션 활용법 (도큐먼트 문서로 대체)
        - [linger.ms](https://docs.confluent.io/current/installation/configuration/producer-configs.html#linger-ms)
        - [max.request](https://docs.confluent.io/current/installation/configuration/producer-configs.html#max-request-size)
        - [max.in.flight.requests.per.connection](https://docs.confluent.io/current/installation/configuration/producer-configs.html#max-in-flight-requests-per-connection)
- 브로커
    - 메세지를 저장
    - topic name - partition 폴더 구조
    - 세그먼트 단위로 저장 (\*.index, \*.log, \*.timeindex)
- 컨슈머
    - Fetcher : 네트워크 스레드와 비슷한 역할
    - Coordinator : 어떤 토픽의 어떤 파티션을 comsume할지, 브로커의 그룹 코디네이터와 통신 (hearbeat, offset comit, consumer group join)

## 마치며

{{< image src="/images/kafka-meetup-2019/meetup.jpg" caption="발표자 분들과 질문 두번에서 받은 책선물" width="80%" >}}

확실히 수박 겉핥기 식으로  보다보니 지식에 대한 깊이도 얕아 발표자분이 전달하시고자 하는 내용을 100% 다 수용하기엔 힘들었다. 다음엔 가기전에 미리 밋업 발표에 대한 공부를 조금이라도 하고 들을 준비를 한 뒤에 참여하는것으로... 하지만 카프카를 활용해서 다양한 시스템 구성 방법론에 대해 간접으로라도 배울수 있었고, 현재 필자가 운영하고 있는 카프카의 설정값들을에 대해 잘 설정이 되어있나 (막연히 기본값들로만 설정되어 있지는 않은가) 살펴볼 계기가 만들어진것 같다. 이번에도 `다행히` "행사에 참여하면 꼭 질문을 하나이상 하자!" 라는 나와의 약속을 지킬수 있어 다행이었다.

