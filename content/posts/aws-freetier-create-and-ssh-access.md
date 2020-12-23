---
title: AWS 프리티어 발급부터 EC2 접속까지
date: 2019-04-14 17:39:03
categories:
  - tech
tags: 
  - aws
  - ec2
  - putty
  - archives-2019
url : /2019/04/14/aws-freetier-create-and-ssh-access/
featuredImage: /images/aws-freetier-create-and-ssh-access/aws-ec2.jpg
images :
  - /images/aws-freetier-create-and-ssh-access/aws-ec2.jpg
---
IT 쪽에 일을 하고 있거나 관심을 가지고 있는 사람이라면 한번쯤을 들어봤을 AWS(Amazon Web Services). 이름에서도 알수있는 것처럼 아마존에서 제공하는 각종 원격 컴퓨팅 웹서비스이다. <!-- more --> 아마존은 이러한 서비스를 누구나 쉽게 접근해볼수 있도록 [AWS 프리티어](https://aws.amazon.com/ko/free/)를 제공해 주는데 이 프리티어 만으로도 과금없이 (또는 최소화 하여) 웹서비스를 구성할수 있다. 필자가 운영하고 있는 [기술블로그 구독서비스](http://daily-devblog.com)또한 AWS 프리티어로 운영되고 있다. 
최근 GDG Seoul, P-typer, Sketch Seoul 에서 주최한 [D.light 345 투게더톤](https://www.meetup.com/ko-KR/GDG-Seoul/events/259463050/)에 참가하며 사이드 프로젝트를 하고 있는데 마침 AWS를 사용하게 되었다. 예전에 사용했을때는 장님 코끼리 만지듯이 설정을 했었는데 이번기회를 통해 다시한번 정리를 해본다.
본 포스팅에서는 AWS 계정을 발급받고 신용카드 확인까지 된 계정에서 EC2 서버를 발급받고 putty를 활용하여 서버에 접근을 해보는것을 목표로 둔다. 
> (사이드 프로젝트를 하면서) 아마도 웹서비스를 개발하면서 AWS를 활용하는 부분에 대해 시리즈물로 포스팅을 하게 될것 같다.
사실 너무 간단해서 이런걸 글로 쓰나? 라고 할수도 있지만 눈으로만 보는것과 직접 해보는 것이 다르고, 이걸 다시 글로써 정리를 하는것 또한 완전 다른 부분이기 때문에 포스팅을 해본다.

## EC2 생성하기
EC2? Amazon Elastic Compute Cloud의 약자로 물리서버가 아닌 클라우드 서버를 제공하고 있다. EC2의 장점은 서버의 스펙을 쉽고 자유롭게 조정할 수 있는점이 가장 매력있게 생각한다. 우선 콘솔에 들어가 EC2를 검색후 접속을 하고 `인스턴스 시작`을 눌러서 인스턴스 생성 화면으로 들어간다.

{{< image src="/images/aws-freetier-create-and-ssh-access/ec2-1.jpg" caption="" width="80%" >}}

AMI 즉 생성할 이미지를 선택하는 부분인데 여기서 주의할점은 잘못선택 했다간 계정 만들었을때의 카드로 생각지도 못할 금액이 결제가 되버릴수도 있다. (실제로 필자도 AWS를 처음 만져볼때 아무생각없이 좋아보이는걸로 했다가 한 30달러 정도를 지불했어야만 했다...) 좌측에 보면 `프리 티어만`이라는 체크박스를 체크하고 자신이 원하는 이미지를 선택하자. 일반적인 리눅스 서버를 발급받고 싶기 때문에 빨간 영역의 이미지를 선택하고 선택한 이미지의 스팩을 다시한번 확인하자. (cpu 1개에 메모리도 1기가... 너무 짜지만 무료니까...)

{{< image src="/images/aws-freetier-create-and-ssh-access/ec2-2.jpg" caption="" width="80%" >}}

마지막으로 `시작하기` 를 누르면 키 페어를 선택 또는 생성하도록 안내가 나오는데 당연히 아무것도 안한 상태라 `새 키 페어 생성`을 선택해 주고 이름을 지정한뒤 키 파일을 받아준다. 이 부분에서도 조심해야할 점이 키 페어를 한번 다운 받으면 다시 동일한 키 페어를 다운받을수가 없게 된다. (나중에 다시 발급을 받아야 하는 번거로운 문제가...) 다운을 받고 잊어버리지 않도록 잘 보관해두자.

{{< image src="/images/aws-freetier-create-and-ssh-access/ec2-3.jpg" caption="" width="80%" >}}

키 페어를 다운 받으면 생성중이라는 메세지와 함께 결과화면이 나온다. 여기서도 중요한 부분! `프리티어`라는 달콤한 키워드 때문에 들뜬 마음으로 성급하게 빨리 서버를 받아보고 싶다고 `다음다음 신공`을 하다보면 자칫 간과할수가 있는데 화면을 보면 `결제 알림 생성`이라는 다행스러운 기능이 있다. 별 어려운 설정이 아니니 꼭 설정을 해서 필자같이 기부(?)를 하는 일이 발생하지 않았으면 한다...

{{< image src="/images/aws-freetier-create-and-ssh-access/ec2-4.jpg" caption="" width="80%" >}}

EC2 인스턴스가 생성이 되었다. 인스턴스의 각종 정보를 확인할수가 있는데 public IP, public DNS 까지 제공되는것을 확인할 수 있다. (추후 DNS를 구입하게 되다면 이 IP에 연결을 시켜 도메인으로 해당 서버에 접속을 할수가 있게 된다.)

{{< image src="/images/aws-freetier-create-and-ssh-access/ec2-5.jpg" caption="" width="80%" >}}

## putty 로 발급받은 EC2 인스턴스에 접속을 해보자.
이제 발급받은 EC2 인스턴스에 접속을 해볼 차례이다. 다양한 서버 접속툴이 있지만 필자는 putty를 가장 선호한다. 디자인은 구닥다리처럼 보일지 모르겠지만 개인적으로 직관적인 UI에 가벼운 프로그램이라 생각이 든다. 우선 putty를 [다운](https://www.putty.org/) 받고 `putty.exe`를 실행시킨뒤에 바로 ssh 접속을 하면 너무 간단하게 서버 접속에 성공을 할수 있지만 위에서 받은 키 페어 파일을 다시 private key 로 전환해야 하는데 putty를 다운받으면 동일한 폴더에 `puttygen.exe`라는 파일을 실행시켜주자.
그다음 `pem`파일을 불러와서 마우스를 움직여서 게이지(?)를 다 채우고 `save private key`를 줄러 저장을 하는데 여기서 주의할점은 `ppk`파일명을 `pem`파일명과 동일하게 저장해야 한다는 것이다. (안그러면 서버 접속시 실패가 남... 삽질...)

{{< image src="/images/aws-freetier-create-and-ssh-access/putty-1.jpg" caption="" width="80%" >}}

`putty.exe`를 실행시킨뒤 `Connection` > `SSH` > `Auth` 탭에서 방금 만들어 놓은 `ppk`파일을 불러오고, 다시 `Session`탭에서 host name 을 입력해주고 적당한 이름으로 저장을 눌러준다. 여기서 host name은 위에서 EC2 생성시 `Amazon Linux AMI`를 선택했기 때문에 사용자의 이름은 `ec2-user`가 되고 인스턴스의 정보중 public DNS와 함께 조합하여 다음과 같은 url을 적어준다.
```markdown
ec2-user@{public DNS}
e.g. ec2-user@ec2-###.compute.amazonaws.com
```

{{< image src="/images/aws-freetier-create-and-ssh-access/putty-2.jpg" caption="" width="80%" >}}

이렇게 하고 해당 세션을 더블클릭 또는 하단에 `Open`을 누르게 되면 해당 서버로 접속이 되는것을 확인할 수 있다. 

{{< image src="/images/aws-freetier-create-and-ssh-access/putty-3.jpg" caption="" width="80%" >}}

사실 기술을 배움에 있어 가장 훌륭한 도구는 제공되는 도큐먼트만한게 없다고 생각한다. 그에 필자의 블로그도 좋지만(?) 도큐먼트를 보면서 좀더 자세한 설명을 봐야 한다는 것을 강조하며 이번 포스팅을 마무리 해본다.
※ putty로 AWS EC2 접속하기 : https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/putty.html

## 마치며
다양한 클라우드 서비스들중에 너무나도 신기할정도로 간편하게 클릭 몇번만으로 서버를 띄우고, 서버 접속없이 이또한 클릭 몇번만으로 어플리케이션을 운영할수도 있는 서비스들이 많다. 하지만 필자는 시스템 아키텍쳐를 구성할때엔 버튼 하나로 설치 및 셋팅되는 것보다 직접 설정을 건드려가며 소스로 설치하는 것을 선호한다. 그럼에 AWS의 EC2라는 서비스는 필자의 취향에 너무 알맞는 서비스라며 매력을 느끼고 있는 중이다. 
사이즈 프로젝트를 진행하면서 보다 다양한 AWS 프리티어 활용기를 포스팅 할 수 있을것 같아 벌써부터 설렌다.