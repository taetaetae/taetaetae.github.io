---
title: 기술블로그 개편기 (by hugo)
date: 2020-11-29T18:12:15+09:00
categories:
  - tech
tags: 
  - hugo
  - blog
featuredImage: /images/blog-reorganization-by-hugo/hexo_to_hugo.png
images :
  - /images/blog-reorganization-by-hugo/hexo_to_hugo.png
---

　웹서비스 개발자라면 나만의 블로그쯤은 있어야지 하며 기술 블로그를 시작한 지도 어느덧 4년이 되었다. 처음엔 그저 새로 알게 된 기술이나 삽질하며 경험한 것들 중에 핵심만을 적어놓는 수준이었다. (지금 다시 보면 뭔가 오글거리는 건 기분 탓이겠지...) 그렇게 계속 글을 써오면서 글쓰기라는 것에 관심을 갖게 되고 내 글이 누군가에게 도움이 될 거라는 기대에 조금이라도 글을 잘 써보고자 단순 기록 용이 아닌 하나의 '글'을 쓰려고 노력해 온 것 같다.

　일주일에 한 개는 써야지. 한 달에 한 개는 써야지. 하며 자꾸 나 자신과의 타협을 하다가 최근에는 회사에서 운영하는 서비스 개편 때문에 정신없이 바쁘다는 핑계로 '블로그'에 'ㅂ'자도 생각하지 못하게 된다. 무엇이 문제일까?라는 생각은 결국 내 기술 블로그도 회사 서비스처럼 '개편'을 해보자는 생각으로 도달하게 되었고 간단할 것만 같았던 기술 블로그 개편 작업은 꽤 오랫동안 + 다양한 삽질들로 작업을 하게 된다.

　이번 포스팅에서는 기술 블로그를 개편하며 겪었던 내용들에 대해 정리해보고자 한다. 기존에 기술 블로그를 운영하시는 분들이나 이번에 새롭게 시작하시는 분들께 도움이 될 거라 기대한다. 더불어 서비스 '출시' 가 아닌 개편'이라는 과정 속에서 느끼게 되었던 인사이트도 간략하게 작성해볼까 한다.

## 기술블로그 플랫폼 선택

　처음 블로그를 쓰기 시작했을 때 포털서비스의 글쓰기 플랫폼을 사용하지 않은 이유는 단 하나다. '글쓰기' 뿐만 아니라 개발자이기에 웹사이트(블로그)를 내 입맛에 맞게 커스터마이징 하기 위해서. 그 이유로 [hexo](https://hexo.io/) 라는 프레임워크에 github의 호스팅을 사용하여 운영을 해왔다. 그렇게 블로그를 운영해오면서 느꼈던 불편했던 부분들과 개편을 하며 기대하는 부분들을 정리하면 아래와 같다.

- 테마(UI)가 이뻐야 하고 기능들이 많으면 좋겠다.
- 기술 블로그인 만큼 코드가 많이 삽입되니 코드 표현 또한 이뻐야 한다.
- 테마 또는 프레임워크의 커뮤니티가 활발해야 한다.
- 페이지 생성 또는 만들어진 웹페이지의 성능이 좋아야 한다.
- 글을 작성하고 배포하는 과정이 심플하고 깔끔해야 한다.
﻿

　위와 같은 이유를 기반으로 검색을 해보다 SSG(쓱 쇼핑몰 아님, Static site generators)를 깔끔하게 정리해 놓은 [사이트](https://jamstack.org/generators/)를 발견한다. 정말 다양한 플랫폼들을 살펴보며 필자에게 맞는 게 어떤 건지 고민하다 결국 [hugo](https://gohugo.io/) 를 선택하게 된다. hugo를 선택한 이유는 go라는 언어를 사용한다는 것과 (간접적으로라도 다른 언어를 경험해보고 싶어서 + go 언어가 빠르다는 소리를 어디선가 들어서) [테마들](https://themes.gohugo.io/)이 너무 다양했기 때문이다.

{{< image src="/images/blog-reorganization-by-hugo/hugo_homepage.jpg" caption="﻿아주 대놓고 빠르다고 하니... 쓰고 싶어진다." width="80%" >}}


　결국 hugo에 [hugo-ranking-trend](https://hugo-ranking-trend.com/)라는 사이트에서 상위에 랭크가 되어있고 기술 블로그 성격에 적합할 것 같은 [LoveIt](https://github.com/dillonzq/LoveIt)이라는 테마를 사용하기로 결정하였다. 자 그럼 시작해볼까?!

## hugo 는 어떻게 쓰는거야?
﻿　대부분의 오픈소스는 hello world 혹은 quick start 같이 처음 접하는 사람들을 위한 도큐먼트가 있기 마련. hugo도 마찬가지로 [quick-start](https://gohugo.io/getting-started/quick-start/)가 있었고 이를 천천히 따라 하면 생각보다 쉽게 초기 세팅을 할 수 있었... 을꺼라 기대했지만 약간 초기 설정 과정이 어려워서 남겨 두고자 한다.

> 참고로 필자는 윈도 10 환경에서 구성하였다. mac이라면 더 쉽게 설정할 수 있는 것 같은데 이 부분은 OS의 차이에서 생겨나는 어쩔 수 없는 약간의 장벽이라 생각한다. 이쁜 테마와 새로운 환경을 사용할 수 있다는 기대감으로 꾹 참아본다.

### 기본설정
　﻿git이 설치되어 있다는 가정하에 우선 hugo는 go 언어기반으로 돌아가기에 우선 go를 설치해야 한다. [다운로드](https://golang.org/dl/)페이지에서 환경에 맞는 설치 파일을 다운로드하고 설치를 해준다. 다음으로 패키지 관리자인 chocolatey 또한 설치가 필요하다. [공식 홈페이지](https://chocolatey.org/install)페이지에서 나와있는 순서대로 진행하면 설치 완료. 필자는 여기서 진행이 잘 안됐었는데, '관리자 권한'으로 PowerShell 을 실행시켜야지만 성공을 할 수 있었다.﻿

﻿　위 설정이 완료되었으면 드디어 hugo를 설치해 주고 초기화를 해준 뒤 샘플로 글 하나를 만들고 서버를 띄우면 끝.

  ```shell
  # chocolatey 에 의해 hugo 설치
  choco install hugo -confirm

  # hugo 초기화
  hugo new site quickstart

  # post 생성
  hugo new posts/post-name.md

  # 서버 run
  hugo server -D

  ```

### 글 작성부터 배포까지
﻿　위에서 이야기 한대로 `hugo new posts/post-name.md` 을 실행하면 `/posts/`하위에 아래의 형태처럼 기본 구조가 만들어진 `post-name.md`가 생겨난다. 파일명에서도 유추할 수 있듯이 마크다운으로 작성된다. (대부분의(?) SSG는 마크다운 문법으로 작성되는듯하다.)
```markdown
---
title: "My First Post"
date: 2019-03-26T08:47:11+01:00
draft: true
---
```

　﻿눈치가 빠른 사람이라면 알겠지만 일단 생성은 `draft`로 만들어진다. 즉, 작성한 것이 바로 릴리스가 되지는 않는다는 것. 그래서 로컬에서 서버를 띄울 때 `-D`옵션을 줘서 draft도 보이도록 띄워준다. 각 옵션에 대한 설명은 [공식문서](https://gohugo.io/getting-started/usage/)를 참고하는 게 좋다.

　글 작성은 앞서 말했지만 마크다운 형식으로 작성을 하고 `hugo`라고만 명령어를 작성하면 /public/ 하위에 마크다운을 기준으로 html이 생성이 된다. 그리고 해당 폴더에서 `{username}.github.io`로 git remote repository 설정을 해준 다음 push를 해주면 github에서 호스팅이 되어 블로그를 확인할 수 있다.
﻿
### 블로그 원본과 서비스의 분리
　﻿github 을 이용하여 블로그를 호스팅 할 경우 `원본` 과 `호스팅 되는 소스`로 나누어 관리를 하는 것이 유지 보수 측면에 좋다. 예컨대 블로그를 작성하는 곳이 지정된 PC만 한다면 상관없지만, 다양한 환경에서 작성할 수 있기에 구분이 되는 게 여러 가지로 좋았다. 과거 hexo로 블로그를 운영했을 때에는 repository를 두 가지로 나누어 하나는 `원본`을 다른 하나는 원본에 의해 생성되어 `호스팅 되는 소스`로 관리를 해왔는데 hugo의 경우는 어떻게 관리를 해야 할까?

　위에서도 이야기 한 것처럼 작성을 다 하고 생성(hugo 명령)을 하게 되면 /public/ 폴더 하위에 html 이 만들어진다. 이러한 구조에서 어떻게 하면 좋을지 고민하다 다음과 같은 구조를 생각해 보았다.
1. 블로그 원본 : 루트 폴더에 `.gitignore`로 /public/ 폴더를 제외하고 repository에 commit/push 하여 관리
2. 호스팅 소스 : /public/ 폴더 자체에서 새로운 repository로 commit/push 하여 관리

이 경우 글을 한번 작성하게 되면 다음과 같은 순서로 관리가 되다 보니 뭔가 배보다 배꼽이 더 큰 느낌이 든다.
﻿
```markdown
1. 글 작성
2. hugo 명령어로 html 생성
3. public 폴더에 가서 commit/push
4. 루트폴더에서 commit/push
```

　그래서 좀 더 알아보니 나와 같은 고민을 이미 누군가는 하였고, 깔끔하게 Github Action으로 만들어 둔 걸 찾을 수 있었다. [marketplace/actions#github-pages-action](https://github.com/marketplace/actions/github-pages-action) 대략 적어보면, 글을 작성하고 commit/push를 하게 되면 이를 캐치하여 Github Action 이 진행되고 Github Action 에서는 가상환경에서 hugo를 빌드하고 해당 내용을 gh-pages라는 이름으로 branch에 push를 해주는 흐름이다. 즉, 기존에는 repository를 두개로 관리했다면 지금은 하나의 repository에서 master(main) 과 gh-pages로 나뉘었고 이를 한 번의 commit/push로 자동화를 구성했다는 게 핵심이다.

　우선 repository에 접근할 [Personal access tokens](https://github.com/settings/tokens)을 생성해 준다. 이름은 나중에 사용되기에 편의상 `GITHUB_TOKEN`이라고 생성하고 권한은 `public_repo`만 있으면 된다. 그다음 위에서 이야기했던 [marketplace/actions#github-pages-action](https://github.com/marketplace/actions/github-pages-action)에서 아래 내용을 복사하고 블로그가 호스팅 되는 repository의 Actions 탭에 가서 New workflow 버튼을 누른 다음 그곳에 붙여 넣고 저장을 누르면 된다.

나머지는 그대로 가져왔고 branch 이름만 변경해 주었다.

```yaml
name: github pages

on:
  push:
    branches:
      - master  # Set a branch to deploy

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.75.1'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```
﻿
　이렇게 하고 repository에 push를 해보면 Github Action 이 실행되면서 gh-pages 브랜치에 빌드 된 내용을 push 해준 걸 확인할 수 있다. 그다음 repository settings > GitHub Pages에서 branch를 master가 아닌 gh-pages로 변경해 주면 끝!

{{< image src="/images/blog-reorganization-by-hugo/hugo_deploy_flow.jpg" caption="블로그 작성부터 배포까지의 흐름이 한방에!" width="80%" >}}


## 마이그레이션 (hexo to hugo)
﻿　엄청난 노가다의 시간들이었다. hexo 나 hugo 둘 다 markdown 문법으로 인식(?) 하고 생성되는 구조였지만 hexo의 테마 특성에 맞춘 문법, 그 반대인 hugo의 테마 특성에 맞춘 문법이 약간씩 달라서 모든 글을 한 번씩 보면서 체크해야 하는 수고를 들여야 했다.

﻿　더불어 hexo는 "글 제목"이라고 새 글을 생성하면 `yyyy/mm/dd/글 제목` 형식의 url로 표시가 된다. 하지만 hugo는 바로 `/글 제목`처럼 표현되는 부분 때문에 이대로 두었다가는 검색엔진이나 다른 곳에 기존 링크가 걸려있을 경우 없는 페이지로 랜딩 될 수밖에 없었다. 결국 과거 링크들을 살려야 해서 어쩔 수 없이 url을 과거의 url로 세팅해 줘야만 했다. ([Docs](https://gohugo.io/content-management/organization/#url-1))﻿

　마지막으로 문단의 길이나 이미지 사이즈 등 hugo 테마에 적용되며 이상해 보이는 부분들을 수작업으로 하다 보니 실무 서비스에서 마이그레이션 작업을 하는 것과 같은 느낌이 들었다. 물론 기술 블로그의 기반 언어나 프레임워크가 짧은 주기로 변경될 일은 크게 없지만 새로 글을 작성할 때에도 이런 부분들을 염두에 두고 작성해야겠다고 생각하게 되었다.

## 무엇이 좋아졌나?
﻿　우선 글 작성-배포-관리 측면에서 Github Actions으로 한 번에 자동화가 되니 훨씬 편리하였다. 그리고 글 작성할 때도 hugo의 [liveReload](https://github.com/livereload/livereload-js) 기능이 있어 내용이 변경될 때마다 (파일이 수정될 때마다) 서버에 바로 반영이 되니 작성하면서 생성되는 html 을 바로 볼 수 있다는 장점이 있었다. (hexo를 이용할 땐 글을 어느 정도 쓰고 서버 재시작, 다시 반복해야만 했다. 지금은 관련 기능이 있는진 모르겠지만...)

　전체적인 레이아웃이 이뻐진 건 기본이고 (우상단에 동그라미를 누르면 요즘 핫한 다크 모드가 된다. ^오^) 프론트 성능도 기존 대비 수치를 따져보진 않았지만 체감상 좋아진 것 같다. 가장 중요한 이번 기술 블로그 개편의 목적인 **글을 쓰고 싶게 만드는 환경이 구축되었다는 점이다.**

## 마치며

{{< image src="/images/blog-reorganization-by-hugo/NoThanksButWereBusy.png" caption="출처 : https://www.astroarch.com/tvp_strategy/no-thanks-busy-pay-back-technical-debt-40188/" width="80%" >}}

　위 그림을 보면 네모난 바퀴를 사람들이 끌고 있고 누군가 동그란 바퀴를 제안하고 있다. 하지만 수레를 끄는 사람들은 바쁘다는 핑계로 그 의견을 무시하고 수레를 끌고 가고 있다. 과연 바쁘면 얼마나 바쁘다고. 필자 또한 그러한 생각에 현실과 타협하는 나 자신을 벗어던지고 새롭게 다시 시작하기 위해 누가 시키지도 않은 블로그 개편을 하게 되었다. 이 글을 읽는 여러분들들 중 기술 블로그를 운영하는데 바쁘다는 핑계로 잘 못쓴다거나, 기술 블로그를 시작해야지 하면서 다음으로 미루기만을 반복하지는 않는지 생각해봤으면 한다. 그렇다면 아주 조금씩 시작해보는 건 어떨까?

　길다면 긴 개편 작업이 끝났다. 그동안 이런저런 이유로 블로그 포스팅을 소홀하게 했었는데 이번을 계기로 좀 더 `나를 위해` 열심히 작성해보자고 다짐을 해본다.