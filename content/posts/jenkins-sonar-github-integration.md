---
title: 소나큐브 이용 코드 정적분석 자동화
date: 2018-02-08 20:10:54
categories: [tech]
tags:
  - SonarQube
  - jenkins
  - github
  - integration
  - archives-2018
url : /2018/02/08/jenkins-sonar-github-integration/
featuredImagePreview : /images/jenkins-sonar-github-integration/concept.png
---
`코드 정적분석`이라 함은 실제 프로그램을 실행하지 않고 코드만의 형태에 대한 분석을 말한다. 이를테면 냄새나는 코드(?)라던지, 위험성이 있는 코드, 미리 정의된 규칙이나 코딩 표준을 준수하는지에 대한 분석을 말하는데 java 기준으로는 아래 다양한 (잘 알려진) 정적분석 도구들이 있다.<!-- more -->
- PMD
  - 미사용 변수, 비어있는 코드 블락, 불필요한 오브젝트 생성과 같은 Defect을 유발할 수 있는 코드를 검사
  - https://pmd.github.io
- FindBugs
  - 정해진 규칙에 의해 잠재적인 에러 타입을 찾아줌
  - http://findbugs.sourceforge.net
- CheckStyle
  - 정해진 코딩 룰을 잘 따르고 있는지에 대한 분석
  - http://checkstyle.sourceforge.net

이외에 `SonarQube` 라는 툴이 있는데 개인적으로 위 알려진 다른 툴들의 종합판(?)이라고 생각이 들었고, 그중 가장 인상깊었던 기능이 github과 연동이 되고 적절한 구성을 하게 되면 코드를 수정하는과 동시에 자동으로 분석을 하고 리포팅까지 해준다는 부분이였다. ( ~~더 좋은 방법이 있는지는 모르겠으나~~ 다른 도구들은 수동으로 돌려줘야 하고 리포팅 또한 Active하지 못한(?) 아쉬운 점이 있었다.  )

지금부터 Jenkins + github web-hook + SonarQube 를 구성하여 코드를 수정하고 PullRequest를 올리게 되면 수정한 파일에 대해 자동으로 정적분석이 이뤄지고, 그에대한 리포팅이 해당 PullRequest에 댓글로 달리도록 설정을 해보겠다. (코드리뷰를 봇(?)이 자동으로 해주는게 얼마나 편한 일인가...)

## 기본 컨셉
전체적인 컨셉은 다음 그림과 같다.

{{< image src="/images/jenkins-sonar-github-integration/concept.png" src_l="/images/jenkins-sonar-github-integration/concept.png" caption="전체 컨셉" >}}

1. IDE에서 코드수정을 하고 remote 저장소에 commit & push를 한다. 
  그 다음 github에서 master(혹은 stable한 branch)에 대해 작업 branch를 PullRequest 올린다.
2. 미리 등록한 github의 web-hook에 의해 PullRequest 정보들을 jenkins에 전송한다. 
3. 전달받은 정보를 재 가공하여 SonarQube로 정적분석을 요청한다.
4. SonarQube에서 분석한 정보를 다시 jenkins로 return 해준다.
5. SonarQube으로부터 return 받은 정보를 해당 PullRequest의 댓글에 리포팅을 해준다.

간단히 보면 (뭐 간단하니 쉽네~) 라고 볼수도 있겠지만 나는 이런 전체 흐름을 설정하는데 있어 어려웠다.
> 사실 셋팅하는 과정에서 적지않은 삽질을 했었기에, 이 포스팅을 적는 이유일수도 있겠다. 
더불어 검색을 해봐도 이렇게 전체흐름이 정리된 글이 잘 안보여서 + 내가 한 삽질을 다른 누군가도 할것같아서(?)

## Maven 설치
기본적으로 Maven의 H2DB를 사용하므로 SonarQube를 설치하기전에 Maven부터 설치해줘야 한다.
```
$ wget http://apache.mirror.cdnetworks.com/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
$ tar -zxvf apache-maven-3.5.2-bin.tar.gz
(환경변수 셋팅후 )
$  mvn -version
Apache Maven 3.5.2 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T16:58:13+09:00)
...
```

## SonarQube 설치
정적분석을 도와주는 SonarQube를 설치해보자. 
```
$ wget https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-6.7.1.zip
$ unzip sonarqube-6.7.1.zip
$ cd sonarqube-6.7.1/bin/linux-x86-64
$ ./sonar.sh start
Starting SonarQube...
Started SonarQube.
```
기본적으로 9000포트를 사용하고 있으니 다른포트를 사용하고자 한다면 /sonarqube-6.7.1/conf/sonar.properties 내 `sonar.web.port=9000` 을 수정해주면 된다. (SonarQube도 Elasticsearch를 사용하구나...) 
설치후 실행을 한뒤 `서버IP:9000`을 접속해보면 아래 화면처럼 나온다. (혹시 접속이 안된다거나 서버가 실행이 안된다면 `./sonar.sh console`로 로그를 보면 문제해결에 도움이 될수도 있다. )

{{< image src="/images/jenkins-sonar-github-integration/sonar_main.png" src_l="/images/jenkins-sonar-github-integration/sonar_main.png" caption="SonarQube 메인화면" >}}

## SonarQube Scanner 설치
소스를 연동시켜 정적분석을 하기 위해서는 SonarQube Scanner 라는게 필요하다고 한다. 아래 url에서 다운받아 적절한 곳에 압축을 풀어두자.
https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner
```
$ wget https://sonarsource.bintray.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.0.3.778-linux.zip
$ unzip sonar-scanner-cli-3.0.3.778-linux.zip
```

### # jenkins 설치 및 SonarQube 연동
jenkins 설치는 간단하니 별도 언급은 안하고 넘어가...려고 했으나, 하나부터 열까지 정리한다는 마음으로~
https://jenkins.io/download/ 에서 최신버전을 tomcat/webapps/ 아래에 다운받고 server.xml 을 적절하게 수정해준다.
```
$ wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
$ vi tomcat/conf/server.xml
<Connector port="19001" protocol="HTTP/1.1" # 포트 변경
<Context path="/jenkins" debug="0" privileged="true" docBase="jenkins.war" /> #추가
# tomcat/bin/startup.sh
```

jenkins 설치를 완료 한 후 필요한 플러그인을 추가로 설치해준다.
- Python Plugin
- GitHub Pull Request Builder
- GitHub plugin

접속 : `서버IP:19001` (참고로 한 서버에서 다 설치하다보니 port 충돌을 신경쓰게되었다. )
처음 jenkins를 실행하면 이런저런 설정을 하는데 특별한 설정 변경없이 next버튼을 연신 눌러면 설치가 완료 되고, SonarQube를 사용하기 위해 `SonarQube Scanner for Jenkins`라는 플러그인을 설치해주자. (이건 각 버전마다 궁합(?)이 안맞을수도 있으니 확인이 필요할수도 있다. 내가 설치한 버전은 jenkins 2.89, SonarQube Plugin 2.6.1이다.)
설치를 하면 jenkins > configure 에서 `SonarQube servers`정보를 등록해준다.

{{< image src="/images/jenkins-sonar-github-integration/jenkins_sonarqube_config.png" src_l="/images/jenkins-sonar-github-integration/jenkins_sonarqube_config.png" caption="SonarQube 젠킨스 설정" >}}

authentication token 은 SonarQube에 처음 접속했을때 아래화면을 볼수 있는데, 여기서 authentication token을 발급받을수 있다. 또한 language 와 build 방법을 선택하자. (이부분은 나중에 변경이 가능하니 개발상황에 맞춰서 설정하면 될듯하다.)

{{< image src="/images/jenkins-sonar-github-integration/authentication.png" src_l="/images/jenkins-sonar-github-integration/authentication.png" caption="authentication token 발급" >}}

그 다음 jenkins > configureTools 에서 SonarQube scanner 정보를 다음 화면과 같이 등록해준다.

{{< image src="/images/jenkins-sonar-github-integration/jenkins_sonarqube_scanner.png" src_l="/images/jenkins-sonar-github-integration/jenkins_sonarqube_scanner.png" caption="SonarQube scanner 젠킨스 설정" >}}

우선 여기까지 하면 jenkins 와 SonarQube 연동은 된걸로 보면 된다.

## Github과 Jenkins 연동
앞서 설명한 전체 컨셉 그림과 같이 github 에 pullRequest가 발생하면 web-hook을 이용하여 jenkins로 정보를 전달하는 방식인데, 이럴려면 우선 github과 jenkins가 연동이된 상태여야 한다. 관련 방법은 아래 링크에 정리를 해놨으니 참고해도 좋을듯 하다.
> [Github과 Jenkins 연동하기](/2018/02/08/github-with-jenkins)

## Jenkins job 등록
Jenkins job은 두개를 만들어 줘야 한다.
- 1번 job `pullrequest_receiver` : github으로부터 web-hook을 통해 pullRequest정보를 받는 job
- 2번 job `sonaqube-job` : 1번 job으로 부터 정보를 받아 SonarQube를 이용해 정적분석후 해당 pullRequest에 댓글로 리포팅 하는 Job

### 1번 job
편의상 job 이름은 `pullrequest_receiver`으로 정하였고, 매개변수로 `String Parameter`에 이름은 `payload`라 정하였다. 그리고 Build 에서 Execute Python 으로 아래 코드를 실행하도록 하였다.
```python
import  os, json, sys, requests

url = 'http://~~~/jenkins/job/sonaqube-job'
hookData = json.loads(os.environ['payload'])
branch = hookData['pull_request']['head']['ref'];
prId = hookData['pull_request']['number'];
componentName = hookData['repository']['name'];

if action == 'closed' or action == 'synchronize' or action == 'assigned' : # 이 부분은 적절하게 수정이 필요하다.
  sys.exit(0)

repository = hookData['repository']['full_name']

url = url + '/buildWithParameters?branch=' + branch + '&pr=' + str(prId) + '&repository=' + repository + '&componentName=' + componentName

response = requests.post(url)
```
WebHook을 이용해서 Jenkins Job을 실행시키는 과정은 아래 링크에 별도로 정리해놨으니 참고해도 좋을듯 하다.
> [Github의 WebHook을 이용하여 자동 Jenkins Job 실행](/2018/02/08/github-web-hook-jenkins-job-excute)

위와 같이 설정을 하고 pullRequest가 발생을 하면 해당 Job이 실행이 되면서 정보들을 적절하게 조합하여 2번 job으로 보내게 된다. 여기서 job을 하나로 나누지 않고 `github정보를 받는 job`, `SonarQube 분석 job` 이렇게 두개로 나눈 이유는 여러 Repository에 대해서 셋팅을 하고 싶은경우 Repository에서는  `1번 job`으로 쏘게 하고 `1번 job`내부에서 payload의 정보를 분석하여 Repository별 SonarQube 분석 job url로 보내면 되기 때문이다. 1번 job 은 하나, 2번 job은 Repository 별로 분기 되는 구조.

### 2번 job
SonarQube job을 작성할 차례다. 1번 job에서 파라미터를 보내주기 때문에 역시 2번 job에서도 파라미터를 미리 받도록 설정해주자.
- 파라미터 목록
  - pr
  - branch
  - repository
  - componentName

이 job에서는 SonarQube 분석이 되는데, 해당 repo에 대한 접근권한이 있어야 하기 때문에 앞서 설정한 `Github과 Jenkins 연동` 방법으로 Repository 와 Credentials 을 지정해주고, 브랜치 경로 또한 파라미터로 받은 값으로 지정해 준다.

{{< image src="/images/jenkins-sonar-github-integration/jenkins_config_source.png" src_l="/images/jenkins-sonar-github-integration/jenkins_config_source.png" caption="소스코드 관리 설정" >}}

마지막으로 최종 목적이였던 SonarQube 분석을 할 차례인데, 이번에 SonarQube가 버전업이 되면서 java 기분 .java 파일만 가지고 하는게 아니라 .java파일을 컴파일 하면 나오는 .class파일 경로를 필수로 지정해줘야 한다고 한다. (예전에는 .class 파일 경로를 지정하지 않아도 되었는데 홈페이지 설명에 의하면 바이너리 파일까지 같이 분석하게 되면 분석 신뢰도가 높아진다고 한다.)
https://docs.sonarqube.org/display/PLUG/Java+Plugin+and+Bytecode

> 딴 이야기로, 난 여태까지 정적분석이라는건 코드만을 가지고 하는줄 알았다. 자바 기준에서는 .java . 하지만 이번에 SonarQube 설정을 하면서 알게된 부분으로, 정적분석이라는건 맨 위에 적어놨듯 `실제 프로그램을 실행하지 않고 코드만의 형태에 대한 분석` 이기 때문에 java 기준에서는 .class 파일또한 분석의 대상으로 볼수가 있다. (실제로 구버전 - .java만 분석 / 신버전 - .java + .class 분석 을 해봤더니 불필요한 분석결과(?)가 줄어든것을 알수 있었다. )

그래서 결국 컴파일을 한 뒤에 SonarQube 분석을 하면 되겠다. 

{{< image src="/images/jenkins-sonar-github-integration/build_sonarqube_config.png" src_l="/images/jenkins-sonar-github-integration/build_sonarqube_config.png" caption="빌드 및 SonarQube 분석 설정" >}}

SonarQube 설정파일은 다음과 같다.
```
sonar.projectKey=${componentName} # 유니크한 프로젝트 키 네이밍 값
sonar.projectVersion=0.1
sonar.sourceEncoding=UTF-8 
sonar.analysis.mode=preview 
sonar.github.repository=${repository}
sonar.github.endpoint=https://api.github.co
sonar.github.login= # github login id
sonar.github.oauth= # github 개인키
sonar.login=admin
sonar.password=admin 
sonar.github.pullRequest=${pr}
sonar.host.url= # SonarQube url
sonar.issuesReport.console.enable=true 
sonar.github.disableInlineComments=true
sonar.sources=.
sonar.exclusions=
sonar.java.binaries=target/classes # 빌드 결과물 경로
```

이렇게 긴~~과정을 거치고 나면 pullRequest 를 올릴때마다 아래 그림처럼 수정한 파일에 대해서 자동으로 분석을 하여 알려준다. 
{{< image src="/images/jenkins-sonar-github-integration/sonar_result_report.png" src_l="/images/jenkins-sonar-github-integration/sonar_result_report.png" caption="SonarQube 분석결과" >}}


또한 분석결과에 따라 (이 부분은 SonarQube 설정으로 조정이 가능할것같다) merge 가 가능/불가능이 조정되어 진다. 

{{< image src="/images/jenkins-sonar-github-integration/sonar_result.png" src_l="/images/jenkins-sonar-github-integration/sonar_result.png" caption="SonarQube 분석 중 critical 항목이 없어서 merge 가능한 화면" >}}

## 마치며
반복적인 일에 대해서 자동화 구성을 한다는것은 참 의미있는 일인것 같다. 팀내 이런 구성을 도입하구서 획기적으로 코드 품질이 나아지진 않았지만 자칫 잘못해서 merge가 되는 위험한 코드들은 어느정도 SonarQube가 잡아주는것 같다. 사족이지만, 이렇게 적고나니 간단한데 이런 구성에 대한 정보를 알기까지는 엄청난 시간이 들었기에... 오랜만의 포스팅에 대한 뿌듯함을 다시한번 느낀다.
