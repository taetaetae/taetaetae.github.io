---
title: hexo 블로그에 tranquilpeak 테마 적용하기
date: 2017-08-27 17:52:56
tags: 
  - hexo
  - tranquilpeak
  - archives-2017
categories:
  - tech
url : /2017/08/27/hexo-themes-tranquilpeak/
featuredImage: /images/hexo-themes-tranquilpeak/1.jpg
images :
  - /images/hexo-themes-tranquilpeak/1.jpg
---
여러가지 hexo 테마중에 그나마(?) 영어로 된 문서가 있어서 적용해보게 된 tranquilpeak 라는 테마. 오늘은 해당 테마를 적용하면서 겪은 문제, 그리고 적용 방법에 대해서 간략하게나마 정리해보고자 한다. (다른 테마들은 거의다 중국쪽이나 일본...)<!-- more -->
먼저 hexo 공식사이트에서 알려주는 테마들은 다음 사이트에서 확인해 볼수 있다. 
  - https://hexo.io/themes/index.html

기존에는 `hueman`이라는 테마를 사용하고 있었는데 ([링크](https://github.com/ppoffice/hexo-theme-hueman)), 오랜만에 블로그를 다시(?) 시작하는 느낌을 내보고 싶었고 보다 더 심플하고 유행에 안탈것 같은(순전히 필자 생각) 테마를 찾아보다 `tranquilpeak`이라는 테마를 선택하게 되었다. 
- 공식홈페이지 : https://github.com/LouisBarranqueiro/hexo-theme-tranquilpeak
- 샘플사이트 : http://louisbarranqueiro.github.io/hexo-theme-tranquilpeak/

우선 간략하게 설치과정을 나열해보면 다음과 같다.
1. themes 폴더내에 테마파일을 받은후 압축 해제 
2. 테마 폴더 이름을 변경
3. _config.yml 파일 내에 테마 설정 부분 변경 ( theme: tranquilpeak )
4. hexo clean → hexo generate → hexo server(or hexo deploy)

이렇게 하면 아주 간단하게 테마가 변경이 된다. 혹여나(필자처럼) 기존 테마를 커스터마이징 하고 싶을 경우는 별도의 과정이 추가로 필요하다. 기존에는 css나 js만 변경하면 간단히 수정되었는데 이 테마는 약간의 빌드(?)를 필요로 한다. 따라서 css나 js등 html 요소들을 수정하였다면 다음과 같은 과정이 필요하다.(테마폴더 최상위에서)
1. npm install
2. bower install 
3. css 나 js 변경
4. grunt build
5. hexo clean → hexo generate → hexo server(or hexo deploy)

나같은 경우는 테마에 적용된 폰트를 바꾸기 위해 [블로그](http://blog.lattecom.xyz/2016/05/08/tranquilpeak-theme-web-font) 를 참조하였다. (해당 아티클에다 댓글폭탄을 ㅎㅎ;;)