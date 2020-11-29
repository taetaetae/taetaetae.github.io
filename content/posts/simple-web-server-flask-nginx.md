---
title: 초간단 API서버 만들기 - 2부 (Python + Flask + Nginx)
date: 2018-07-01 02:00:00
categories:
  - tech
tags: 
  - apache
  - flask
  - nginx
  - web server
  - archives-2018
url : /2018/07/01/simple-web-server-flask-nginx/
featuredImage: /images/simple-web-server-flask-nginx/flask-apache-nginx.png
images :
  - /images/simple-web-server-flask-nginx/flask-apache-nginx.png
---
[지난포스팅](https://taetaetae.github.io/2018/07/01/simple-web-server-flask-apache/)에 이어 이번엔 Flask와 Nginx를 연동하는 방법을 정리해보고자 한다. Apache로 연동했는데 왜 또 Nginx로 연동하는걸 정리하지(?)하며 의문이 들수 있는데 다른 포스팅을 봐도 <!-- more --> Apache + Flask 조합보다 Nginx + Flask 조합이 더 많고 지난 포스팅에서도 알수있었듯이 ([Apache VS Nginx](https://taetaetae.github.io/2018/06/27/apache-vs-nginx/)) 둘중 어느것이 좋다고 할수도 없고 각 상황에서 연동하는 방법을 알고 있다면 이 또한 나만의 무기가 될것같아 Nginx를 연동하는 방법을 정리해보려 한다.

1부에서 왜 Flask인가, Flask의 장점에 대해 정리를 했으니 이번 포스팅에서는 별도로 작성하진 않는다.

## Nginx 설치 ( https://nginx.org/en/ )
역시 소스설치를 한다. 
```markdown
- 다운을 받고
$ https://nginx.org/download/nginx-1.14.0.tar.gz
- 압축을 푼 다음
$ tar -zxvf nginx-1.14.0.tar.gz
- 폴더로 이동해서 
$ cd nginx-1.14.0
- 설치할 디렉토리를 설정하고
$ ./configure --prefix=/~~~/apps/nginx
- make 파일을 만들고
$ make
- 설치를 진행한다.
$ make install
```
이렇게 하면 일단 Nginx는 설치가 되었다.

## uWSGI 설치 ( https://uwsgi-docs.readthedocs.io/ )
앞서 Apache와 연동할때는 별도의 모듈을 Apache에게 등록하는 형태였다면 Nginx는 WSGI프로토콜을 활용하는 WSGI 어플리케이션을 실행하는 어플리케이션 서버를 활용해야 한다. 
```markdown
- 다운을 받고
$ wget https://projects.unbit.it/downloads/uwsgi-latest.tar.gz
- 압축을 풀고
$ tar zxvf uwsgi-latest.tar.gz
- 폴더로 이동하여
$ cd uwsgi-2.0.17
- make 명령어를 호출하면 'uwsgi'이라는 실행파일이 생성된다.
$ make
```

## Nginx 설정
Apache와 비슷하게 uWSGI 관련 설정을 해준다.
```markdown
server {
  listen       80;
  server_name  localhost;

  location / { # ( / ) 경로로 들어올 경우
    include uwsgi_params; # GET/POST 등 기본적으로 필요한 환경변수를 include 해준다.
    uwsgi_pass 127.0.0.1:3031; # 요청을 IP:PORT로 전달한다.
  }
}
```
별도의 모듈을 사용하지 않기때문에 전달해주는 (proxy느낌) 설정을 해준다.

## uWSGI 실행 및 Nginx 재시작
앞서 설치한 `uwsgi`를 아래처럼 IP:port 를 명시적으로 적어주고 (위에서 전달받은 IP:PORT와 동일하게) Apache 연동시 활용했던 wsgi파일을 이번에도 동일하게 사용하도록 해서 실행한다.
```markdown
$ ./uwsgi -s 127.0.0.1:3031 --wsgi-file /~~~/python_app/hello_world.wsgi
```
이렇게 하면 background로 실행되는게 아닌 foreground로 실행되기 때문에 `&`을 사용한다던지 해서 background로 실행되도록 해준다. 그후 Nginx를 재시작 해주면 원하는 그토록 원했던 `Hello World!`를 만날수가 있게 된다.

Apache연동과 조금 다른점은 모듈을 사용하지않고 별도의 전달 어플리케이션(?)이 필요하다는점이다. 간단히 Apache처럼 모듈만 넣으면 되는게 아니라서 불편할수도 있을것 같지만 한편으로는 관리할수있는 포인트가 더 늘어난 셈이라 어떤 측면에서는 활용할수 있는 방법이 하나 늘어난것으로 볼수도 있다.

## 마치며
막상 정리하고 나면 아무것도 아닌데 알기 위해서 몸부림을 쳐가며 책이며 구글링을 하는 과정을 통해 점점 성장을 하는것 같다. (성장통이라고나 할까) 이렇게 단순히 Flask를 할수있다 가 아닌 웹서버를 연동할수있다. 그것도 Apache와 Nginx 두개나. 이것도 언젠간 나만의 무기가 되지 않을까?