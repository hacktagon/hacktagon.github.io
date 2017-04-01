---
layout: post
title: "XSS Detection with Phantomjs"
headline: Python05
modified: 2017-04-01
categories: web
comments: true
featured: true
---

Phantomjs를 사용하여 XSS(Cross-site Scripting)를 탐지하는 방법에 대해 소개합니다. 

브라우저(Phantomjs)가 직접 스크립팅하여 취약점을 탐지하기에 문자열 탐지 방식보다 정확합니다. 하지만 Phantomjs가 페이지를 완성하는 시간만큼 추가적으로 시간이 소비 되는 단점이 있습니다.

본문에서는 Python을 통해 웹서버와 통신하고 받아온 HTTP Response를 Phantomjs에게 넘겨서 XSS 취약점이 발생하는지 확인하는 방법을 소개합니다. 추가적으로 Python이 아닌 Burpsuite에서 확장프로그램을 통해 탐지하는 방식을 소개합니다.

## 요구사항
- [phantomjs](http://phantomjs.org/)
- [python](https://www.python.org/) 2 or 3
- (선택) [burpsuite](https://portswigger.net/burp/)


## Python으로 웹서버와 통신하자
Python을 통해 웹서버에 질의를 하고 HTTP Response를 받아옵니다.

```python
# -*- coding: utf-8 -*-

import requests
import random

url = "http://xss.nisambot.com/anchor.php?returnUrl=index.php%22%3E%3Cscript%3Ewindow.alert%281%29%3C%2Fscript%3E%3Ca"

r = requests.get(url)

print(r.content)
```
HTML Response Body가 출력되면 Python의 준비는 끝났습니다.

> (참고) requests 라이브러리 의존성이 필요합니다.

## Phantomjs 정답을 알려줘!
Python에서 받아온 HTTP Response에 XSS가 있는지 Phantomjs가 판단합니다.
우선 Phantomjs xss detection Server를 구동합니다.

XSS Server 코드는 [XSS Validator의 xss server](https://github.com/nVisium/xssValidator/blob/master/xss-detector/xss.js)를 사용합니다. MIT License를 따릅니다.
xss.js의 172~174 라인 제거하고 사용하세요.
```javascript
// pageResponse = atob(pageResponse);
// pageUrl = atob(pageUrl);
// responseHeaders = atob(responseHeaders);
```

```
phantomjs xss.js
```
정상적으로 phantomjs가 실행되면 phantomjs의 준비는 끝났습니다.

> (참고) xss.js 코드에 메모리 관리를 제대로 하지 않는 결함이 있습니다. 완전한 자돔화 도구를 만드신다면 코드를 재작성하는것을 추천드립니다 :)


```python
# -*- coding: utf-8 -*-

import requests
import random

url = "http://xss.nisambot.com/anchor.php?returnUrl=index.php%22%3E%3Cscript%3Ewindow.alert%281%29%3C%2Fscript%3E%3Ca"

r = requests.get(url)

rh = json.dumps(r.headers.__dict__['_store'])
phantom_data = {"http-response": r.content, "http-url" : r.url, "http-headers": rh }
phantom_r = requests.post("http://127.0.0.1:8093", data=phantom_data)
phantom_json = json.loads(phantom_r.content)


if phantom_json["value"] is 1:
    print(r.url)
    print(phantom_json["msg"])
```

python 코드를 조금 수정하여 phantomjs 서버로 Response를 보내는 내용을 추가합니다.


```
python xss.py

http://xss.nisambot.com/anchor.php?returnUrl=index.php%22%3E%3Cscript%3Ewindow.alert%281%29%3C%2Fscript%3E%3Ca
XSS found: alert(1)XSS found: alert(1)
```
짠!!

## 합체 Burpsuite!
Python이 아닌 Burpsuite에서 직접 확장 프로그램을 통해 Phantomjs로 전달하고 결과를 받아오는 방법입니다.
별도로 설명하지 않고 잘 설명되어있는 글을 링크합니다.
https://nvisium.com/blog/2014/01/31/accurate-xss-detection-with-burpsuite/

## 결론
Phantomjs로 XSS를 탐지하는 방법을 소개했습니다. 간략하게 컨셉만 소개했지만, XSS는 자동화 도구를 통해 쉽게 탐지할 수 있는 취약점입니다. 자신만의 도구를 만들어 편하고 행복한 해킹라이프 즐기세요!

#### Reference
1. https://github.com/nVisium/xssValidator
