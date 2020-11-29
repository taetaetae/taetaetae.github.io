---
title: linux(centOS)에서 selenium 설정하기 (feat. python)
date: 2018-02-01 14:52:10
categories: [tech]
tags:
  - linux
  - python
  - selenium
  - chromedriver
  - archives-2018
url : /2018/02/01/linux-selenium/

---
테스트 코드로 안되는 실제 브라우저단 사용성 테스트를 하고싶은 경우가 있다. 이를테면 화면이 뜨고, 어떤 버튼을 누르면, 어떤 결과가 나와야 하는 일련의 `Regression Test`. 이때 활용할수 있는게 다양한 도구가 있지만 이번엔 `selenium` 에 대해서 알아보고자 한다.<!-- more -->

처음부터 사실 web application 테스트를 하려고 selenium 를 알아보게 된건 아니고, 내가 참여하고 있는 특정 밴드(네이버 BAND)에서 일주일에 한번씩 동일한 형태의 글을 올리고 있는데 (일종의 한주 출석체크 같은...) 이를 자동화 해볼순 없을까 하며 밴드 API를 찾아보다 selenium 라는것을 알게되었고, 매크로처럼 어떤버튼 누르고 그다음 어떤버튼 누르고 하는 일련의 과정을 코드로 구성할수 있다는 점에 감동을 받아(?) + 별도의 API를 발급받지 않아도 되어 사용하게 되었다. (물론 UI가 바뀌면 골치아프겠지만...)

> 여기서는 selenium 이 무엇인지에 대한 설명은 하지 않는다. (인터넷에 나보다 정리 잘된글이 많으니...) 단, linux 환경에서 셋팅하는 정보가 너무 없고 몇일동안 삽질을 한게 아쉬워서 그 과정을 포스팅 해본다. (나같은 분이 이 글을 보고 도움이 되실꺼라는 기대를 갖으며...)

> ※ 주의 : 본 포스팅은 밴드 서비스에 글을 올릴수 있는 비 정상적인 방법의 공유가 아닌, selenium에 대한 사용 후기(?)에 대한 글입니다. (참고로 막혔어요 -ㅁ-)

## 설정하기
서버 환경은 CentOS 7.4 64Bit + Python 3.6.3 + jdk 8 이다. 우선 selenium 을 설치해준다.
```
$ sudo python3.6 -m pip install selenium
```
그 다음 CentOS에서 크롬브라우저를 설치하기 위하여 yum 저장소를 추가한다. (꼭 크롬이 아니더라도 파이어폭스나 지금은 지원이 끊긴 팬텀JS 같은것으로 활용할수도 있으나 다른것들도 해봤는데 자꾸 설정에서 걸려서 크롬에 대한 내용을 포스팅 한다.)
```
$ sudo vi /etc/yum.repos.d/google-chrome.repo
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/x86_64
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
```
그리고는 yum 으로 크롬 브라우저를 설치한다. (내가 설치했을때의 버전은 `google-chrome-stable.x86_64 0:64.0.3282.119-1`)
```
$ yum install google-chrome-stable
```

크롬드라이버를 설치해야한다. 다음 url에서 받을수 있는데 https://sites.google.com/a/chromium.org/chromedriver/downloads 나는 2.35 linux64 버전을 받았다. 다운받고 unzip 하면 딱하나 파일이 있는데 나중에 selenium 을 사용할때 이용되니 path를 알아두자.

그다음 파이썬 코드를 작성한다. 내가 짠 파이썬 코드는 다음과 같은 순서로 실행이 된다.
- 밴드 접속 ( https://band.us/home )
- 로그인
- 특정 밴드 선택
- 글쓰기 버튼 누르고 양식에 맞춰 글 작성
- 글 등록

파이썬 코드는 아래처럼 작성하였다. (중요부분만.. 그 아래는 자유)
```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

# 초기화 --------------------------------------------
chrome_options = Options()
chrome_options.add_argument("--headless")

driver  = webdriver.Chrome(executable_path='home/~~~/chromedriver', chrome_options=chrome_options)
driver.implicitly_wait(3)
driver.get('https://band.us/home')

# 로그인 --------------------------------------------
driver.find_element_by_class_name('_loginLink').click()
...생략
```
그리고 실행을 해보면 작동이 잘~ 된다.
{{< image src="/images/linux-selenium/result.png" src_l="/images/linux-selenium/result.png" caption="selenium + python 으로 자동작성된 밴드 글" >}}



## 마치며
`selenium` 에 대해 찾아보면 거의 윈도우 환경에서 돌아가는것들에 대한 포스팅이 많았다. 난 리눅스 환경에서 스케쥴러(젠킨스 같은)를 통해 자동으로 화면없이 작동시키고 싶었는데 아무리 찾아봐도 + 삽질해도 잘 안되었다. 결국 사내에도 나같은 삽질을 하신 분을 찾고 묻고 물어 `크롬드라이버`만 있어야 하는것이 아니라 `크롬앱`또한 있어야 동작을 한다는것을 알게 되었다.
**역시, 내가 한 삽질은 누군가 이미 한 삽질이라는걸 다시한번 깨닳은 좋은(?) 시간이였다.**

이걸로 나중에 내가 맡고있는 서비스에 대한 웹 자동 테스트 툴도 만들어 볼 생각이다.