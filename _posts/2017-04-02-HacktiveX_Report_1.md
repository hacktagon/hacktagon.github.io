---
layout: post
title: "ActiveX Bug Hunting"
headline: ActiveX
modified: 2017-04-02
categories: RealWorld ActiveX HacktiveX Persu Elmo XothSec
comments: true
featured: true
---

# ActiveX 취약점 분석

<br>

1. 분석 개요

   * 분석 대상

      |Category|Contents|
      |-|-|
      |URL|XXXXX|
      |Service|XXXX 게임|
      |S/W|게임 시작을 위한 ActiveX|

<br>

   * 분석 환경

     * 본 분석은 모의해킹 형식으로 Attacker와 Victim으로 나누어 진행되었으며, 각각의 환경은 다음과 같다. 또한 취약점을 찾는데 있어 사용된 환경은 Victim과 같고, 표에 기재된 툴을 사용하여 분석하였다.

        |Category|Contents (Attacker)|
        |-|-|
        |운영체제|Windows XP 64Bit|
        |서버 구축 정보|APM Setup 7|

        |Category|Contents (Victim)|
        |-|-|
        |운영체제|Windows 8.1 K 64Bit|
        |서버 구축 정보|Internet Explorer v11.0.9600.18500|

        |Tools|Contents|
        |-|-|
        |IE 개발자 도구|Web 소스 분석|
        |IDA 6.8|DLL 분석|
        |VM Ware 12 Pro|Attacker 환경 구축|
        |Wireshark 2.0.1|패킷 분석|

        <br>

     * 공격 흐름도

       * 프로그램 정상 작동 흐름도

       <br>

       <img src="{{ site.url }}/images/2017-04-02/5.png" style="display: block; margin: auto;">

       * 공격 흐름도

       <br>

       <img src="{{ site.url }}/images/2017-04-02/6.png" style="display: block; margin: auto;">

       <br>

2. Proof of Concept

   * 프로그램 정상 작동 로직

     * Web 영역

     <br>

     <img src="{{ site.url }}/images/2017-04-02/1.png" style="display: block; margin: auto;">

     Target URL에 접속 시 XXXXX.dll이 설치되며 이는 위 GAME START 버튼 클릭 시 사용된다.

     * 게임 실행

     <br>

     <img src="{{ site.url }}/images/2017-04-02/2.png" style="display: block; margin: auto;">

     GAME START 버튼 클릭 시 GameStart() 함수를 호출하는 것을 확인할 수 있으며, 해당 함수는 아래 그림5와 같다. GameStart() 함수는 /cab/active.php를 요청하고, 반환된 응답(그림 6)을 이용하여 uid, upass, gname, gurl, gender, age, shutdown변수로 설정한다. 이 변수들은 fnStart() 함수의 인자로 사용 된다.

     <br>

     <img src="{{ site.url }}/images/2017-04-02/3.png" style="display: block; margin: auto;">

     /cab/active.php를 호출하여 반환된 응답을 이용하여 uid, upass, gname, gurl, gender, age, shutdown을 세팅하고 fnStart 함수 호출.

     <img src="{{ site.url }}/images/2017-04-02/4.png" style="display: block; margin: auto;">

     |인자명|반환된 값 (/cab/active.php)|
     |-|-|
     |uid|a5e9w80vb2c~ (암호화된 것으로 추정)|
     |upass|40098|
     |gname|xxxxxxxx|
     |gurl|http://xxxxxxxxxxxxx.xxxxx.co.kr/xxxxxxx/xxxxxxx/xxx|
     |gender|XXXXXXXXX (생년월일과 성별 값)
     |age|XX|
     |shutdown|success|

     <br>

     <img src="{{ site.url }}/images/2017-04-02/7.png" style="display: block; margin: auto;">

     fnStart() 함수에서는 변수 a (uid)를 통해 로그인 여부를 체크하고, 변수 k(shutdown)으로 셧다운제 적용 대상 여부를 체크한다. 통신 장애를 확인한 뒤, XXXXXXXXXXXX 라는 object로  <span style="color:red">DLL 내부의 Start()함수를 호출</span>한다.

     Start함수는 다음과 같은 인자를 가지고 호출된다. 첫 번째 인자는 게임명이며, 두 번째 인자는 info파일 (2.1.2.에서 설명)을 다운받기 위한 URL이다.

     Start("xxxxxxxx", "http://xxxxxxxxxxxxx.xxxxx.co.kr/xxxxxxx/xxxxxxx/xxx", “a5e9w80vb2c5bbbc3105383b6c77e4cdcd6ac152fffd405ec82dbd8e8d1be9ad86e5d263a1e415d7e671116fff57dcc4dd60f1f8baafa597af1ad27a0989e28de0692b3c5c0c808ef8ade0778c451da86eeff62e3e8735b21af702a91774e496cfff2abc72544c623d5ea989dbecd15656b95c6b9”, 40098, XXXXXXXXX, ‘1’, ‘1’, ‘1’, ‘1’, ‘S’);

     <img src="{{ site.url }}/images/2017-04-02/17.png" style="display: block; margin: auto;">

     fnStart() 함수밖에서 XXXXXXXXXXXX object의 정의를 찾을 수 있고, 해당 object는 XXXXXXXX.dll (ActiveX)를 제어한다

     * ActiveX 영역

     <br>

     <img src="{{ site.url }}/images/2017-04-02/8.png" style="display: block; margin: auto;">

     IDA를 통해 DLL을 분석하면 Start() 함수를 찾을 수 있다. 현재 lpParameter+176에는 인자로 넣어준 게임 이름이, lpParameter+689에는 인자로 넣어준 URL이 들어가 있다. 따라서 http://xxxxxxxxxxxxx.xxxxx.co.kr/xxxxxxx/xxxxxxx/xxx/XXXXXXXXXXX/xxxx.info 를 다운받아 C:\\---------\\----------\\xxxx.info로 저장하게 되는 것이다. 이는 아래 그림과 같이 Wireshark로 확인이 가능하다.

     <img src="{{ site.url }}/images/2017-04-02/9.png" style="display: block; margin: auto;">

     원본 info 파일의 내용은 다음 그림과 같다. IF_PATCH_URL은 패치를 진행하는 동안 파일을 다운 받을 URL이며, IF_PATCH_FNAME은 패치를 진행할 목록의 정보를 가지고 있는 파일의 이름이다. 또한 IF_EXE_FNAME은 패치가 끝난 뒤 게임을 실행하는 로직에서 사용되며, 실행할 파일의 이름을 가지고 있다.

     <img src="{{ site.url }}/images/2017-04-02/10.png" style="display: block; margin: auto;">

     다음 그림에서는 info 파일 내부의 IN_PATCH_URL의 값을 ReturnedString에 저장하고, IF_PATCH_FNAME 의 값을 v104에 저장한 뒤, 게임 프로그램을 패치하는 것을 확인할 수 있다.

     <img src="{{ site.url }}/images/2017-04-02/11.png" style="display: block; margin: auto;">

     URL에서 v104에 해당하는 패치 파일 리스트를 다운로드 한 뒤에 패치 파일 리스트를 확인하여 프로그램의 CRC32가 다른 경우 패치를 진행한다. 패치 파일 리스트와 프로그램을 다운로드 하는 것은 Wireshark에서도 확인이 가능하다.

     원본 패치 파일 리스트는 다음 그림에서 확인할 수 있다.

     <img src="{{ site.url }}/images/2017-04-02/12.png" style="display: block; margin: auto;">

     패치 파일 리스트 다운로드
     <img src="{{ site.url }}/images/2017-04-02/13.png" style="display: block; margin: auto;">

     프로그램 다운로드
     <img src="{{ site.url }}/images/2017-04-02/14.png" style="display: block; margin: auto;">

     <img src="{{ site.url }}/images/2017-04-02/15.png" style="display: block; margin: auto;">

     info파일에 있던 IF_EXE_FNAME의 값(실행할 파일의 이름)을 “C:\\---------\\----------”이라는 문자열 뒤에 붙여주고, 이를 lpParameter+5818에 저장한다. 그리고 lpParameter+5818을 이용하여 실행할 프로그램의 파일명과 그 프로그램에서 사용할 인자 값(argv)들을 붙여서 lpParameter+6078에 저장한다. 그 다음 CreateThread() 함수를 이용하여 새로운 쓰레드를 만들고, StartAddress() 함수를 호출한다.

     <img src="{{ site.url }}/images/2017-04-02/16.png" style="display: block; margin: auto;">

     StartAddress() 함수에서는 CreateProcessA() 함수를 이용해 lpThreadParameter+6078에 담겨있는 정보를 실행한다.

     즉 아래 그림과 같이 XXXXXXXXX.exe 가 실행 된다.

     <img src="{{ site.url }}/images/2017-04-02/18.png" style="display: block; margin: auto;">

   * 프로그램 공격 로직

     * Web 영역

     <br>

     <img src="{{ site.url }}/images/2017-04-02/19.png" style="display: block; margin: auto;">

     위 그림은 Web 영역의 정상 작동 방법을 활용하여 직접 작성한 html POC 코드이다. fnStart() 함수에서 XXXXXXXXXXXX라는 object로 DLL 내부의 Start()함수를 호출했듯이 Test object를 생성하여 Start()함수를 호출한다.

     첫 번째 인자인 게임명을 my_poc_test로, 두 번째 인자인 info 파일을 다운받기 위한 URL을 공격자 서버로 가정된 VM 내부의 영역의 주소로 변경하여 공격한다.

     * ActiveX 영역

     <br>

     <img src="{{ site.url }}/images/2017-04-02/20.png" style="display: block; margin: auto;">

     위 그림은 VMWare로 구성한 Attacker(서버) 환경이다. cmd 콘솔창을 통해 Attacker IP를 확인했다.

     <img src="{{ site.url }}/images/2017-04-02/21.png" style="display: block; margin: auto;">

     위 그림은 Attacker의 Web root 디렉토리 구성이다.

     <img src="{{ site.url }}/images/2017-04-02/22.png" style="display: block; margin: auto;">

     위 그림은 Web root 디렉토리 안에 있는 file.txt의 내용이다.

     file.txt는 정상 작동 로직에서의 filelist.txt(그림12)의 역할을 하는 파일이며, 같은 디렉토리 안에 있는 Hacktive_POC.exe의 파일명, 크기와 CRC32를 담고있다. 이 CRC32는 원본 filelist.txt의 안에 있던 WindyZone.exe의 CRC32와 동일하게 설정했다. Hacktive_POC.exe는 악성 프로그램이라고 간주한다.

     <img src="{{ site.url }}/images/2017-04-02/23.PNG" style="display: block; margin: auto;">

     위 그림은 하위 디렉토리 XXXXXXXXXX 구성이다.

     <img src="{{ site.url }}/images/2017-04-02/24.PNG" style="display: block; margin: auto;">

     Web root 하위의 XXXXXXXXXXX 디렉토리에 있는 “my_poc_test.info” 파일은 사용자가 ActiveX를 통해 다운로드 받을 info 파일이며, 내용은 위 그림24와 같다. IF_PATCH_URL에는 파일 다운로드 URL인 Attacker의 IP를 넣어줬다. IF_PATCH_FNAME에는 악성 프로그램을 다운받도록 할 정보를 가지고 있는 리스트(file.txt)를 넣어줬다. IF_EXE_FNAME은 다운받을 악성 프로그램의 이름을 의미한다.

     <img src="{{ site.url }}/images/2017-04-02/25.png" style="display: block; margin: auto;">

     위 그림을 통해 Victim이 html POC코드를 실행 또는 접속하면, Attacker의 서버로부터 변조된 info 파일을 다운받는 모습을 Wireshark로 확인할 수 있다,

     <img src="{{ site.url }}/images/2017-04-02/26.PNG" style="display: block; margin: auto;">

     변조된 info 파일로 인하여 Victim이 file.txt 파일을 다운받는다. 같은 방식으로 file.txt로 인하여 Victim은 Hacktive_POC.exe를 다운받으며, 이를 Wireshark를 통해 확인할 수 있다.

     <img src="{{ site.url }}/images/2017-04-02/27.{O}" style="display: block; margin: auto;">

     다음 그림에서 Victim의 컴퓨터에서 POC.html을 실행하면 변조된 info 파일 내부의 IF_EXE_FNAME로 인하여 악성 프로그램 Hacktive_POC.exe가 실행된다.

     <img src="{{ site.url }}/images/2017-04-02/28.png" style="display: block; margin: auto;">

     <br>

3. 결론

   * 조치 방안

     * Web 영역 조치 방안

     <br>

     Web 영역에서는 해당 ActiveX의 Start() 함수 실행 시 사용되는 인자 값을 공격자가 확인 할 수 없도록, 인코딩을 하는 방안이 필요하다.

     * ActiveX 영역 조치 방안

     <br>

     게임 업데이트를 위한 info 파일 내 URL을 준인터가 패치를 위해 제공하는 주소로 고정 해야한다.

     게임의 특성상 패치가 자주 일어나게 되고 이에 따라 패치버전 별 관리를 위해 유동적인 패치 URL의 사용이 필요할 수 있다. 이와 같은 경우에는 URL의 Domain name부분이 패치를 위해 제공하는 URL인지 검증하는 단계가 필요하다.
