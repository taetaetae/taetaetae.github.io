---
title: Jenkins 설치 치트키
date: 2018-12-02 04:37:59
categories:
  - tech
tags: 
  - jenkins
  - archives-2018
url : /2018/12/02/jenkins-install/
---
"show me the money", "black sheep wall". 
어렸을적 스타크래프트라는 게임이 나오고서 입에 달고 살았던 `치트키`. 게임이 시작되고 해당 치트키를 입력하면 돈이 들어오거나 맵이 훤하게 보여 컴퓨터를 이기는데 도움을 주곤 했었다. <!-- more -->
개발을 하면서 Jenkins는 나 대신 어떤 업무를 수행하는데 강력한 툴 중에 하나이다. (물론 만능이라는 소리는 아니지만...) 새로운 프로젝트가 시작되거나 개발도중 무언가 자동화를 하고 싶을 경우엔 Jenkins를 찾게 되는데 그럴때마다 설치를 하고 이런저런 설정이 필요하다.
눈치를 챘을수도 있지만 이 포스트는 오로지 `젠킨스 설치하는 방법`을 아주 간단하고 핵심만 정리하고자 한다. 마치 `치트키`처럼. 
나중에 다시 보기위해 + 누군가 해당 포스트를 보고 도움이 되었으면 하는 바람으로.

(물론 이 방법밖에 있는건 아니지만 필자는 아래와 방법을 사용하고 있다.)

---
우선 CentOS 환경에 Java가 설치되어 있는 상황이라 가정한다.


- 적당한 위치에 tomcat 다운 ( https://tomcat.apache.org/download-80.cgi )
  ```
  wget {압축파일 다운경로, 필자는 apache-tomcat-8.5.35 }
  ```
- 압축 해제후 하위 폴더중 webapps로 이동
  ```
  tar -zxvf apache-tomcat-8.5.35.tar.gz
  cd apache-tomcat-8.5.35/webapps
  ```
- Jenkins 다운 ( https://jenkins.io/download/ )
  ```
  wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
  ```
- tomcat 하위폴더중 conf 폴더로 이동
  ```
  cd ../conf
  ```
- server.xml 수정 및 http port 확인
  ```
  vi server.xml

  <Host> 하위에 추가
  <Context path="/jenkins" debug="0" privileged="true" docBase="jenkins.war" />

  port 확인
  <Connector port="8080" protocol="HTTP/1.1"/>
  ```
- 해당 서버의 ip와 위 port에 맞춰 url 입력후 jenkins 설치
  ```
  http://ip:8080/jenkins
  ```