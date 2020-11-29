---
title: Github과 Jenkins 연동하기
date: 2018-02-08 17:10:21
categories: [tech]
tags:
  - jenkins
  - github
  - archives-2018
url : /2018/02/08/github-with-jenkins/
---
Jenkins에서 Github의 소스를 가져와서 빌드를 하는 등 Github과 Jenkins와 연동을 시켜줘야하는 상황에서, 별도의 선행 작업이 필요하다. 다른 여러 방법이 있을수 있는데 여기서는 SSH로 연동하는 방법을 알아보고자 한다.<!-- more -->

우선 Jenkins가 설치되어있는 서버에서 인증키를 생성하자.

```
$ ssh-keygen -t rsa -f id_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_rsa.
Your public key has been saved in id_rsa.pub.
The key fingerprint is:
SHA256:~~~~~ ~~~@~~~~~
The key's randomart image is:
+---[RSA 2048]----+
|     o*+**=*=**+ |
|     o B=o+o++o  |
|     E+.o+ + oo .|
|      oo. * o ...|
|     .+ S  =   o |
|     . +    o .  |
|      . .  .     |
|       .         |
|                 |
+----[SHA256]-----+

$ ls
id_rsa  id_rsa.pub
```
개인키(id_rsa)는 젠킨스에 설정해준다. (처음부터 끝까지 복사, 첫줄 마지막줄 빼면 안된다... )

{{< image src="/images/github-with-jenkins/ssh_jenkins.png" src_l="/images/github-with-jenkins/ssh_jenkins.png" caption="젠킨스에 SSH 개인키 설정" >}}

그 다음 공개키(id_rsa.pub)는 Github에 설정을 해준다.

{{< image src="/images/github-with-jenkins/ssh_github.png" src_l="/images/github-with-jenkins/ssh_github.png" caption="Github에 SSH 공개키 설정" >}}

이렇게 한뒤 Jenkins 에서 임의로 job을 생성하고 job 설정 > 소스코드 관리 에서 git 부분에 아래처럼 테스트를 해서 정상적으로 연동이 된것을 확인한다. `Credentials` 값을 위에서 설정한 개인키로 설정하고, repo 주소를 SSH용으로 적었을때 에러가 안나오면 성공한것이다.

{{< image src="/images/github-with-jenkins/ssh_complete.png" src_l="/images/github-with-jenkins/ssh_complete.png" caption="정상 연결되면 Jenkins 오류도 없고, github SSH 키에 녹색불이 들어온다." >}}


끝~