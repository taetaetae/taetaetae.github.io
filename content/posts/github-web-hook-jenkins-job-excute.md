---
title: Github의 WebHook을 이용하여 자동 Jenkins Job 실행
date: 2018-02-08 17:10:54
categories: [tech]
tags:
  - jenkins
  - github
  - Webhook
  - archives-2018
url : /2018/02/08/github-web-hook-jenkins-job-excute/

---
PullRequest가 발생하면 알림을 받고싶다거나, 내가 관리하는 레파지토리에 댓글이 달릴때마다 또는 이슈가 생성될때마다 정보를 저장하고 싶다거나. 종합해보면 `Github에서 이벤트가 발생할때 어떤 동작을 해야 할 경우` Github에서 제공하는 Webhook 을 사용하여 목적을 달성할 수 있다.
아 당연한 이야기이지만 언급하고 넘어갈께 있다면, Github에서 Jenkins Job을 호출하기 위해서는 Jenkins가 외부에 공개되어 있어야 한다. (내부사설망이나 private 한 설정이 되어있다면 호출이 안되어 Webhook기능을 사용할 수 없다.)

## Jenkins Security 설정
Jenkins Job을 외부에서 URL로 실행을 하기 위해서는 아래 설정이 꼭 필요하다. (이 설정을 몰라서 수많은 삽질을 했다.)
CSRF Protection 설정 체크를 풀어줘야 한다. 이렇게 되면 외부에서 Job에 대한 트리거링이 가능해 진다.

{{< image src="/images/github-web-hook-jenkins-job-excute/jenkins_config.png" src_l="/images/github-web-hook-jenkins-job-excute/jenkins_config.png" caption="Jenkins > Configure Global Security" >}}

## Jenkins Job 설정
Github 에서 Webhook에 의해 Jenkins Job을 실행하게 될텐데, 그때 정보들이 `payload`라는 파라미터와 함께 `POST` 형식으로 호출이 되기 때문에 미리 Job에서 받는 준비(?)를 해둬야 한다.
설정은 간단하게 다음과 같이 Job 파라미터 설정을 해주면 된다.

{{< image src="/images/github-web-hook-jenkins-job-excute/jenkins_job_param.png" src_l="/images/github-web-hook-jenkins-job-excute/jenkins_job_param.png" caption="Jenkins > 해당 Job > configure" >}}

## Github Webhook 설정
이제 Github Repository 의 Hook 설정만 하면 끝이난다. 해당 Repository > Settings > Hooks 설정에 들어가서 `Add webhook`을 선택하여 Webhook을 등록해준다. 
URL은 `{jenkins URL}/jenkins/job/{job name}/buildWithParameters`식으로 설정해주고 Content Type 은 `application/x-www-form-urlencoded`으로 선택한다. 언제 Webhook을 트리거링 시킬꺼냐는 옵션에서는 원하는 설정에 맞추면 되겠지만 나는 `pullRequest`가 등록 될때만 미리 만들어 놓은 Jenkins Job을 실행시킬 계획이였으니 `Let me select individual events.`을 설정하고 `Pull Request`에 체크를 해준다. 아래 그림처럼 말이다.

{{< image src="/images/github-web-hook-jenkins-job-excute/oss_webhook.png" src_l="/images/github-web-hook-jenkins-job-excute/oss_webhook.png" caption="해당 Repositroy > Settings > Hooks" >}}

이렇게 등록하고 다시 들어가서 맨 아랫 부분`Recent Deliveries`을 보면 ping test 가 이루어져 정상적으로 응답을 받은것을 확인할수가 있다.

{{< image src="/images/github-web-hook-jenkins-job-excute/oss_webhook_result.png" src_l="/images/github-web-hook-jenkins-job-excute/oss_webhook_result.png" caption="Webhook 등록 결과" >}}

이렇게 설정을 다 한 뒤 PullRequest를 발생시키면 Jenkins 해당 Job에서는 파라미터를 받으며 실행이 된것을 확인할수가 있다.

{{< image src="/images/github-web-hook-jenkins-job-excute/jenkins_webhook_result.png" src_l="/images/github-web-hook-jenkins-job-excute/jenkins_webhook_result.png" caption="Jenkins Job 실행 결과" >}}

끝~