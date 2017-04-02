---
layout: post
title: "ActiveX Bug Hunting"
headline: ActiveX
modified: 2017-04-02
categories: RealWorld ActiveX XothSec
comments: true
featured: true
---

# ActiveX 취약점 분석

1. 분석 개요

   * 분석 대상

     * URL : XXXXX

     * Service : XXXX 게임

     * S/W : 게임 시작을 위한 ActiveX

   <br>
   * 분석 환경


<br>
2. Web 영역

   * Original Logic

     * URL 접속 및 ActiveX 설치

     <img src="{{ site.url }}/images/2017-04-02/1.png" style="display: block; margin: auto;">

     <br>

     URL에 접속 시 ActiveX 를 설치.

     <br>

     * 게임 실행

     <img src="{{ site.url }}/images/2017-04-02/2.png" style="display: block; margin: auto;">

     <br>

     게임 실행 버튼 클릭 시 GameStart 함수 호출.

     <br>

     <img src="{{ site.url }}/images/2017-04-02/3.png" style="display: block; margin: auto;">

     <br>

     /cab/active.php를 호출하여 반환된 응답을 이용하여 uid, upass, gname, gurl, gender, age, shutdown을 세팅하고 fnStart 함수 호출.

     <br>

     <img src="{{ site.url }}/images/2017-04-02/4.png" style="display: block; margin: auto;">

     uid는 어떤 암호화되어있다고 판단되는 값이 들어있고, upass는 40098, gname은 xxxx, gurl은 XXXXXXXXXXX 값이 gender=XXXXXXX 로 생년월일과 성별 값을 나타내며, age는 XX, shutdown은 success가 들어있음.

     결국 fnStart 함수는 다음과 같은 인자를 가지고 호출됨.

     fnStart(“a5e9w80vb2c5bbbc3105383b6c77e4cdcd6ac152fffd405ec82dbd8e8d1be9ad86e5d263a1e415d7e671116fff57dcc4dd60f1f8baafa597af1ad27a0989e28de0692b3c5c0c808ef8ade0778c451da86eeff62e3e8735b21af702a91774e496cfff2abc72544c623d5ea989dbecd15656b95c6b9”, 40098, “xxxx”, “XXXXXXXXXXX”, XXXXXXX, XX, “success”)
