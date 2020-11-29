---
title: 더이상 기다리지 않아도 되는 배치 무중단 배포
date: 2019-10-13 15:46:12
categories:
  - tech
tags: 
  - batch
  - jenkins
  - linux
url : /2019/10/13/batch-nondisruptive-deploy/
featuredImage: /images/batch-nondisruptive-deploy/wait_illustration.jpg
---

[지난 포스팅](https://taetaetae.github.io/2019/09/29/woowabros-spring-batch/), 그러니까 우아한 형제들에서 초대를 받아 Spring batch 에 대한 테크세미나에 다녀 왔다. 그 중 가장 인상깊었던 부분이 바로 `무중단 배포`. 차일피일 미루다 필자가 속한 팀에서도 배포때마다 가장 불편을 느끼고 있었던 부분이었기도 했고<!--more -->, `그런가보다` 하며 개념만 알고 넘어가기엔 무언가 양심에 찔려 직접 무중단 배포를 할 수 있도록 구성을 해보고 테스트까지 해보고자 한다.

## 상황 및 문제점
리눅스 서버에 Jenkins가 설치되어 있고, Spring batch 모듈을 실행시키고 있다. 수동으로 실행을 하거나, Jenkins RestApi를 이용해서 실행을 할 수 있지만 주로 정해진 시간 즉, 스케쥴링에 의해 실행되곤 한다. 스케쥴링의 가장 작은 단위는 1분단위 배치도 있기 때문에 24시간 멈추지 않고 실행되고 있다고 무방하다. 하지만 배치 모듈이 수정되고, 배포를 하기 위해서는 다음과 같은 시나리오로 진행이 된다.
1. Jenkins 설정의 `끄기전 준비` 를 실행하여 더이상 Jenkins에 의해 Spring batch 모듈(이하 Job)이 실행되지 않도록 한다.
2. 새로운 Job은 더이상 실행되지 않지만 이미 실행중이였던 Job 은 강제로 중단을 하거나 Job 이 끝날때까지 기다린다.
3. 실행중인 Job이 없을 경우 이제 배포를 진행한다.
4. 배포가 완료되면 Jenkins 설정의 `끄기전 준비`를 해제한다.

{{< image src="/images/batch-nondisruptive-deploy/wait.jpg" caption="실행중인 Job이 안끝나면 마냥 기다릴텐가? <br>출처 : https://m.post.naver.com/viewer/postView.nhn?volumeNo=14100660&memberNo=2032633" width="80%" >}}

실행되는 Job을 중단하지 못하는 상황 즉, 실행중에 중단하면 트랜잭션이 깨져 무조건 기다려야만 하는 상황이라면 배포 또한 계속 지연될 수 밖에 없는 상황인 것이다. Spring boot에 java config 를 활용하고 딱 `jar` 파일 하나를 실행하는 방식이라면 `jar`파일을 바꿔치기 하는 식으로 고민을 해볼수도 있을것 같다. 하지만 Legacy 코드가 아직 존재하여 일반 Spring 에 xml 로 config 하는 방식으로 운영중이라 `jar`파일 하나만 바꿔치기 하기엔 무리가 있는 상황. 

은총알처럼 어디에서나 사용이 가능한 만병통치약 같은 방법은 없다. 언제나 그랬듯 현재 시스템(xml config 방식)에 가장 최적화된 방법, 그리고 java config 방식에서도 사용이 가능할것 같은 방법을 생각해 보았다.

## 무중단 배포를 가능케 하는 3가지 핵심
**1. 배포를 매번 새로운 경로에 배포한다.**
각 회사마다, 그리고 서비스마다 정말 다양한 배포 시스템이 있다. 그들의 공통점은 원격서버의 `특정 경로`에 빌드된 파일들을 밀어 넣어준다는 것. 시나리오는 다음과 같다.
1. 배포할때마다 별도의 디렉토리를 생성한뒤 심볼릭 링크를 연결해준다.
2. 배포는 `1`에서 연결한 심볼릭 링크에 배포되도록 설정, 결국 매번 만들어지는 디렉토리에 배포가 되게 된다.

여기서 중요한점은 "배포할 때마다 새로운 디렉토리에 배포가 된다" 와 배포시에는 항상 심볼릭 링크에만 배포를 하면 되기 때문에 "배포시스템이 새로 만들어지는 디렉토리의 경로를 몰라도 무방하다"는 점이다.
```bash
#!/bin/sh
cd /~~~/deploy/

# 임시 디렉토리
DIRECTORY_NAME=batch_$(/bin/date +%Y%m%d%H%M%S)
mkdir $DIRECTORY_NAME
```
위 쉘 스크립트를 실행하면 batch_20191012205218 와 같은 디렉토리가 생성이 된다. 심볼릭 링크 관련해서는 바로 아래 이어서 설명하겠다.

**2. 심볼릭 링크의 원래 링크를 즉시 변경**
보통 심볼릭 링크 (즉, 바로가기) 의 경로를 변경하기 위해서는 아래처럼 지웠다가 삭제하는 식으로 했었는데
```bash
$ mkdir directory_a
$ mkdir directory_b
$ ln -s directory_a asdf
$ ll
asdf -> directory_a
directory_a
directory_b

# directory_a 에서 directory_b 로 바꾸는 경우 (심볼릭 링크 자체를 삭제하고 다시 심볼릭 링크 생성)
$ rm asdf
$ ln -s directory_b asdf
$ ll
asdf -> directory_b
directory_a
directory_b
```

이렇게 되면 삭제하고 \~ 다시 만들어지는 타이밍에 배포가 되거나 실행이 되는 즉, 해당 경로에 엑세스 하는 경우 이전의 경로를 바라본다거나 의도했던 방식으로 실행이 되지 않는 상황이 발생한다. (찰나의 타이밍 이지만 필자는 이러한 문제로 이전의 경로를 바라보는 문제가 발생했었다.) 그래서 ln 의 옵션중인 `-Tfs`옵션으로 즉시 변경을 해주도록 하자. ([ln man 참고](https://linux.die.net/man/1/ln))
```bash
# 만든 임시 디렉토리로 배포될수 있도록 설정한다.
ln -Tfs /deploy/$DIRECTORY_NAME /~~~/deploy/batch
```

**3. 심볼릭 링크가 가리키는 원래 링크에서 실행**
리눅스 명령어 중에 [readlink](https://linux.die.net/man/1/readlink)라는게 있다. 실제 링크를 얻어오는 명령어 인데 이를 활용하여 위에서 설정해둔 심볼릭 링크의 실제 링크(최신 배포된 경로)를 가져오고 그곳에서 Spring batch 모듈을 실행하는 식으로 구성을 해보자.
```bash
#!/bin/bash

BASEDIR=`readlink -f $(dirname $0)` # -f 옵션 : 전체경로
cd $BASEDIR # 이후 Spring batch jar 실행
```
이렇게 되면 Job이 실행중이라도 기존에 실행중인 Job은 기존 모듈을 바라보고 실행이 되고, 도중에 새로 배포가 되어도 기존 실행되는 Job에는 영향을 주지 않으며(심볼릭 링크에 연결되었던 과거 배포 경로에서 실행되고 있기 때문) 새롭게 배포된 후 Job이 실행될때도 배포된 경로의 > 심볼릭 링크의 > 실제 링크 즉, 새롭게 배포된 경로에서 실행되기 때문에 무중단 배포가 가능하게 된다.

## 전체 흐름
핵심만 설명하다보니 전체적으로 어떻게 돌아가는지 이해를 못하셨을 분들을 위해 전체 흐름에 대해 설명을 해보고자 한다.
**1. 배포 전**
  임시 디렉토리를 생성하고, 그곳에 배포가 될 수 있도록 심볼릭 링크를 연결해준다.
  ```bash
  #!/bin/sh
  cd /~~~/deploy/

  # 임시 디렉토리
  DIRECTORY_NAME=batch_$(/bin/date +%Y%m%d%H%M%S)
  mkdir $DIRECTORY_NAME

  # 만든 임시 디렉토리로 배포될수 있도록 설정한다.
  ln -Tfs /~~~/deploy/$DIRECTORY_NAME /~~~/deploy/batch
  ```

**2. 배포**
  배포 시스템에 의해 `/~~~/deploy/batch` 로 배포되도록 한다.

**3. 배포 후**
  이후 배포 실행은 새롭게 배포된 경로에서 실행되도록 심볼릭 링크를 수정해준다.
  ```bash
  #!/bin/sh
  cd /~~~/deploy/

  # 배포된 경로를 실행할 경로로 변경해준다.
  REAL_DIRECTORY_PATH=$(readlink -f /~~~/deploy/batch)
  ln -Tfs $REAL_DIRECTORY_PATH /~~~/deploy/batch

  # 예전에 배포된 폴더들을 삭제해준다. (최근 몇개까지만 지울것인가는 상황에 따라)
  ```
  
**4. 배치 실행**
  ```bash
  #!/bin/bash

  BASEDIR=`readlink -f $(dirname $0)` # -f 옵션 : 전체경로
  cd $BASEDIR # 이후 배치 실행 ( e.g. batch.jar xxxJob )
  ```

### # 마치며
`jar`파일이 실행되고 `JVM`에 올라가게 되면 `jar`파일을 삭제한다거나 위치를 이동시켜도 에러가 나거나 하지는 않지만 코드 내에서 상대경로같은 설정들이 있기 때문에 폴더 전체를 심볼릭 링크로 연결하고 그 안에서 실행되도록 수정하였다. 앞서 이야기 했지만 이러한 설계는 어디까지나 필자가 운영하고 있는 상황에 맞춘것이기 때문에 이를 어떻게 잘 `활용`하는가가 이번 포스팅에 주요 핵심이 될 수 있을것 같다.
항상 배포 할때마다 `예전에 그렇게 해왔기 때문에` 라는 핑계로 Job이 돌고있으면 기다렸다가 배포해야만 했던 필자 자신이 부끄러워진다. 시도조차 안해보고 그런가보다 하고 적응만 하려 하거나, 불편하지만 안불편한척 하는 그런 태도를 버려야 하지 않을까 하는 반성을 해보는 시간이 되었다.