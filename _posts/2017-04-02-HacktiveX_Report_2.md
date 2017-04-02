---
layout: post
title: "ActiveX Bug Hunting 2"
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
      |S/W|게임 설치를 위한 ActiveX|

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

       <img src="{{ site.url }}/images/2017-04-02/post2/1.png" style="display: block; margin: auto;">

       * 공격 흐름도

       <br>

       <img src="{{ site.url }}/images/2017-04-02/post2/2.png" style="display: block; margin: auto;">

       <br>

2. Proof of Concept

   * 프로그램 정상 작동 로직

     * Web 영역

     <br>

     <img src="{{ site.url }}/images/2017-04-02/post2/3.png" style="display: block; margin: auto;">

     Target URL에 접속 시 XXXXXXXXXXXXXXXXXXX.ocx이 설치되며 이는 위 GAME START 버튼 클릭 시 사용된다.

     * 게임 실행

     <br>

     <img src="{{ site.url }}/images/2017-04-02/post2/4.png" style="display: block; margin: auto;">

     최초 GAME START 버튼 클릭 시 /common/fort_shutdown.asp 를 요청하게 된다.

     <br>

     <img src="{{ site.url }}/images/2017-04-02/post2/5.png" style="display: block; margin: auto;">

     이후 서버는 “false” 라는 응답을 보내게 된다.

     <img src="{{ site.url }}/images/2017-04-02/post2/6.png" style="display: block; margin: auto;">

     TnStartLauncher() 함수에 따라 Dll 내부의 Run함수를 실행하게 된다. Run 함수는 \\Launcher\\Launcher.exe를 실행하게 하는 함수로, 해당 exe 파일이 없을 시 SetupPath에서 파일 다운로드를 요청하게 된다.

     <br>

     <img src="{{ site.url }}/images/2017-04-02/post2/7.png" style="display: block; margin: auto;">

     SetupPath의 경로에서 Launcher를 다운받고 실행한다.

     <img src="{{ site.url }}/images/2017-04-02/post2/8.png" style="display: block; margin: auto;">

     위 그림에서 Launcher(설치 마법사)를 통해 게임 프로그램을 다운 받는 것을 볼 수 있다.

   * 프로그램 공격 로직

     * Web 영역

     <br>

     <img src="{{ site.url }}/images/2017-04-02/post2/9.png" style="display: block; margin: auto;">

     위 그림은 Web 영역의 정상 작동 방법을 활용하여 직접 작성한html POC 코드이다. Run() 함수에서 SetupPath를 공격자의 서버로 가정된 VM 내부의 영역의 주소로 변경하여 공격한다.

     * ActiveX 영역

     <br>

     <img src="{{ site.url }}/images/2017-04-02/post2/10.png" style="display: block; margin: auto;">

     위 그림은 VMWare로 구성한 Attacker(서버) 환경이다. cmd 콘솔창을 통해 Attacker IP를 확인했다.

     <img src="{{ site.url }}/images/2017-04-02/post2/11.png" style="display: block; margin: auto;">

     그림 13 처럼 가상 환경에서 XXXXXXXXXXXXXXXXXXXXX 이라는 이름을 가진 악성 파일을 위치 시킨다.
     ActiveX에서는 SetupPath 의 URL에서 파일을 다운로드 및 실행하여 런쳐를 다운받으려 한다. 따라서 SetupPath 에 바로 악성코드를 삽입하여도 된다. 하지만 그렇게 할 경우 C:\\XXXXXXXXXXXXX\\XXXXXXXX 디렉토리에 XXXXXXXX.exe가 다운로드 되지 않고, 그 경우 에러가 발생한다. 즉 피해자가 공격을 당했음을 인식할 수 있다. 따라서 Dropper를 두었고 그 역할은 다음과 같다.
     Dropper는 C:\\XXXXXXXXXXXXX\\XXXXXXXX 디렉토리를 생성한다. 그 다음 공격자의 악성코드를 XXXXXXXX.exe를 해당 디렉토리로 다운 받는다. 그러면 자연스럽게 ActiveX의 Run() 함수 내부에서 C:\\XXXXXXXXXXXXX\\XXXXXXXX\\XXXXXXXX.exe, 즉 공격자의 악성코드를 실행한다.

     Dropper에 해당하는 소스는 다음과 같다.

     <img src="{{ site.url }}/images/2017-04-02/post2/12.png" style="display: block; margin: auto;">

     다음 그림에서 Victim의 컴퓨터에서 POC.html을 실행하면 변조된 info 파일 내부의 IF_EXE_FNAME로 인하여 악성 프로그램 Hacktive_POC.exe가 실행된다.

     <img src="{{ site.url }}/images/2017-04-02/post2/13.png" style="display: block; margin: auto;">

     <br>

3. 결론

   * 조치 방안

   ActiveX 내 Run() 함수에서 사용되는 SetupPath 인자 값을 사용자가 인자로 넣어줄 수 있는 형태로 작성 되어있다. 하지만 이는 URL 변조 공격을 통해 공격자 서버에서 파일들을 다운로드 받을 수 있는 위험이 있기 때문에 패치 해야 한다.

     * Web 영역 조치 방안

     <br>

     Web 영역에서는 해당 ActiveX의 Run() 함수 실행 시 사용되는 SetupPath 인자 값을 공격자가 확인 할 수 없도록, 인코딩을 하는 방안이 필요하다.

     * ActiveX 영역 조치 방안

     <br>

     게임 업데이트를 위한 OCX 파일 내 URL을 패치를 위해 제공하는 주소로 고정 해야한다.

     게임의 특성상 패치가 자주 일어나게 되고 이에 따라 패치버전 별 관리를 위해 유동적인 패치 URL의 사용이 필요할 수 있다. 이와 같은 경우에는 URL의 Domain name부분이 패치를 위해 제공하는 URL인지 검증하는 단계가 필요하다.
