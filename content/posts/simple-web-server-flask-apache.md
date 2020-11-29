---
title: 초간단 API서버 만들기 - 1부 (Python + Flask + Apache)
date: 2018-06-29 23:00:00
categories:
  - tech
tags: 
  - apache
  - flask
  - python
  - web server
  - archives-2018
url : /2018/06/29/simple-web-server-flask-apache/
featuredImage: /images/simple-web-server-flask-apache/flask-apache-python.png

---
Static한 HTML이 아닌 로직이 필요한 API서버를 구성한다고 가정해보자. (이제까지 지식으로)처음 머릿속에 떠오르는건 Java를 사용하고 스프링으로 어플리케이션을 만들고 apache에 tomcat을 연동한 다음 ...<!-- more --> 이러한 방법으로 API서버를 구성할수 있겠지만 프로토타이핑 또는 테스트 목적으로 만들기 위해서는 설정하는 시간이 은근 많이 소요된다. (물론 Java Config, Spring Boot 등 간소해졌지만...)
얼마전부터 Python에 대한 매력을 뼈저리게 느끼고 있다보니 Python으로 API서버를 구성할순 없을까 알아봤고 (모바일 게임 듀랑고 서버가 python이라고 하기도 하고...) `Flask`와 `Django`가 있어서 둘다 써본 결과 필자는 `Flask`가 맞겠다고 생각해서 정리를 해볼까 한다.

> '장고'라고도 불리는 Django에는 모든것들이 다 들어가 있어서 사용하기 너무 편리하다. (DB, 어드민 등 ) 하지만 Flask는 내가 사용할 것들만 import해서 사용하는 방식이라 어떤 측면에서는 아무것도 없다 할수 있겠으나 커스터마이징에 용이하다고 볼수 있었기에 Flask를 선택하게 되었다. (Django가 Flask보다 안좋다는 말은 아니니 오해는 하지 마시길...)

글쓰기에 앞서 본 포스팅은 2개의 포스팅에 걸쳐 시리즈(?)형식으로 작성할 예정이다. 1부에서는 Flask가 무엇이고 이를 어떻게 사용하며 Apache와 연동하는 방법을 소개하고, 2부에서는 Nginx와 연동하는 방법을 소개한다.
환경은 다음과 같다.
- CentOS 7.4
- Python 3.6 (기본은 2.7이였으나 추가로 설치)

## Flask ( http://flask.pocoo.org/ )
공식 홈페이지에서도 보면 알수 있듯이 너~무 간단하다. 단지 아래 코드 몇줄만 작성하면 우리가 모든 프로그램 초기 작성시 항상 만나는 "Hello World"를 볼수 있다.
```python
#hello_world.py

from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"
```
위와같이 작성하고 `python hello.py`로 실행해두고 브라우저에서 `http://127.0.0.1:5000` 을 요청하면 반가운 Hello World를 만날수 있다. (너무 간단;;) 자세한 문법은 [도큐먼트](http://flask.pocoo.org/docs)를 참조하면 될듯하고 이 Flask를 잘만 활용한다면 보다 빠르고 간단하게 API서버를 구성할수 있을거라 생각한다.

Hello World를 찍었으면 된거 아닌가 라고 질문할수도 있겠으나 실제 서비스에서 사용하기 위해서는 앞단에 웹서버를 두는게 여러 측면에서 효율적이다. 주로 사용하는 웹서버는 Apache 와 Nginx가 있는데 여기서는 Apache와 연동하는 방법을 정리 해보고자 한다.

## Apache 설치 ( http://archive.apache.org/ )
우선 필자는 `yum` 이나 `apt-get`처럼 패키지 관리자로 설치하는것을 그렇게 좋아하지 않는다. 이유는 커스터마이징을 할 경우 시스템 어느곳에 설치되어있는지를 한눈에 파악하기 어렵고 윈도우경우 `Program Files`처럼 내가 추가로 설치하고 관리하는 프로그램들을 한곳에서 관리하고 싶기에 왠만하면 소스를 직접 컴파일하여 설치하곤 한다. 이번 역시 아파치도 소스로 설치하려고 한다.
현재 아파치는 2.4버전이 Stable버전으로 되어있지만 보다 레퍼런스가 많은 2.2버전으로 설치하기 위해 어렵게 아카이빙된 경로를 통해 다운을 받고 설치를 한다.
```markdown
- 다운을 받고
$ wget http://archive.apache.org/dist/httpd/httpd-2.2.29.tar.gz 
- 압축을 푼 다음
$ tar xvzf httpd-2.2.29.tar.gz
- 해당 폴더로 들어가
$ cd httpd-2.2.29
- 컴파일 후 설치 경로를 정해주고
$ ./configure --prefix=/~~~/apps/apache
- make 파일을 만든다음
$ make
- 설치를 해준다.
$ make install
```
이렇게 되면 /~~~/apps/apache/ 하위에 필요한 파일들이 설치가 되는데 root계정이 아닌 일반계정으로 실행하기 위해서는 /bin하위에 있는 httpd에 대한 실행/소유권한을 변경해줘야 한다. (아니면 그냥 root권한으로 시작/종료. 왜? Apache는 80port를 사용하는데 일반적으로 리눅스에서는 1024 아래 port를 컨트롤 하기 위해서는 root권한이 있어야 사용이 가능, 그게 아니라면 이처럼 별도의 설정이 필요하다.)

```
$ sudo chown root:계정명 httpd
$ sudo chmod +s httpd
```

## mod_wsgi 설치 ( https://code.google.com/archive/p/modwsgi/ )
웹 서버 게이트웨이 인터페이스(WSGI, Web Server Gateway Interface)는 웹서버와 웹 애플리케이션의 인터페이스를 위한 파이선 프레임워크다. 라고 정의되어있다. 즉, 웹서버(Apache)와 위에서 만든 Flask 어플리케이션을 연동해주기 위한 프레임워크이다. 이또한 소스로 설치해보자. (위와 같은 이유로~)
```markdown
- 다운을 받고
$ wget https://github.com/GrahamDumpleton/mod_wsgi/archive/3.5.tar.gz
- 압축을 푼 다음
$ tar -zxvf 3.5.tar.gz
- 폴더에 들어가서
$ cd mod_wsgi-3.5
- 아파치의 빌드툴인 apxs의 경로를 설정해주고
- 필자와 같이 기본 python 버전을 사용하지 않을꺼라면 꼭 python경로를 설정해줘야 한다! (중요)
$ ./configure --with-apxs=/~~~/apps/apache/bin/apxs --with-python=/usr/bin/python3.6
- make 파일을 만들고
$ make
- 설치~
$ make install
```
이렇게 설치를 하면 자동으로 아파치 하위 /modules 폴더안에 mod_wsgi.so 파일이 생긴다. (필자는 이것도 모르고 mod_wsgi.so파일을 다운 받으려고 구글링을 몇일째 했던 기억이 ㅠ)

## Apache httpd.conf 설정
Aapche에 mod_wsgi 모듈이 생겼고, 이를 적용하기 위해서 httd.conf 아파치 기본설정 파일을 수정해야 한다.
```markdown
- 모듈을 사용하겠다고 정의
LoadModule wsgi_module modules/mod_wsgi.so
- mod_wsgi로 실행한 WSGI 파이썬 어플리케이션을 특정 URL( / )로 설정하기 위해 다음과 같이 등록해준다.
WSGIScriptAlias / /~~~/python_app/hello_world.wsgi
- 등록된 파이썬 어플리케이션을 데몬 프로세스로 실행하기위해 다음과 같이 설정해준다.
WSGIDaemonProcess hello_world(어플리케이션 명) user=계정명 group=계정명 threads=5(스레드 개수)

<Directory /~~~/python_app>
  WSGIApplicationGroup %{GLOBAL} # 해당 어플리케이션을 처리하는 프로세스에서 첫번째로 생성된 파이썬 인터프리터를 사용
  Order deny,allow
  Allow from all
</Directory>
```
자세한 내용은 [공식 홈페이지](http://modwsgi.readthedocs.io/en/develop/)를 참고하는걸 추천한다.

## WSGI 파일 작성
Apache 설정에서 등록한 wsgi파일은 다음과 같이 작성해준다. 물론 이 내용도 [공식 홈페이지](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)를 참조하는게 좋다.
```python
import sys
sys.path.insert(0, '/~~~/python_app')
from hello_world import app as application
```
필자는 여기서 한참을 해맸던 부분이, `sys.path.insert`구문의 두번째 인자는 실제 실행될 Falsk 어플리케이션이 있는 경로를 적어줘야 하고, `hello_world`는 Flask 어플리케이션 파일의 확장자를 제외한 이름을 적어주면 된다.

이렇게 한 후 아파치를 시작해주면 앞서 만든 Flask 어플리케이션을 별도로 실행해 주지 않아도 Apache가 알아서 설정한 URL로 요청이 들어올경우 Flask 어플리케이션으로 전달해준다.

## 마치며
사실 이렇게 Apache를 연동하면서 까지 하게 된 계기는, Flask 어플리케이션 구동시 80 port를 받도록 구현하고 root권한을 가진 계정으로 실행하도록 해뒀는데 가끔 종료가 되는 부분을 해결하고자 시작하게 되었다.
이 밖에도 Flask의 다양한 기능들과 Flask만 제외하면 일반 Python 이기 때문에 활용성은 무궁무진할것으로 보인다.

2부로는 또다른 웹서버인 Nginx를 연동하는 방법을 알아보고자 한다.