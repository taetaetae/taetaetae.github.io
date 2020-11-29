---
title: Jupyter 설치하고 원격접속까지 (for 파.알.못)
date: 2020-02-09 20:06:15
categories:
  - tech
tags: 
  - python
  - jupyter
  - archives-2020
url : /2020/02/09/jupyter-install/
featuredImagePreview: /images/jupyter-install/jupyter_logo.jpg
images :
  - /images/jupyter-install/jupyter_logo.jpg
---

파이썬이라는 언어는 다른 프로그래밍 언어들에 비해 쉽고 직관적이라 그런지 프로그래밍을 처음 시작하는 사람들에게 더욱이 주목을 받고 있는것 같다. 정말 다양한 모듈들이 많아 여러분야에서 활용되고 있고 <!--more -->특히 언제부터인가 핫! 해진 분야(?)라 해도 과언이 아닐정도인 "머신러닝" 분야에서도 다양하게 사용되고 있는것 같다. 

마침 필자가 속해 있는 팀 내에 머신러닝 스터디가 시작이 되었고, 그에 파이썬을 이용하여 스터디를 해야하는 상황. 하지만 스터디를 하는 팀원 절반 이상이 파이썬을 이용한 개발 경험이 없었고, 서로 배운것을 공유를 하면서 스터디를 하면 더 좋겠다는 생각이 들때 즈음. 언제 어디선가 봤던것이 머릿속을 스쳐 지나간다. 그건 바로 Jupyter(이하 주피터).

{{< image src="/images/jupyter-install/jupyter_logo.jpg" caption="출처 : https://jupyter.org/" width="80%" >}}

[주피터](https://jupyter.org/)는 수십 개의 프로그래밍 언어에서 대화 형 컴퓨팅을위한 오픈 소스 소프트웨어, 오픈 표준 및 서비스를 개발하기 위한 툴이라고 한다. 이 포스트를 작성하기 전까지만 해도 "주피터 == 파이썬 웹 개발툴" 이라고만 알고있었는데 좀더 찾아보니 [다양한 언어를 지원](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels)하는것 같다. 

그럼 이러한 주피터를 특정 서버에 설치하고 로컬에 파이썬을 설치하지 않아도 원격으로 파이썬 코딩을 해보면 좀더 스터디에 도움이 되지 않을까 하는 마음이 들었다. 또한 학교에서 운동장에 잔디를 깔아서 맘껏 뛰놀수 있게 하는 느낌으로 팀원들을 위해 설치를 해두고 원격으로 접속할 수 있게 해두면 모두가 편하고 쉽게 파이썬에 대해 경험을 해볼 수 있지 않을까 하는 마음으로 주피터를 설치를 해 보고자 한다.

본 포스팅의 목표는 다음과 같다.
- 환경 : CentOS 7.4 64Bit, python 2.7 (기본)
- 목표
  - anaconda 를 활용하여 시스템 기본 파이썬을 건드리지 않는 가상환경을 구축한다.
  - 주피터를 설치하고 원격으로 접속할 수 있도록 설정한다.

여기까지 보면 필자가 엄청나게 파이썬에 대해 잘 아는것처럼 보일수도 있어 미리 말하지만 필자는 찐 자바 개발자이면서 파이썬 개발 수준은 기본적인 스크립트를 작성하는 정도이다. 그러니 이 포스트를 읽고 있는 필자같은 파알못(?) 분들도 충분히 설치가 가능하다. (최대한 따라할수 있을 정도의 치트키 수준으로 작성 하고자 한다.)

## 아나콘다 설치 (덤으로 설치되는 주피터)
우선 아나콘다를 설치하자. 아나콘다는 Anaconda(이전: Continuum Analytics)라는 곳에서 만든 파이썬 배포판으로, 수백 개의 파이썬 패키지를 포함하고 있다고 한다. 즉, 아나콘다를 설치하고 만들어진 가상환경에서 파이썬 개발을 하면 다양한 모듈이 이미 설치되어 있기 때문에 편리하다는 이야기. 

{{< image src="/images/jupyter-install/ananconda.jpg" caption="출처 : https://www.anaconda.com/" width="40%" >}}

더불어 시스템에 기본으로 설치되어 있는 파이썬을 건드리면 여러 복잡한 문제가 발생할 수 있기에. 아나콘다를 활용하여 파이썬 3을 사용하는 가상환경을 만들어 보자.
설치는 아주 간단하다. 아나콘다 설치파일을 다운받고 이를 실행하면 끝.
(user 레벨이 root 면 sudo 명령어를 생략해도 된다.)
```shell
$ wget https://repo.anaconda.com/archive/Anaconda3-2019.10-Linux-x86_64.sh
$ sudo bash Anaconda3-2019.10-Linux-x86_64.sh

Welcome to Anaconda3 2019.10

In order to continue the installation process, please review the license
agreement.
Please, press ENTER to continue
>>>
===================================
Anaconda End User License Agreement
===================================

Copyright 2015, Anaconda, Inc.
~~~ 중략 ~~~
Do you accept the license terms? [yes|no]
[no] >>> yes # yes!!

Anaconda3 will now be installed into this location:
/root/anaconda3

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below

[/root/anaconda3] >>> /home/anaconda3 # 설치될 경로를 설정해주고 기본 설정값에 설치하려면 그냥 엔터

~~~뭐가 엄청 설치된다. 물 한잔 먹고 오자.~~~

installation finished.
Do you wish the installer to initialize Anaconda3
by running conda init? [yes|no]
[no] >>> yes # yes!!
```
이렇게 되면 설치는 끝. 환경변수를 설정해서 기본 파이썬 환경을 아나콘다에 의해 설정되도록 맞춰주자.
```shell
sudo vi .bashrc

__conda_setup="$('/home/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/anaconda3/etc/profile.d/conda.sh" ]; then
        . "/home/anaconda3/etc/profile.d/conda.sh"
    else
        export PATH="/home/anaconda3/bin:$PATH"
    fi
fi
unset __conda_setup

source .bashrc
```
그러면 다음과 같이 프롬프트가 변경된것을 확인할 수 있다.
```markdown
환경변수 변경 전
[user@server ~]$ 

환경변수 변경 후
(base) [user@server ~]$
```
참, 아나콘를 설치하면 주피터가 같이 설치가 된다. 좀전에 설치된 내용을 보면 주피터가 설치된것을 확인할 수 있다.
```markdown
  ...
  jinja2             pkgs/main/noarch::jinja2-2.10.3-py_0
  joblib             pkgs/main/linux-64::joblib-0.13.2-py37_0
  jpeg               pkgs/main/linux-64::jpeg-9b-h024ee3a_2
  json5              pkgs/main/noarch::json5-0.8.5-py_0
  jsonschema         pkgs/main/linux-64::jsonschema-3.0.2-py37_0
  jupyter            pkgs/main/linux-64::jupyter-1.0.0-py37_7
  jupyter_client     pkgs/main/linux-64::jupyter_client-5.3.3-py37_1
  jupyter_console    pkgs/main/linux-64::jupyter_console-6.0.0-py37_0
  jupyter_core       pkgs/main/noarch::jupyter_core-4.5.0-py_0
  jupyterlab         pkgs/main/noarch::jupyterlab-1.1.4-pyhf63ae98_0
  jupyterlab_server  pkgs/main/noarch::jupyterlab_server-1.0.6-py_0
  keyring            pkgs/main/linux-64::keyring-18.0.0-py37_0
  kiwisolver         pkgs/main/linux-64::kiwisolver-1.1.0-py37he6710b0_0
  ...
```
## 주피터 원격 설정
다음으로 주피터 환경설정을 만들어 주자. 기본 설정이 아닌 별도 설정을 만드는 이유는 원격으로 띄울때 입맛에 맞도록 띄우기 위함이다. 설정파일을 생성하고 수정을 해주자.
```shell
$ sudo jupyter notebook --generate-config
Writing default config to: /root/.jupyter/jupyter_notebook_config.py
$ sudo vi /root/.jupyter/jupyter_notebook_config.py
	c.NotebookApp.notebook_dir = '/home/data' # 주피터에서 만든 결과물들이 저장되는 경로
	c.NotebookApp.ip = '0.0.0.0' # 외부에서 접속하기 위한 설정
	c.NotebookApp.port = 80 # 서버 ip (혹은 설정한 도메인) 으로 바로 접속하기 위해
```
이렇게 하면 설정 끝! 이제 아래 명령어로 드디어 주피터를 실행시켜 보자.
```shell
$ sudo jupyter notebook --config=/root/.jupyter/jupyter_notebook_config.py --allow-root --no-browser

[I 21:30:18.278 NotebookApp] JupyterLab extension loaded from /home/anaconda3/lib/python3.7/site-packages/jupyterlab
[I 21:30:18.278 NotebookApp] JupyterLab application directory is /home/anaconda3/share/jupyter/lab
[I 21:30:18.279 NotebookApp] Serving notebooks from local directory: /home/data
[I 21:30:18.280 NotebookApp] The Jupyter Notebook is running at:
[I 21:30:18.280 NotebookApp] http://server:80/?token=82db9~~~~~~be6b9d5
[I 21:30:18.280 NotebookApp]  or http://127.0.0.1:80/?token=82db9~~~~~~be6b9d5
[I 21:30:18.280 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 21:30:18.283 NotebookApp]

    To access the notebook, open this file in a browser:
        file:///root/.local/share/jupyter/runtime/nbserver-9366-open.html
    Or copy and paste one of these URLs:
        http://server:80/?token=82db9~~~~~~be6b9d5
     or http://127.0.0.1:80/?token=82db9~~~~~~be6b9d5

```
서버 ip로 접속을 해보면 로그인 페이지가 나오고 토큰을 입력하라고 한다. 그럼 당황하지 말고 서버에 나온 토큰을 입력하고 로그인을 하면 아래처럼 브라우저(?)같은 화면을 맞이할 수 있다.

{{< image src="/images/jupyter-install/jupyter_install.jpg" caption="기나긴 여정끝에 맞이하는 로그인 페이지!" width="80%" >}}

그런데 기다란 토큰을 입력하는 것보단 외우기 쉬운 패스워드를 입력하는게 더 편할테니, 하단 영역에서 패스워드를 설정해두면 다음번엔 (토큰보다는 외우기 쉬운) 패스워드를 입력하고 로그인이 가능하다.

> 여기서 80 port 로 띄우기 위해서는 root 권한이 있어야 한다. 
처음엔 80 port 가 아닌 다른 port 로 띄우고 앞단에 아파치를 둬서 프록시 태우려 했으나 프록시가 되는 과정에서 뭔가 정보전달이 잘 안되는 느낌이었다. 그렇기에 앞서 말한대로 root 권한으로 띄워야 한다.

모든 foreground 프로세스들은 그 프로세스를 띄운 세션이 종료되면 해당 프로세스 또한 종료가 되어버린다. 리눅스의 "&" 명령어를 이용하여 backgrund로 띄워주자. 
```shell
sudo jupyter notebook --config=/root/.jupyter/jupyter_notebook_config.py --allow-root --no-browser &
```

## 주피터 활용팁
IDE처럼 엄청난 기능을 제공하진 않지만 외부에서 누구나 간단히 접속하여 파이썬 코딩을 할 수 있는 상태가 되었다. 폴더 기능도 제공하니 각자의 환경(폴더)을 만들어서 코드를 작성할 수 있다.
여기서는 "notebook" 이라는 개념으로 불리우는데 파이썬 파일을 만들고 언제나 그랬듯 hello world 를 출력한뒤 Shift+Enter 을 누르면 바로 결과물이 나오는 것을 확인할 수 있다.

{{< image src="/images/jupyter-install/download_notebook.jpg" caption="위 메뉴에서 다양한 포맷으로 다운을 받을 수 있다." width="40%" >}}

자동완성은 "Tab", 어느정도 시간이 지나면 자동으로 저장이 되고, File → Download as → Notebook(.ipynb) 로 추출을 한뒤 gist 에도 업로드가 가능하다. 아주 이쁘게. (gist 에 올려진 주피터 노트북 결과물 : [링크](https://gist.github.com/search?l=Jupyter+Notebook&q=ipynb))

더 자세한 내용은 주피터 도큐먼트를 참고하자. [링크](https://jupyter.org/documentation)

## 마치며
이로써 팀원들이 파이썬을 가지고 놀아볼 수 있는(?) 운동장이 만들어 졌다. 우리는 개발자니까. 환경이 없으면 직접 만들면 되니까. 개발자로 살아가면서 이런 부분이 참 매력적인 것 같다.