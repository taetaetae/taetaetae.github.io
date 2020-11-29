---
title: Jenkins에서 파이썬 출력을 실시간으로 보고싶다면?
date: 2018-12-02 01:40:11
categories:
  - tech
tags: 
  - jenkins
  - python
  - archives-2018
url : /2018/12/02/python-buffer/

---
필자가 운영하고 있는 [Daily Dev Blog](http://daily-devblog.com) 라는 서비스는 매일 동일한 시간에 주기적으로 데이터를 크롤링 하고 사용자에게 메일을 발송하는 일련의 작업을 수행하고 있다. 헌데 예상하지 못한 부분에서 예외가 발생하게 되면 어떤경우는 메일 발송을 못한다거나 기존에 발송했던 데이터를 다시 보내는 등 정상적이지 못한 상황을 맞이하게 된다.<!-- more -->

{{< image src="/images/python-buffer/ddb_duplication.png" src_l="/images/python-buffer/ddb_duplication.png" caption="메일이 하루라도 잘못오면 여기저기서 연락이 온다. 감사한 분들..." >}}

이런저런 바쁜일들로 차일피일 미루다 마침 여유가 생겨 기존에는 Crontab 스케쥴로 파이썬 스크립트를 실행하던 것에서 Jenkins로 옮기는 작업을 했다. 젠킨스가 스케쥴링을 해주고 실행이력을 보여주며, 실시간으로 스크립트가 돌아가는걸 볼수 있을것 같다는 기대감에서이다. 위에서 이야기 했던 예외상황을 보다 빠르고 편하게 실시간으로 디버깅을 하기 위해서가 가장 컸다.

## 당연히 될거라고 생각했으나...
작업은 간단할꺼라 생각했다. 

1. 우선 [Jenkins를 설치](https://taetaetae.github.io/2018/12/02/jenkins-install/)하고 
2. 기존에 스크립트 파일을 Jenkins Job으로 옮긴후에 
3. 적당한 코드 중간중간에 디버깅이 용이하도록 로그를 출력하게 해둔다음
4. 스케쥴링만 걸어두면 끝이라고 생각했다. 

하지만, 이렇게 간단하게 끝날것만 같았던 작업이 은근 귀찮은 작업이 될줄이야. 디버깅을 위해 로그를 출력하도록 해놨는데 모든 스크립트가 끝이 나서야 해당 로그가 출력되는 것이였다. 로그를 실시간으로 볼수 없다면 Crontab에서 Jenkins로 옮기는 이유가 크게 없게 된다. 실제로 아래처럼 코드를 작성하고 Jenkins Job을 실행시켜보면 다 끝나고서야 출력이 되는걸 볼수 있었다. 

(1초에 한번씩 5초동안 로그를 찍는 간단한 코드다.)

```python
import time

print('start')

for second in range(0,5) :
  print(second)
  time.sleep(second)

print('end')
```

{{< image src="/images/python-buffer/before.gif" src_l="/images/python-buffer/before.gif" caption="스크립트가 다 끝나서야 출력을 볼수 있다ㅠ 실시간으로 디버깅이 어렵다." >}}


## 그럼 어떻게 해야할까?
개발을 하면서 만나는 대부분의 문제들은 누군가 과거에 경험했던 문제였고, 이미 해결된 문제일 확률이 상당히 높은것들이 많다. 이번에도 역시, 갓 스택 오버플로우 : https://stackoverflow.com/questions/107705/disable-output-buffering

위 링크에서 알려준것처럼 해보면 다음과 같이 로그가 출력되는대로 젠킨스에서 볼수 있게 된다.

{{< image src="/images/python-buffer/after.gif" src_l="/images/python-buffer/after.gif" caption="콘솔환경에서의 디버깅은 로깅이 최고!" >}}

정리해보면 다음과 같은 방법이 있겠다.
1. `Execute Python script` 을 활용하여 Jenkins 에 직접 코드를 작성하는 경우 
  - print의 flush옵션을 활용 ( https://docs.python.org/3/library/functions.html?highlight=print#print )

  ```python
  print('hello', flush=True)
  ```
  - 매번 print 가 될때마다 flush가 되도록 재정의

  ```python
  import sys

  class Unbuffered(object):
     def __init__(self, stream):
       self.stream = stream
     def write(self, data):
       self.stream.write(data)
       self.stream.flush()
     def writelines(self, datas):
       self.stream.writelines(datas)
       self.stream.flush()
     def __getattr__(self, attr):
       return getattr(self.stream, attr)

  sys.stdout=Unbuffered(sys.stdout)
  ```

2. `Execute shell`을 활용하여 특정경로의 Python 파일을 실행할 경우
  - `-u` 옵션을 줘서 실행시킨다. ( python -u python_module.py )

이렇게 두고보면 너무 간단한 작업인데 이런 방법을 모르는 상황에서는 작성된 Python Script를 Shell Script로 다시 감싸보거나 Python 코드를 쓰지 말까 까지 생각했었다... 삽질의 연속들... (Shell Script로 작성하면 바로바로 보였기 때문...)

다시한번 모르면 몸이 고생한다(?)라는걸 몸소 체험한 좋은...시간이였다.