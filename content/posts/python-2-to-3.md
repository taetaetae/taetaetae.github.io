---
title: 파이썬 버전 업그레이드 (2.6 > 3.6)
date: 2018-01-08 13:44:50
categories:
  - tech
tags:
  - python
  - archives-2018
url : /2018/01/08/python-2-to-3/

---
파이썬 2.x 에서는 depreate 된 모듈도 많고 3.x에서만 지원되는 버전들이 많아지면서 실컷 개발을 해도 파이썬 버전때문에 다시 짜야하는 상황이 생긴다. 파이썬 버전업을 하고싶어 구글링을 해보면 이렇다할 정리된 문서가 잘 안나온다. (영어로된 포스트는 많이 있긴 하나, 필자의 환경과는 맞지 않는 ...)<!-- more --> 그래서 이것저것 삽질을 한 결과 파이썬 버전을 올릴수 있었고, 이를 포스팅 해보고자 한다.
> 그러보고니 2018년 첫 포스팅이네... 올해는 정말 적어도 한달에 1~2개는 올릴수 있는 내가 되기를...

## 환경
- CentOS 6.9
- 기본으로 python 2.6 이 설치되어 있는것을 확인할수 있다. (환경마다 다를수 있음.)
```
$ python -V
Python 2.6
```

## 설치순서
- 필요한 유틸리티를 설치한다.
```
$ sudo yum update
$ sudo yum install yum-utils
$ sudo yum groupinstall development
```
- yum 저장소에서는 최신 파이썬 릴리즈를 제공하지 않으므로 RPM 패키를 제공하는 IUM 이라는 추가 저장소를 설치
```
$ sudo yum install https://centos6.iuscommunity.org/ius-release.rpm
```
- 파이썬 3.6 버전을 설치
```
$ sudo yum install python36u
```
- pip 등 패키지 관련 모듈도 함께 설치
```
sudo yum install python36u-pip
sudo yum install python36u-devel
```
- 여기까지 하면 기존 파이썬 `2.6`과 새로 설치된 파이썬 `3.6` 이 설치되어있다.
```
$ ll /usr/bin/python*
-rwxr-xr-x 1 root root 9997450 Jan  2 16:02 python
lrwxrwxrwx 1 root root       6 Jan  1 06:02 python2 -> python
-rwxr-xr-x 1 root root    9032 Aug 19  2016 python2.6
-rwxr-xr-x 1 root root    1418 Aug 19  2016 python2.6-config
-rwxr-xr-x 2 root root    6808 Oct 12 08:19 python3.6
lrwxrwxrwx 1 root root      26 Jan  2 20:48 python3.6-config -> /usr/bin/python3.6m-config
-rwxr-xr-x 2 root root    6808 Oct 12 08:19 python3.6m
-rwxr-xr-x 1 root root     173 Oct 12 08:19 python3.6m-config
-rwxr-xr-x 1 root root    3339 Oct 12 08:16 python3.6m-x86_64-config
lrwxrwxrwx 1 root root      16 Apr 25  2017 python-config -> python2.6-config
```
- 환경변수를 설정해준다.
```
$ sudo mv python python_backup
$ sudo ln -s python3.6 python
```
- 확인
```
$ python -V
Python 3.6.3
```

## pip 를 이용한 모듈 설치
- pip란 Python Package Index 의 약자로 공식홈페이지는 다음과 같다. ( https://pypi.python.org/pypi/pip )
- 설치할 모듈을 다음과 같이 설치해주면 된다. ex : requests 모듈인 경우
```
$ sudo python3.6 -m pip install requests
```