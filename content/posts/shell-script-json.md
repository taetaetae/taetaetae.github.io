---
title: 리눅스상에서 json 파싱
date: 2017-02-28 17:50:44
categories: [tech]
tags:
  - linux
  - json
  - jq
  - archives-2017
url : /2017/02/28/shell-script-json/
featuredImage: /images/shell-script-json/jq.png
---

리눅스 상에서 json형태의 String 을 파싱해야하는 상황이라면 아래 라이브러리를 사용해보는것을 추천해본다.
<!-- more -->
## jq
사용방법은 너무너무 간단하다.
- 자신의 시스템에 맞는 라이브러리를 다운받고

```
(32-bit system)
$ wget http://stedolan.github.io/jq/download/linux32/jq
(64-bit system)
$ wget http://stedolan.github.io/jq/download/linux64/jq
```
- 실행 권한을 설정해 준 뒤

```
chmod +x ./jq
```

- root 권한으로 해당 파일을 이동시킨다.

```
sudo cp jq /usr/bin
```

- 실행은 다음과 같이 한다.
Json String 이 아래와 같이 있다고 가정했을때

```json
{
"name": "Google",
"location":
  {
    "street": "1600 Amphitheatre Parkway",
    "city": "Mountain View",
    "state": "California",
    "country": "US"
  },
"employees":
  [
    {
      "name": "Michael",
      "division": "Engineering"
    },
    {
      "name": "Laura",
      "division": "HR"
    },
    {
      "name": "Elise",
      "division": "Marketing"
    }
  ]
}
```

실제 사용과 결과는 다음과 같이 이루어 진다.
```
$ cat json.txt | jq '.name'
"Google"

$ cat json.txt | jq '.location.city'
"Mountain View"

$ cat json.txt | jq '.employees[0].name'
"Michael"

$ cat json.txt | jq '.location | {street, city}'
{
  "city": "Mountain View",
  "street": "1600 Amphitheatre Parkway"
}
```

보다 자세한 사용방법은 공식홈페이지( https://stedolan.github.io/jq/ )를 참조하면 좋을듯 하다.
