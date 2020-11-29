---
title: 아파치 로드밸런싱으로 여러 WAS 운영하기
date: 2019-08-04 19:50:43
categories:
  - tech
tags: 
  - apache
  - tomcat
  - Load Balance
url : /2019/08/04/apache-load-balancing/
featuredImage: /images/apache-load-balancing/lb_logo.jpg

---
웹서버 하나만 사용하거나 WAS 하나만을 사용하며 웹서비스를 운영하는 경우는 극히 드물다. 웹서버의 장점과 WAS의 장점 그 두마리의 토끼를 다 잡기 위해 보통 앞단에 웹서버를 두고 그 뒤에 WAS를 두며 서비스를 운영하곤 한다. 헌데 운영하는 서비스가 인기가 많아져(?) 사용량이 많아지다면 그만큼 응답이 느려 (TPS 등) 서버를 늘려야 하는 상황이 생긴다고 가정해보자.<!--more --> (물론 서버를 늘리는 것보다 캐시를 적용하거나 로직을 바꿔보는 노력이 선행되야 하겠지만...) 당연히 서버부터 구매하며 "Scale Out"을 하려고 할것이다. 만약 원래 운영하던 서버가 너무 좋아서 CPU나 메모리 사용률이 거의 바닥이여도 서버를 구매해야 할까?
서버를 구매하게되면 결국 두개 이상의 서버가 운영될텐데 그 서버들을 앞에서 묶어주며 트래픽을 분산시켜주는 무언가가 필요하다. 그러한 기술을 바로 `로드밸런싱` 이라고 한다. 통상 L4 스위치를 활용하여 요청을 여러 서버들로 분산시키며 산술적으로는 서버 대수만큼 성능이 좋아지는 효과를 볼 수 있다.
하지만 앞서 말했듯 서버의 자원 사용률이 바닥일 정도로 거의 사용을 안할경우 서버를 구매하는건 너무나 비효율적이다. 이번 포스팅에서는 서버를 늘리지 않으면서 웹서버 중 아파치를 활용하여 여러 WAS를 운영하는 방법에 대해 알아보고자 한다. 서버 늘려야 하는 상황에서 사용해 볼 수 있는 나만의 좋은 무기(?)가 생긴게 아닐까 생각이 든다.

아파치는 [EOL](https://httpd.apache.org/#apache-httpd-22-end-of-life-2018-01-01)이 되었기 때문에 2.4버전으로 설치하고, WAS는 편의상 톰켓 최신버전으로 설치해서 동일한 서버에 아파치 한대와 톰켓 3대를 연동하는것을 목적으로 한다. 로드밸런싱이 어떤식으로 이루어 지고 하위에 연결된 톰켓을 컨트롤 하는 방법 또한 알아볼 예정이다.

> 서버 환경 및 설치하게 될 각 버전은 다음과 같다.
서버 : CentOS 7.4 64Bit
apache : httpd-2.4.39
tomcat : apache-tomcat-8.5.43
tomcat-connectors(mod_jk) : 1.2.46

## Apache 와 Tomcat 설치
필자의 포스팅에서 종종 나오는 부분이기도 하고, 구글링 해보면 바로 설치 방법을 쉽게 찾을 수 있겠지만 그렇다고 언급을 안하고 넘어가기엔 너무 불친절하니... 치트키처럼(?) 빠르게 정리해보자.
- Apache
```markdown
$ wget http://apache.tt.co.kr//httpd/httpd-2.4.39.tar.gz
$ tar -zxvf httpd-2.4.39.tar.gz
$ ./configure --prefix=/home/~~~/apache
$ make && make install
$ cd /home/~~~/apache/bin
$ sudo chown root:계정명 httpd
$ sudo chmod +s httpd
$ vi /home/~~~/apache/conf/httpd.conf
User 계정명
Grop 계정명
$ /home/~~~/apache/bin/apachectl start ← 실행
```
이렇게 설치를 한뒤 실행을 시키고 서버의 ip를 접속해보면 아래와 같은 화면을 볼 수 있다.

- Tomcat
```markdown
$ wget http://mirror.apache-kr.org/tomcat/tomcat-8/v8.5.43/bin/apache-tomcat-8.5.43.tar.gz
$ tar -zxvf apache-tomcat-8.5.43.tar.gz
$ /home/apache-tomcat-8.5.43/bin/start.sh ← 실행
```
톰켓의 기본 http 포트인 8080으로 접속을 해보면 귀여운 고양이가 있는 톰켓 기본화면을 볼 수 있다.

## 아파치와 톰켓 연동하기
아파치와 톰켓의 연동은 `mod_jk` 와 `mod_proxy` 등 다양한 모듈로 연동을 할 수 있는데 이번 포스팅에서는 `mod_jk` 를 활용하는 방법에 대해 알아보고자 한다. 우선 mod_jk 를 설치하자.
> 간단히 mod_jk 는 컴파일, 설정 등 복잡하지만 톰켓 전용 바이너리 프로토콜인 AJP를 사용하기 때문에 높은 성능을 기대할수가 있다. mod_proxy 는 반면 기본으로 아파치에 탑재되어있는 모듈이기 때문에 별도의 모듈 설치가 필요 없고 설정도 간단하다는 장점이 있다. 각 연동방식의 장단점이 있기 때문에 본인이 운영하는 서버 상황에 맞추어 적용 할 필요가 있다.

- mod_jk 설치
```markdown
$ wget http://apache.tt.co.kr/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.46-src.tar.gz
$ tar -zxvf tomcat-connectors-1.2.46-src.tar.gz
$ cd tomcat-connectors-1.2.46-src/native
$ ./configure --with-apxs=/home/~~~/apache/bin/apxs
$ make && make install
$ /home/~~~/apache/modules 하위에 mod_jk.so가 생김
```

mod_jk 를 활용하면 AJP라는 통신으로 아파치와 톰켓이 연동되는데 톰켓의 기본 AJP 포트는 8009번임을 알고 다음처럼 설정을 해주자.

- apache/conf/workers.properties
```markdown
worker.list=tomcat1
worker.tomcat1.port=8009
worker.tomcat1.host=localhost
worker.tomcat1.type=ajp13
worker.tomcat1.lbfactor=1
```
- apache/conf/httpd.conf
```markdown
LoadModule jk_module modules/mod_jk.so
<IfModule jk_module>
    JkWorkersFile    conf/workers.properties
    JkLogFile        logs/mod_jk.log
    JkLogLevel       info
    JkMount /* 	tomcat1
</IfModule>
```
이렇게 하고서 아파치와 톰켓을 재시작 후에 서버의 ip로 접속해보면 (별도의 port 없이) 톰켓 설정페이지로 랜딩이 되는것을 확인할 수 있다.

## 로드밸런싱을 위한 작업
여기까지는 본 포스팅을 작성하기 위한 밑거름이라고 말할 수 있다. 이제 실제로 로드밸런싱을 해볼 차례.
앞서 톰켓 하나만 설치했는데 편의상 톰켓 3개를 설치해두자. (하나를 설치하고 cp -r 명령어를 활용하는게 빠르다.) 그 다음 각 톰켓의 모든 포트를 셋다 다르게 설정해야 하는데 겹치지 않도록 설정해 두고 (필자는 앞자리를 1,2,3 이런식으로 다르게 설정하였다.) 워커(workers.properties)를 아래처럼 설정해주자.

- apache/conf/workers.properties
```markdown
worker.list=load_balancer

worker.load_balancer.type=lb
worker.load_balancer.balance_workers=tomcat1,tomcat2,tomcat3

worker.tomcat1.port=18009
worker.tomcat1.host=localhost
worker.tomcat1.type=ajp13
worker.tomcat1.lbfactor=1

worker.tomcat2.port=28009
worker.tomcat2.host=localhost
worker.tomcat2.type=ajp13
worker.tomcat2.lbfactor=1

worker.tomcat3.port=38009
worker.tomcat3.host=localhost
worker.tomcat3.type=ajp13
worker.tomcat3.lbfactor=1
```
이렇게 설정을 한 뒤 앞서 설정한 httpd.conf 에 JkMount 부분도 아래처럼 변경해주자.

- apache/conf/httpd.conf
```
JkMount /* load_balancer
```

위 설정을 다시한번 살펴보자면, `/*`으로 들어오는 요청을 `load_balancer`라는 워커로 넘기는데 워커 설정에서는 로드밸런싱이 설정되어 있기 때문에 tomcat1, tomcat2, tomcat3 골고루 요청을 분산해준다는 의미이다.
tomcat 하위 logs 폴더에 보면 아래 기본 설정에 의해 엑세스 로그가 로깅이 되는데 
```
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
```
실제로 테스트를 해보면 다음처럼 9번의 요청을 3대의 톰켓에 골고루 요청된 것을 확인할 수 있다.

{{< image src="/images/apache-load-balancing/apache_lb_test.jpg" caption="" width="80%" >}}

## 로드밸런싱을 컨트롤 하기 (jkmanager)
위에서 알아본 mod_jk 를 활용한 로드밸런싱을 별도의 서버 재시작 없이 `컨트롤`이 가능하다고 한다. 이게 어떤것을 의미하냐면 연동된 톰켓 3대중에 한대를 별도의 서버 셧다운을 하지 않아도 제외시킬수 있으며 반대로 다시 투입도 가능하다는 이야기이다. 이를 활용해보면 서비스 배포를 할 경우 위와 같은 설정이 되어있을때 제외 > 배포 > 투입하는 식으로 서비스가 무중단 상태에서 배포가 될수 있는 효과를 얻을 수 있다.
설치는 별도로 하지 않아도 되고 mod_jk 모듈 내에 있기 때문에 별도의 설정만 추가해주면 된다.
- apache/conf/httpd.conf
```markdown
<IfModule jk_module>
    JkWorkersFile    conf/workers.properties
    JkLogFile        logs/mod_jk.log
    JkLogLevel       info
    JkMount /* load_balancer

    <Location /jkmanager/>
        JkMount jkstatus
        Order deny,allow
        Deny from all
        Allow from 127.0.0.1
        Allow from {접근 가능한 IP}
    </Location>
</IfModule>
```
- apache/conf/workers.properties
```
worker.list=jkstatus
worker.jkstatus.type=status
```

설정에서 볼 수 있듯이 해당 설정은 다른측면에서는 상당히 취약점이 많은 부분이다. 해당 설정이 외부에 노출이 되어있다면 그 컨트롤을 서버 관리자가 아닌 다른 누군가가 할수 있기 때문에 꼭 Allow 설정으로 접근 제한을 해둬야 한다. 이렇게 하고 `서버 IP/jkmanager/` 을 접속해보면 "JK Status Manager" 이라는 문구와 함께 아파치에 연동된 톰켓의 상태를 한눈에 파악할 수 있다.

{{< image src="/images/apache-load-balancing/jk_status_manager.jpg" caption="" width="80%" >}}

여기서 tomcat1 좌측에 있는 `E`(=edit)를 클릭하고 Activation 값을 "Disabled" 으로 바꿔본 뒤 앞서 테스트한 방법을 다시 해보면 tomcat1 에는 엑세스가 들어오지 않고 9번 엑세스가 골고루 tomca2 와 tomcat3 으로 로드밸런싱이 된것을 확인할 수 있다.

{{< image src="/images/apache-load-balancing/apache_lb_test_jkmanager.jpg" caption="" width="80%" >}}

## 마치며
각 설정값들은 아무리 필자가 설명을 잘 해도 도큐먼트를 따라갈 수 없듯이 실제 각 도큐먼트를 보면서 설정값 하나하나를 조절해보며 운영하고 있는 서비스의 특징과 상황에 맞도록 맞춰가는것이 핵심일것 같다. (본 포스팅은 아주 가볍게 연동만 해보는 형태이고, 각 설정이나 워커들간의 우선순위 로드밸런싱 같은 경우는 직접 설정을 해가면서 확인이 필요하다. )
사실 이부분은 머릿속으로는 어떻게 하는구나라고 알고만 있었는데 실제로 해보니 각 설정들이 어떤 의미이고 어떻게 조절하면 보다 더 좋은 성능이나 다양한 이득을 취할수 있을것 같다는 생각을 해본다.

- 참고 링크
https://tomcat.apache.org/connectors-doc/reference/workers.html
https://tomcat.apache.org/connectors-doc/common_howto/loadbalancers.html