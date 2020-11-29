---
title: Jenkins 업그레이드 및 Master-Slave 구성
date: 2019-03-17 18:23:03
categories:
  - tech
tags: 
  - jenkins
url : /2019/03/17/jenkins-upgrade-master-slave/
featuredImage: /images/jenkins-upgrade-master-slave/jenkins.jpg
---
어떠한 작업(Job)이 있다고 가정해보자. 이를 "정해진 시간에 주기적" 이나 "필요할때" 작업을 수행하고 싶다면 어떤 툴(Tool)이 떠오르는가? <!-- more -->그리고 이 작업(Job)들의 실행이력 등 전체적으로 관리하고 필요에 따라 다양한 플러그인을 활용하여 입맛에 맞는 작업(Job)으로 구성하고 싶을때 가장 첫번째로 떠오르는 툴은 바로 "Jenkins" 다. (극히 필자 개인적인 생각일수도 있지만... ) 물론 리눅스 기반의 crontab 이나 다른 스케쥴러를 활용할수도 있다. 다만 필자 개인적인 느낌으로 나만의 Jarvis(?)처럼 내가 원하는데로 설정만 해두면 정해진 시간에 수행하고 그 결과를 로그로 남겨놓고 문제가 발생했을때 알림도 받을수 있으니 너무 좋은 툴이라 생각이 든다.

{{< image src="/images/jenkins-upgrade-master-slave/ColossalSociableBuffalo-size_restricted.gif" caption="실제로 Jarvis가 있다면 얼마나 편할까<br>출처 : https://gfycat.com/ko/colossalsociablebuffalo" width="80%" >}}

지난 [포스팅](https://taetaetae.github.io/2018/12/02/jenkins-install/)에서는 Jenkins 를 설치하는 방법에 대해 알아보았다. (정확히 말하면 치트키 수준의... ) 이번 포스팅에서는 Jenkins에 노드를 추가하여 master-slave 분산환경으로 구성하는 방법과 Jenkins 버전을 업그레이드 하는 방법에 대해 정리해보고자 한다.

> 마침 필자의 팀에서 젠킨스를 분산환경으로 운영하고 있었는데 버전은 1.x ... 간헐적으로 Jenkins 버전 이슈로 에러가 발생해서 업그레이드를 해야하는 상황이 생긴것이다. 시키지도 않은 일을 하면서 팀에 도움도 될겸, 포스팅도 할겸, 1석 2조 효과. 서버 환경은 CentOS 7.4 64Bit 에서 테스트 하였다.

## Jenkins 버전 업그레이드 하기
Jenkins를 업그레이드 하게되면 기존에 있었던 Jenkins의 환경설정은 어떻게 마이그레이션 할까? Job 실행기록들은 그냥 날려버려야 하나? 걱정을 하며 구글링을 해본다. 그러면 "안해본것에 대한 두려움" 을 갖는 필자의 마음이 무색할 정도로 너무 간단하게도 그냥 기존에 있던 war 파일을 최신버전으로 교체하고 재시작 하라고 나온다.  읭? 뭐이리 간단해? 대부분의 문제들은 지레 겁부터 먹고 실행에 옮기지 ~~못해서~~ 않아서 해결을 하지 못하는게 절반 이상같다.  자, 바로 실행에 옮겨보자.
우선 버전 업그레이드를 테스트 하기 위해 일부러 [낮은버전](http://mirrors.jenkins.io/war-stable/)으로 설치를 해둔다. (필자는 1.609.1로 설치해봤다.) 그리고 버전 업그레이드 후 설정이 그대로 옮겨지는지를 확인하기위해 Security 설정을 해서 Jenkins 접근시 로그인 여부를 물어보록 설정해둔다.

{{< image src="/images/jenkins-upgrade-master-slave/old_jenkins.jpg" caption="우측 하단에 빨간영역으로 낮은버전이 설치된것을 확인할수 있다." width="80%" >}}

설정이 완료되었으면 최신버전의 war를 다운받아 교체하고 재시작을 해준다. 그러면 너무나도 간단하게 버전이 업그레이드가 된것을 확인할수 있다. 그리고 처음에 설정한 Security 설정까지 그대로 유지되는것 또한 확인이 가능하다. 물론 구 버전에서 설치되었던 플러그인들이 버전업이 되며 그에 따라 지원하지 않는 문제들이 생길 수 있는데 이 부분은 플러그인을 업그레이드를 해준다거나 각 상황에 맞는 대응을 해줘야 한다. 이렇게 해서 생각보다(?) 너무 간단하게 버전업이 완료되었다.

{{< image src="/images/jenkins-upgrade-master-slave/upgrade_complete.jpg" caption="업그레이드 후 플러그인 업그레이드도 동일하게 맞춰주는게 중요하다." width="80%" >}}

## Jenkins 분산환경 구성하기 (노드 추가하기)

이번엔 Jenkins를 분산환경으로 구성해보고자 한다. 이렇게 노드를 추가하며 분산환경을 구성하는 이유는 마스터-슬레이브(Master-Slave) 패턴의 장점을 얻고자 함이다. 마스터는 작업을 쪼개고 슬레이브로 구성된 노드에게 분배를 하게되면 슬레이브 서버는 마스터의 요청을 처리하고 리턴하게 된다. 마치 스타크래프트에서 일꾼을 늘려서 미네랄과 가스를 더 빨리 얻는것처럼 말이다.

여기서 필자가 가장 많이 삽질한 부분. 슬레이브 서버를 추가하는데 슬레이브 서버가 되는 서버에 동일하게 젠킨스를 설치하고 그들을 모두 연결하려 했던것... 마치 클러스터링 하는것처럼...  당연히 Jenkins 들의 묶음형태(?) 가 되야 할것같은 생각으로 시도하였지만 엄청난 삽질의 연속이 되어버렸다. 알고보니 마스터 Jenkins에서 슬레이브 서버에 작업을 전달할수 있도록 연동만 시켜주면 자동으로 Agent를 마스터 Jenkins가 슬레이브 서버에 설치/실행을 하고 작업을 분할하는것을 확인할 수 있었다. 자, 그럼 시작해보자.

1. 마스터 서버에서 공개키와 개인키 생성
먼저 마스터 서버와 슬레이브 서버를 SSH로 통신할수 있도록 SSH 키 설정을 해준다. 통상 홈 디렉토리 하위 .ssh 폴더에서 생성한다.
```shell
ssh 키 생성
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/~/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /~/.ssh/id_rsa.
Your public key has been saved in /~/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ user@hostname
The key's randomart image is:
+---[RSA 2048]----+
|oo. .            |
|o... o  +        |
|. .o  o+.o       |
|.++++. +o+o..    |
|o.+*=.o.SEoo=    |
| .  o+.*...+ +   |
|   .. + +.  +    |
|     + .     .   |
|      ...        |
+----[SHA256]-----+

공개키 확인
$ cat id_rsa.pub
ssh-rsa AAAAB3Nza~~~~~~~~eQKcx8B6uAflRm1J8In1 user@hostname
```

2. 슬레이브 서버에서 마스터 서버에서 만든 공개키를 등록
슬레이브 서버에서는 마스터 서버에서 SSH 접속을 허용해야 하기때문에 마스터 서버에서 생성한 공개키를 등록해준다. 슬레이브 서버의 홈 디렉토리 하위 .ssh 폴더아래 파일을 만들고 위 공개키를 넣어주자.
```shell
$ vi authorized_keys
ssh-rsa AAAAB3Nza~~~~~~~~eQKcx8B6uAflRm1J8In1 user@hostname
```
3. Jenkins 에서 Credentials 을 만들때 Private Key 설정을 "From the Jenkins master \~/.ssh"으로 설정한다. 나중에 이 정보로 인증을 처리한다.

{{< image src="/images/jenkins-upgrade-master-slave/jenkins_upgrade_3.jpg" caption="" width="80%" >}}

4. 노드를 추가하고 조금 있으면 마스터 노드가 슬레이브 서버에 에이전트를 설치/실행하고 연동이 된것을 확인할수 있다.

{{< image src="/images/jenkins-upgrade-master-slave/jenkins_upgrade_4.jpg" caption="" width="80%" >}}

실제로 슬레이브 서버에서 프로세스를 확인하면 아래처럼 에이전트( slave.jar )가 설치/실행되고 있는것을 확인할수 있다.
```ssh
$ ps -ef | grep java
user   105431 105288  0 01:03 ?        00:00:00 bash -c cd "/home" && java  -jar slave.jar
user   105463 105431  3 01:03 ?        00:00:08 java -jar slave.jar
```
위와 같은 방법으로 슬레이브 서버를 총 두개를 구성하고 job을 여러개 실행하게 하면 자동으로(랜덤으로) 분배되어 실행하는것을 확인할 수 있다. 

{{< image src="/images/jenkins-upgrade-master-slave/jenkins_node_job_execute.jpg" caption="" width="80%" >}}

특정 job은 특정 슬레이브 서버에서 실행하고 싶은 경우도 있다. 예로들어 특정 슬레이브 서버가 성능이 더 좋다거나 네트워크 ACL이 특정 슬레이브 서버만 오픈되었다거나... 그럴 경우에는 아래처럼 job 실행설정에서 슬레이브를 강제로 지정할수도 있다. (짱...)

{{< image src="/images/jenkins-upgrade-master-slave/execute_target_node.jpg" caption="" width="80%" >}}

## Jenkins 분산환경에서 버전 업그레이드 하기
이제 위에서 했던것들의 종합 세트인 "Master-Slave로 되어있는 구성에서의 Jenkins 업그레이드" 를 해보자.  우선 위에서 했던것처럼 구버전으로 Master-Slave 를 구성한다.
이제부터가 중요한데 필자는 당연히 위에서 했던 업그레이드 방법처럼 (이렇게 노드가 연결되어있는 상황에서) 기존의 war을 교체하면 되겠거니 했다. 하지만 업그레이드는 되었지만 노드가 연결이 안되면서 너무나도 다양한(?)에러를 만나야만 했다. 에러 내용을 찾아보니 필자처럼 버전 업그레이드를 하며 예외상황이 발생해 에러가 나는 경우가 많았고 삽질을 거듭해본 결과 다음과 같은 방법으로 하면 업그레이드도 되고 노드도 연결이 가능한것을 확인할 수 있었다. (다른 더 좋은 방법이 있다면 알고싶다... )

1. 우선 기존에 추가해둔 노드들을 제거한다.
2. 그 다음 위에서 했던것처럼 war를 교체하며 업그레이드를 진행한다.
3. Credentials 항목에 보면 개인키가 있는것을 볼수 있다. (기존에는 "From the Jenkins master ~/.ssh" 항목이 있었는데 없어졌다. )
4. 위에서 했던것처럼 노드를 추가해준다. 그럼 다음과 같은 에러를 만날수 있는데 에러 내용을 보면 known_hosts 파일이 없다고 나온다. 뭔가 해결할수 있을것만 같은 느낌이 든다. master 서버에서 `ssh 슬레이브서버주소` 명령어를 실행해서 known_hosts 파일을 생성하도록 해준다.
```shell
$ pwd
/home/.ssh
$ ls
id_rsa  id_rsa.pub
$ ssh slave-host
The authenticity of host 'slave-host (0.0.0.0)' can't be established.
ECDSA key fingerprint is SHA256:0zb~~~~~B1A.
ECDSA key fingerprint is MD5:~~~~:87.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'slave-host,0.0.0.0' (ECDSA) to the list of known hosts.
Last login: Thu Mar 14 13:30:04 2019 from 10.113.219.197
[user@slave-host ~]$ exit
logout
Connection to slave-host closed.
$ ls
id_rsa  id_rsa.pub  known_hosts
$ cat known_hosts
slave-host,0.0.0.0 ~~~ AAAA~~~~~~~8=
```

{{< image src="/images/jenkins-upgrade-master-slave/final_upgrade.jpg" caption="업그레이드 후 노드 구성한 화면" width="80%" >}}

## 마치며
"Master-Slave로 되어있는 구성에서의 Jenkins 업그레이드"를 하며 정말 많은 시간을 할애할 수 밖에 없었고 (관련 지식도 없고 경험도 없었으니... ) 너무 안되어 포기할까도 싶었다. 하지만 경험하지 않은 모든 일들은 다 그만큼의 고통이 필요하고, 그 고통이 있어야지만 비로소 내것이 된다는 생각을 하고 있다. 이것도 나만의 무기가 되어 나중에 jenkins 를 업그레이드 한다거나 노드구성을 할때 보다 쉽고 빠르게 할수있지 않을까 기대를 해본다. 더불어 어려운 이야기이지만 삽질도 올바른 삽질을 할수 있도록 소망해본다...

{{< image src="/images/jenkins-upgrade-master-slave/spadework.gif" caption="이런 삽질은 그만... <br>출처 : https://gfycat.com/ko/illiterateonlyicelandgull" width="50%" >}}