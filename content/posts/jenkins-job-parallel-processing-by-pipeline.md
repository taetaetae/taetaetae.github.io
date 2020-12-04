---
title: 제목을 뭘로하지
date: 2020-12-03T08:19:47+09:00
draft: true
categories:
  - tech
featuredImage: /images/jenkins-job-parallel-processing-by-pipeline/pipeline.jpg
images :
  - /images/jenkins-job-parallel-processing-by-pipeline/pipeline.jpg
---

　Jenkins Job으로 운영하는 서비스의 url 모니터링을 한다고 가정해보자. 서비스가 커지고 그에 따라 모니터링 해야할 url 이 많아진다면 모니터링 하는 시간이 늘어나게 되면서 정해진 시간에 모니터링을 하려는 목적이 늘어난 시간때문에 의미가 없게 될 수도 있다. (가령 10분에 한번씩 체크를 해야하는데 모니터링 한번 실행하는 시간이 10분을 넘어서는 경우) 또한 모니터링 하려는 url 마다 응답속도가 다를 경우 (e.g. 국내/해외) 하나의 Job에서 모든 url 을 모니터링 한다는건 한계가 있을 수 있다.

　이번 포스팅에서는 Jenkins Job을 동시에 여러번 사용해야 하는 경우를 pipeline을 통해서 개선한 내용에 대하여 공유 해보려 한다. Jenkins pipeline 에 대해 들어만 봤는데 이번에 실제로 사용해보니 꽤 큰 개선을 경험할 수 있었고 옵션들을 상황에 맞게 조합을 잘 한다면 상당히 활용성이 높아보이는 기능인것 같다.

## 기존 상황

## 개선을 해보자

### Job을 범용적으로 (Jenkins paramters 활용)

### 병렬 실행을 위한 Jenkins 설정

### Jenkins Pipeline

## 개선 결과

## 마치며