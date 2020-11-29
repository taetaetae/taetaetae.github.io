---
title: github api 사용방법
date: 2017-03-02 11:18:05
categories:
  - tech
tags:
  - github
  - archives-2017
url : /2017/03/02/github-api/
featuredImage: /images/github-api/github.png

---
github 에서는 레파지토리의 전반적인 상황에 대해 다양한 API를 제공해주고 있다. 이번에는 그 API를 사용하는 방법에 대해 알아보고자 한다.

## Personal access tokens 발급
우선 정상적인 API를 사용하기 위해 `Personal access tokens`를 발급받아야 한다. github 초기화면 > 우측상단 프로필사진 클릭 > setting > Personal access tokens 에 들어가 토큰을 생성을 한다.
해당 토큰의 허용범위를 설정한뒤 생성을 하면 만들어 지는데 여기서 발급되는 문자열은 따로 보관하는게 좋다. (나중에 다시 확인하려면 새로 재 생성하는 방법말고는 없기 때문에 한번 만들때 메모해 두는게 좋다.)
아래와 같이 생성완료.
{{< image src="/images/github-api/github_access_tokens.png" caption="" width="80%" >}}

## API 사용방법
권한이 없는 Repository 의 내용을 확인할수 없듯이 github에서 제공하는 API또한 권한이 있는 Repository에 대해서만 API를 제공한다. 위에서 발급한 token 을 권한 체크할때 사용하는데 다양한 방법이 있을수 있겠으나 나는 간단하게 헤더에 포함시켜서 일반 GET 호출을 하는 방식으로 하였다. 윈도우 환경에서는 헤더 셋팅하고 호출하는게 조금 어려울수 있으니 이러한 부분을 설정할수 있는 `Postman`이라는 프로그램으로 호출을 해본다.
아래처럼 url은 `https://api.github.com/`으로 설정하고 `Headers`파라미터에 `Authorization`라는 key에 value를 위에서 발급받았던 token을 이용하여 `token abcd~~`식으로 입력해준다음 `send`버튼을 눌러주면 응답을 받을수가 있는데, 아래 그림은 제공하는 api의 모든 url을 확인하는 방법이다.

{{< image src="/images/github-api/github_api_call_1.png" src_l="/images/github-api/github_api_call_1.png" >}}

아래애서는 위에서 확인된 api url을 활용하여 내가 권한이 있는 레파지토리 내에서 확인할수있는 정보에 대한 API를 호출해보았다.

{{< image src="/images/github-api/github_api_call_2.png" src_l="/images/github-api/github_api_call_2.png" >}}

API 호출시 가장 보편화되어있는(?) 스팩인 `JSON`으로 응답이 내려오기때문에 어떠한 환경에서도 충분히 활용할수 있을것이라 생각한다.
나는 개인적으로 팀 내에서 하나의 `Organizations`내에 여러 `Repository`가 있는데 각각의 `PullRequest`에 대해 코드리뷰를 해야하는 상황에서 일일히 다 찾아보기 귀찮아 github-api를 활용해 open된 `PullRequest`가 있으면 알림을 주는 걸 만들어 보았다.
