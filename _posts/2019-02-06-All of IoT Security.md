---
layout: post
title:  "All of IoT Security"
tag:   Analysis
categories: persu IoT
modified: 2019-02-06
comments: true
featured: true
---

# 개요
해당 글은 2년 정도 IoT 보안과 관련된 업무를 수행하면서, 자료를 정리하는 글입니다. IoT 보안과 관련된 공부를 수행하는 분에게 참고가 되었으면 합니다.

## 1. IoT 서비스 종류 파악
IoT 서비스는 사용 목적에 따라 크게 CIoT와 IIoT로 구분되게 됩니다.
- Client IoT : Smart Home, Healthcare
- Industry IoT : Smart Factory, Grid

아래 URL에서는 다양한 IoT 서비스에 대해 설명하고 있습니다.

- 한국사물인터넷협회 : https://www.kiot.or.kr/main/index.nx

- 사물인터넷 제품 및 서비스 열람 서비스 : https://www.kiot.or.kr/cms/fileDown?PAGE=1&SC_WORD=&SC_CATE=&CM_CODE=0u8j06&open_modal=&c=1676

- 사물인터넷융합포럼 포럼 표준자료실 : http://www.iotforum.kr/stan/stan.list.asp

- Building an Open End-to-End Internet of Things Architecture (IoT의 기본적인 아키텍처에 대해 설명을 하고 있음) : https://next.redhat.com/2017/12/19/building-an-open-end-to-end-internet-of-things-architecture/?lipi=urn%3Ali%3Apage%3Ad_flagship3_feed%3BKPRZYJ4HTWKAeIZgwqJd4Q%3D%3D&fbclid=IwAR3CiD8nYSvJXfniKaT7T4PrtiEiAeqy7onXjS6QXCnlYUhz-Bx8zFHOybE

- Table Comparing Wireless Protocols for IoT Device (IoT 디바이스에서 사용되는 무선 프로토콜 비교) : http://glowlabs.co/wireless-protocols/?fbclid=IwAR23_NPDVqi_inuzErE8pgIzZn8lAmrtRNMyKEJFQXIDyK8WLyLLghKG-Jw

- IoT 네트워크 기술 정보 : https://m.blog.naver.com/PostView.nhn?blogId=scw0531&logNo=220679511015&fbclid=IwAR0-Ju2zva3ZzzT0itBrPo4tMcKzEa8dYufZ3-72c4WH_flGP9vFex2m7pg&proxyReferer=https%3A%2F%2Fwww.facebook.com%2F

## 2. IoT 보안 가이드
IoT 관련해서는 국내/외 다양한 관련 부처에서 다양한 보안 가이드를 제공하고 있습니다. 이러한 보안 가이드는 IoT 보안 관련 컴플라이언스 수립과 보안 점검에 대해 중요한 기준이 될 수 있습니다. 아래 보안 가이드는 순수 IoT 및 스마트 홈에 관한 내용으로 아래를 정리하였으며, 산업 별 보안 권고 사항은 제외하겠습니다.

### 국내 KISA

[https://www.kisa.or.kr/public/laws/laws3_View.jsp?cPage=1&mode=view&p_No=259&b_No=259&d_No=80&ST=&SV=]
(https://www.kisa.or.kr/public/laws/laws3_View.jsp?cPage=1&mode=view&p_No=259&b_No=259&d_No=80&ST=&SV=)
- IoT 공통보안원칙 :
```
https://www.kisa.or.kr/public/laws/laws3_View.jsp?cPage=2&mode=view&p_No=259&b_No=259&d_No=67&ST=T&SV=
```
- IoT 공통보안 가이드 :


[https://www.kisa.or.kr/public/laws/laws3_View.jsp?cPage=1&mode=view&p_No=259&b_No=259&d_No=80&ST=&SV=]


- 홈ㆍ가전 IoT 보안가이드 : https://www.kisa.or.kr/public/laws/laws3_View.jsp?cPage=1&mode=view&p_No=259&b_No=259&d_No=93&ST=&SV=
- 사물인터넷(IoT) 환경에서의 암호인증기술 이용 안내서 : https://www.kisa.or.kr/public/laws/laws3_View.jsp?cPage=1&mode=view&p_No=259&b_No=259&d_No=84&ST=total&SV=

### 국내 KISA 정보보호클러스터 융합보안혁신센터
- IoT 보안인증서비스 : https://www.kisis.or.kr/user/bbs/kisis/66/312/bbsDataView/10176.do?page=1&column=&search=&searchSDate=&searchEDate=&bbsDataCategory=
- 홈네트워크건물인증 : https://www.kisis.or.kr/user/bbs/kisis/62/273/bbsDataView/8791.do?page=1&column=&search=&searchSDate=&searchEDate=&bbsDataCategory=


### 해외 NIST
- Interagency Report on Status of International Cybersecurity  Standardization for the Internet of Things :
https://csrc.nist.gov/CSRC/media/Publications/nistir/8200/draft/documents/nistir8200-draft.pdf

- Securing the Home IoT Network :
https://www.sans.org/reading-room/whitepapers/internet/paper/37717

### 해외 OWASP
- OWASP 2018 IoT Top 10 : https://www.owasp.org/index.php/OWASP_Internet_of_Things_Project

### 해외 일본
- 일본 IoT 보안평 지침 :
https://www.ccds.or.jp/public_document/index.html?fbclid=IwAR2pt8ezXISwgyg8YnGqmRAm4kZVoBEZ5AAiFc5kowb-_sZjeDn9cyq3GBA

### 해외 SANS
- A Black-Box Approach to Embedded Systems Vulnerability Assessment :
https://www.sans.org/reading-room/whitepapers/basics/paper/37452

### 기타 모음 자료
- IoT에 대한 Security & Privacy Guidelines :
```
https://www.schneier.com/blog/archives/2017/02/security_and_pr.html?fbclid=IwAR02gbKHU0tZCJG1AQ21q-1ryfi-jGnesceJha7RUgppNHf86JsgjZmQBxw
```

## 3. IoT 보안 입문 안내서
아래 URL은 IoT 해킹을 하기 위해 필요한 지식을 잘 정리한 자료입니다. 요즘은 쉽게 찾을 수 있는 내용이지만, 당시 작성했을 작성자분들에게 감사에 인사를 드립니다!

### [하드웨어 해킹]
- GrayHash 정구홍 수석님 자료 :
https://www.hackerschool.org/HardwareHacking/?fbclid=IwAR2Edhs8Fuml72N9gugJRaI5__TyE5GJ8Fy9TM7x4mkmcFbJmulg3Jbwe3k

- The JTAG Interface: An Attacker's Perspective : https://optivstorage.blob.core.windows.net/web/file/55e86eae3f04450d9bafcbb3a94559ca/JTAG.Whitepaper.pdf?fbclid=IwAR2M8NVJknkM8NpuiE6lMU-RcMKtMf5ejy4JOvitRiiYjLvWf4y9ugE3Pag

- Practical Reverse Engineering Part 1 (Hunting for Debug Ports) : http://jcjc-dev.com/2016/04/08/reversing-huawei-router-1-find-uart/

### [MIPS 기반의 IoT Device Hacking Bible]
위 두 URL은 개인적으로 MIPS 기반의 IoT Device의 Exploit을 위해 잘 설명된 문서라 생각하여, 참고하게 되었습니다.

- https://www.vantagepoint.sg/papers/MIPS-BOF-LyonYang-PUBLIC-FINAL.pdf
- https://www.exploit-db.com/docs/36806.pdf

### [ARM 기반의 Exoloit 기법 정리]
- Beginner's Guide to Exploitation on ARM : https://zygosec.com/Products/
- Very vulnerable ARM application (CTF style exploitation tutorial) : https://github.com/bkerler/exploit_me?fbclid=IwAR34sJuay_BY0mCOsPqrTIVgjBngdx6Ge8s0Ld2mqbFUVSu3bOzP1isf7uk


4. IoT Threat Analysis
MITRE ATT&CK : https://www.mitre.org/publications/technical-papers?fbclid=IwAR0CLO9qh6_mepxspFIxnsFs6ddKUmSOpJgXiPiFE0688H7K4owy7UiT0aw

Smart Grid Threat Modeling :
https://pure.qub.ac.uk/portal/files/140549879/2017_STRIDE_based_threat_modeling_for_cyber_physical_systems.pdf?fbclid=IwAR1EhWjQ9Csef_f3UGw8dHhvZTzbYslgiNl_5sfzSgDI5hC3adFKf_W6GlQ

IoT Security Design :
https://www.govinfosecurity.com/interviews/iot-moving-to-security-by-design-i-3860?fbclid=IwAR3cFHDAQsukDKnZk4RX9pShEMLF5BY-HiFuoFYd0zKRLVmxvpgA_xol4Xs#.WnAm-VNE_8w.facebook

Pentest Life Cycle :
https://blog.cobalt.io/pen-testing-as-a-service-life-cycle-87e122b0c61b?fbclid=IwAR36K2IFqE-dKySfEVpb8gwDdBz0_KcyX8g1GZFAmPij0_lbYdkyrc-oUpg

Getting Started with IoT Security with Threat Modeling :
https://www.denimgroup.com/resources/blog/2017/11/getting-started-with-iot-security-with-threat-modeling/?lipi=urn%3Ali%3Apage%3Ad_flagship3_pulse_read%3BtGhVz%2BNYQRiSgJC%2Bt%2FEWQQ%3D%3D&fbclid=IwAR1EgBfBAsz4628Kmuzr-qAR_vQ-KeaGe76fOXx6turl3DFPGRcs7mI5OzM
4. IoT 보안 이슈

TR-069 Protocol : https://sec-consult.com/en/blog/2018/05/tr-069-iot-before-it-was-cool/?fbclid=IwAR3wj1VspIVnEghYA-YRPcL6W77chsQT-xouradAFYzqqGjJuzRbIvjZu5k

Z-Wave Protocol :
https://securityaffairs.co/wordpress/72898/hacking/z-shave-z-wave-attack.html?fbclid=IwAR3VewvW-NIonp75AHNJYYq-5Ihs25t3GvSiklaXYEUCiFCxr_9mSB0gaVo

폴라로이드 카메라 해킹 사례 : https://mitxela.com/projects/thermal_paper_polaroid?fbclid=IwAR3WKb8mTq-DydMIbiRGrtFG7phreAWybugaTTCbcTy6JicBQjfJdgDFecM

Dasan Router Exploit :
https://isc.sans.edu/diary.html?fbclid=IwAR1rv6XHOd-6a8fvRX2cDh22KFMV4AkFwyxfdQ9XZoMIMh26uqZBFhq_iFQ

Hijacking Philips Hue :
https://www.pentestpartners.com/security-blog/hijacking-philips-hue/?fbclid=IwAR1rAEXh23kqCwRV_aqAPjEhnuC6gA6i5nO910cjOQgD7-MKrmRBGNiUyG8

GE Healthcare MAC 5500 Vulnerabilities :
https://www.atredis.com/blog/2018/5/14/ge-healthcare-mac-5500-vulnerabilities?fbclid=IwAR2gY9y2WSmHlDDNUs-tN_j6QjCrxFqZbpuDTY4peUTq5yxfz6fnQoaCC8E

1-day exploit development for Cisco IOS
https://media.ccc.de/v/34c3-8936-1-day_exploit_development_for_cisco_ios?fbclid=IwAR2nlM822vWYvTgCYiOGWECD0sInr6jQ5FDK9t3r26XTftuMQ9_D-zLf9wA

GoAhead that affects hundreds of thousands IoT devices
http://securityaffairs.co/wordpress/67113/iot/goahead-flaws.html?fbclid=IwAR1vx0qECDTJfFLSwgc-TVzWZT3tUzjqa4omtul8U-7s_PqlcZrrbWZctSk

REMOTE CODE EXECUTION (CVE-2017-13772) WALKTHROUGH ON A TP-LINK ROUTER :
MIPS 기반의 장비를 QEMU에 올려 취약점을 찾는 내용 Basic으로 좋음
https://fidusinfosec.com/tp-link-remote-code-execution-cve-2017-13772/?fbclid=IwAR3M8HQCxVV_mdh5CgVO6qcOfTwZHGvUkcqJfQAGyIw_zPZR9dkFOvb0GG8

HACK XIAOMI MI SMARTHOME :
http://faire-ca-soi-meme.fr/hack/2017/03/13/hack-xiaomi-mi-smarthome-zigbee-sniffer/?fbclid=IwAR0_cQj4zKPKre29gkOSDFlzYlbxKlkeNVAAp33q9WATieez_UW-iGLJFYw
5. IoT 보안 Tool
Attify OS : https://blog.attify.com/getting-started-with-firmware-emulation/?fbclid=IwAR3K1_P3N_OsQmObEcoY1JLr_vkGi0ae6CD5HeuvKY9nXkYhYlxxzd-VeLU

Kalirouter (TL;DR) :
https://github.com/koenbuyens/kalirouter?fbclid=IwAR2vutdRQZuXbPe80g_ERUNReC2L-nGUJnXLHqu76vXtNLgiDQKqDOlgq54

TCPDump Examples :
https://hackertarget.com/tcpdump-examples/?fbclid=IwAR0ubp6oiiToF_jC9em9Fl8wAI3_ovoihrM5WLRcvxALHey3BpR__RoS88w

ROUTERSPLOIT 3.0 :
https://www.threat9.com/blog.html?fbclid=IwAR39pKseDz0IJsz-1Jau5J0bFCuy_SntGYlnoufBD7a0ue5w3IXpJMHoebM

Firmware Analysis and Comparison Tool(FACT) :
https://n0where.net/the-firmware-analysis-and-comparison-tool-fact?fbclid=IwAR17LssLsEfwNnur14XhmCtwUJELLdlVYO_anmZXbpHTHF-U-9-mdK-Iu4g

Binwalk :
https://gbhackers.com/analyzing-embedded-files-and-executable-code-with-frimware-images-binwalk/?fbclid=IwAR2XI1wBJv6VBratTL5Te5Zq4YSEtL3hhQyBio2DTD2XdGLJa33eyPzozqc

ASTo - An IoT Network Security Analysis Tool and Visualizer :
https://www.kitploit.com/2017/07/asto-iot-network-security-analysis-tool.html?utm_source=dlvr.it&utm_medium=facebook&fbclid=IwAR3NEkJGl03gpMlivhOWKv53d9FrL9nJEJnQSXQcaR-UOywzKnYsGI1uGYI

Mipsdis: MIPS disassembler in the browser :
http://blog.loadzero.com/blog/announcing-mipsdis/?fbclid=IwAR0FExLw0fjCMIKIa8ePZEb2j3B1zZZQHz5IPbd26-dX5S8nqFrpxhuifJA
5. IoT Bonet 보고서
The Moon :
https://blog.netlab.360.com/file/TheMoon-botnet.pdf?fbclid=IwAR1qmdaMVChS6qLectoo5lwLk2kqWBSiRbEXLi2_YAvK9UjX15jgtEEa6Xs

DoubleDoor :
방화벽을 우회하는 기능을 탑재한 한국발
https://blog.newskysecurity.com/doubledoor-iot-botnet-bypasses-firewall-as-well-as-modem-security-using-two-backdoor-exploits-88457627306d?fbclid=IwAR0skVFTKZPZ-V0m5SC-NtogHD6P_72JXEUsxxuwlk8f_n2jc2RL-logm3k

HNS :
https://www.zdnet.com/article/this-unsuaul-new-iot-botnet-is-spreading-rapidly-by-using-peer-to-peer-communication/?fbclid=IwAR2LnBVVnzVQQ2lKBx2LW9tdourXcg7fIF7sFloCm-bUaYssTpoj4zeta5c

Satori :
Huawei Home Routers in Botnet Recruitment
https://research.checkpoint.com/good-zero-day-skiddie/?fbclid=IwAR3nzV1oxx3yM0Mvd_vH0JERUr6ixSPU_1wtqljKwp2fQjJfiUoD2VIWT0w

Persirai :
https://blog.trendmicro.com/trendlabs-security-intelligence/persirai-new-internet-things-iot-botnet-targets-ip-cameras/?fbclid=IwAR1JalIKaAJlW-6niRknOaSf-5CUwYkV7d5q6UShOh98M3z42Twwj0iBP-o

Hajime :
http://securityaffairs.co/wordpress/58415/malware/hajime-botnet.html?fbclid=IwAR35vEI_Il_qMgUASjiZtDzJ6nlcz6ZKnWBcQvOMK6P-Pr4lpMD-0zrgymg

brickerbot :
https://the01.jp/p0004886/?fbclid=IwAR0EzS6hAZbgZbjpVtnrvX1WoPQUuoBUhNups9K0moFskMLFe94P7SqHd_w

Amnesia :
https://unit42.paloaltonetworks.com/unit42-new-iotlinux-malware-targets-dvrs-forms-botnet/
# 1. IoT Device 서비스 파악
모든 사항은 쉘 접근이 가능하다고 하였을 때를 가장합니다.

## 1.1 CPU Architecture 확인

Memory corruption의 취약점을 찾기 위해서는 해당 IoT Device의 CPU(MCU)의 Architecture를 확인해야 한다.

```
cat /proc/cpuinfo
system type             : Ralink SoC
processor               : 0
cpu model               : MIPS 24K V4.12
```

위 명령어를 통해 해당 IoT Device는 MIPS 임을 확인 할 수 있다.

## 1.2 Endian 확인

Endian은 메모리 주소를 표현할 때 중요한 부분으로 대부분의 IoT Device에는 Endian을 확인 할 수 있는 명령어가 구성되어 있지 않다.
그렇기 때문에 펌웨어의 바이너리를 ubuntu에 옮겨 아래 명령어를 통해 Endian을 확인 한다.

```
persu@ubuntu:~/Desktop$ readelf -a [Daemon_name] |grep "endian"
Data:                              2's complement, little endian
```

명령어 결과를 통해 little endian 임을 확인하였다. endian을 확인 하는 방법은 다양하게 존재한다.

## 1.3 Target Daemon(service) 선정

대부분의 IoT Device의 Hacking은 원격지로 부터 특정 서비스에 접근하여 발생하게 된다.(IoT Device 뿐만 아니라 대부분...)
Linux 기반의 OS가 탑제된 IoT Device의 경우 유저/관리자가 IoT Device 관리를 용이하게 하기 위해 Web Service를 기본적으로 활성화 시킨다.

개인적으로 Target Daemon을 선정하기 위한 나름의 규칙은 아래와 같다.
- /etc/init.d/rcS 에 등록된 서비스 파악(부팅 시 제일 처음 실행되는 파일 : init, rcS)
- TCP/UDP 포트를 이용하여, 원격지에서 접근이 가능한 경우
- 특정 조건(설정 변경, Device 연동 시)이 되었을 경우 활성화 되는 서비스

위 규칙에 가장 만만하고 적합한 것이 HTTP(TCP 80) service다. 물론 해당 취약점도 Web Daemon 중 하나에서 발생하였다.

# 2. Web Daemon 분석

Web Daemon 분석 시 Memory corruption 취약점을 찾기 위해 필자는 두 가지 취약 함수에 대해 집중적으로 파악하였다.

- strcpy : 문자열을 복사하는 함수
- sprintf : 입력된 데이터를 버퍼로 출력하는 함수

두 함수는 길이 값 검증이 미흡할 시 Buffer over flow와 같은 취약점이 발생하는 함수이다.
