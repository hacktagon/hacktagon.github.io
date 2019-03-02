---
layout: post
title:  "Almost of IoT Security"
tag:   Analysis
categories: persu IoT
modified: 2019-02-06
comments: true
featured: true
---

# 개요 (2019.02.07 v1.0)
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

- IoT 공통보안원칙 :https://www.kisa.or.kr/public/laws/laws3_View.jsp?cPage=2&mode=view&p_No=259&b_No=259&d_No=67&ST=T&SV=
- IoT 공통보안 가이드 : https://www.kisa.or.kr/public/laws/laws3_View.jsp?cPage=1&mode=view&p_No=259&b_No=259&d_No=80&ST=&SV=
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
- IoT에 대한 Security & Privacy Guidelines : https://www.schneier.com/blog/archives/2017/02/security_and_pr.html?fbclid=IwAR02gbKHU0tZCJG1AQ21q-1ryfi-jGnesceJha7RUgppNHf86JsgjZmQBxw

## 3. IoT 보안 입문 안내서
아래 URL은 IoT 해킹을 하기 위해 필요한 지식을 잘 정리한 자료입니다. 요즘은 쉽게 찾을 수 있는 내용이지만, 당시 작성했을 작성자분들에게 감사에 인사를 드립니다!

### [하드웨어 해킹]
- GrayHash 정구홍 수석님 자료 :
https://www.hackerschool.org/HardwareHacking/?fbclid=IwAR2Edhs8Fuml72N9gugJRaI5__TyE5GJ8Fy9TM7x4mkmcFbJmulg3Jbwe3k
- The JTAG Interface: An Attacker's Perspective : https://optivstorage.blob.core.windows.net/web/file/55e86eae3f04450d9bafcbb3a94559ca/JTAG.Whitepaper.pdf?fbclid=IwAR2M8NVJknkM8NpuiE6lMU-RcMKtMf5ejy4JOvitRiiYjLvWf4y9ugE3Pag
- Practical Reverse Engineering Part 1 (Hunting for Debug Ports) : http://jcjc-dev.com/2016/04/08/reversing-huawei-router-1-find-uart/

### [MIPS 기반의 IoT Device Hacking Bible]
아래 두 URL은 개인적으로 MIPS 기반의 IoT Device의 Exploit을 위해 잘 설명된 문서라 생각하여, 참고하게 되었습니다.
- https://www.vantagepoint.sg/papers/MIPS-BOF-LyonYang-PUBLIC-FINAL.pdf
- https://www.exploit-db.com/docs/36806.pdf

### [ARM 기반의 Exoloit 기법 정리]
- Beginner's Guide to Exploitation on ARM : https://zygosec.com/Products/
- Very vulnerable ARM application (CTF style exploitation tutorial) : https://github.com/bkerler/exploit_me?fbclid=IwAR34sJuay_BY0mCOsPqrTIVgjBngdx6Ge8s0Ld2mqbFUVSu3bOzP1isf7uk


## 4. IoT Threat Analysis
위협 분석은 모든 Pentest와  Vuln Assessment 를 진행하기 전 반드시 필요한 Task 입니다. IoT 에 대해 모의해킹이나 버그바운티 하시는 분들의 대부분은 IoT 디바이스에 대해 초점을 맞추고 있습니다. IoT 는 디바이스로만 구성되지 않고 다양한 서버, App 등 구성요소를 가지고 있는 특징을 가지고 있습니다. 그렇기 때문에 정확한 서비스 파악을 통해서 많은 위협을 분석해야 하는 필요성이 있습니다.

- MITRE ATT&CK : https://www.mitre.org/publications/technical-papers?fbclid=IwAR0CLO9qh6_mepxspFIxnsFs6ddKUmSOpJgXiPiFE0688H7K4owy7UiT0aw
- Smart Grid Threat Modeling :
https://pure.qub.ac.uk/portal/files/140549879/2017_STRIDE_based_threat_modeling_for_cyber_physical_systems.pdf?fbclid=IwAR1EhWjQ9Csef_f3UGw8dHhvZTzbYslgiNl_5sfzSgDI5hC3adFKf_W6GlQ
- IoT Security Design :
https://www.govinfosecurity.com/interviews/iot-moving-to-security-by-design-i-3860?fbclid=IwAR3cFHDAQsukDKnZk4RX9pShEMLF5BY-HiFuoFYd0zKRLVmxvpgA_xol4Xs#.WnAm-VNE_8w.facebook
- Pentest Life Cycle :
https://blog.cobalt.io/pen-testing-as-a-service-life-cycle-87e122b0c61b?fbclid=IwAR36K2IFqE-dKySfEVpb8gwDdBz0_KcyX8g1GZFAmPij0_lbYdkyrc-oUpg
- Getting Started with IoT Security with Threat Modeling :
https://www.denimgroup.com/resources/blog/2017/11/getting-started-with-iot-security-with-threat-modeling/?lipi=urn%3Ali%3Apage%3Ad_flagship3_pulse_read%3BtGhVz%2BNYQRiSgJC%2Bt%2FEWQQ%3D%3D&fbclid=IwAR1EgBfBAsz4628Kmuzr-qAR_vQ-KeaGe76fOXx6turl3DFPGRcs7mI5OzM


## 5. IoT 보안 / 해킹 사례
요즘은 구글링 조금만하면 IoT 디바이스 해킹 사례를 다수 확인 할 수 있습니다. 아래 URL은 다양한 공격 벡터를 가진 사례들입니다.

- TR-069 Protocol : https://sec-consult.com/en/blog/2018/05/tr-069-iot-before-it-was-cool/?fbclid=IwAR3wj1VspIVnEghYA-YRPcL6W77chsQT-xouradAFYzqqGjJuzRbIvjZu5k
- Z-Wave Protocol :
https://securityaffairs.co/wordpress/72898/hacking/z-shave-z-wave-attack.html?fbclid=IwAR3VewvW-NIonp75AHNJYYq-5Ihs25t3GvSiklaXYEUCiFCxr_9mSB0gaVo
- 폴라로이드 카메라 해킹 사례 : https://mitxela.com/projects/thermal_paper_polaroid?fbclid=IwAR3WKb8mTq-DydMIbiRGrtFG7phreAWybugaTTCbcTy6JicBQjfJdgDFecM
- Dasan Router Exploit :
https://isc.sans.edu/diary.html?fbclid=IwAR1rv6XHOd-6a8fvRX2cDh22KFMV4AkFwyxfdQ9XZoMIMh26uqZBFhq_iFQ
- Hijacking Philips Hue :
https://www.pentestpartners.com/security-blog/hijacking-philips-hue/?fbclid=IwAR1rAEXh23kqCwRV_aqAPjEhnuC6gA6i5nO910cjOQgD7-MKrmRBGNiUyG8
- GE Healthcare MAC 5500 Vulnerabilities :
https://www.atredis.com/blog/2018/5/14/ge-healthcare-mac-5500-vulnerabilities?fbclid=IwAR2gY9y2WSmHlDDNUs-tN_j6QjCrxFqZbpuDTY4peUTq5yxfz6fnQoaCC8E
- 1-day exploit development for Cisco IOS
https://media.ccc.de/v/34c3-8936-1-day_exploit_development_for_cisco_ios?fbclid=IwAR2nlM822vWYvTgCYiOGWECD0sInr6jQ5FDK9t3r26XTftuMQ9_D-zLf9wA
- GoAhead that affects hundreds of thousands IoT devices
http://securityaffairs.co/wordpress/67113/iot/goahead-flaws.html?fbclid=IwAR1vx0qECDTJfFLSwgc-TVzWZT3tUzjqa4omtul8U-7s_PqlcZrrbWZctSk
- REMOTE CODE EXECUTION (CVE-2017-13772) WALKTHROUGH ON A TP-LINK ROUTER (MIPS 기반의 장비를 QEMU에 올려 취약점을 찾는 내용 Basic으로 좋음) : https://fidusinfosec.com/tp-link-remote-code-execution-cve-2017-13772/?fbclid=IwAR3M8HQCxVV_mdh5CgVO6qcOfTwZHGvUkcqJfQAGyIw_zPZR9dkFOvb0GG8
- HACK XIAOMI MI SMARTHOME : http://faire-ca-soi-meme.fr/hack/2017/03/13/hack-xiaomi-mi-smarthome-zigbee-sniffer/?fbclid=IwAR0_cQj4zKPKre29gkOSDFlzYlbxKlkeNVAAp33q9WATieez_UW-iGLJFYw

## 6. IoT 보안 Tool
제가 사용하는 최애 Tool 은 Attify OS 입니다. IoT 해킹을 하기 위한 툴이 자동으로 인스톨되어 있고, 유지도 잘되어 있죠. 그 외에는 Qemu, burp, iptables, tcpdump 등입니다. 또한 static compile 된 busybox, tcpdump, gdbserver 등이 필요한 경우가 있는데 구글링 혹은 아래 url을 통해 다운로드 받으시길 바랍니다.

- static compile tools : https://github.com/hacktagon/embed_tools
- Attify OS : https://blog.attify.com/getting-started-with-firmware-emulation/?fbclid=IwAR3K1_P3N_OsQmObEcoY1JLr_vkGi0ae6CD5HeuvKY9nXkYhYlxxzd-VeLU
- Kalirouter : https://github.com/koenbuyens/kalirouter?fbclid=IwAR2vutdRQZuXbPe80g_ERUNReC2L-nGUJnXLHqu76vXtNLgiDQKqDOlgq54
- TCPDump Examples : https://hackertarget.com/tcpdump-examples/?fbclid=IwAR0ubp6oiiToF_jC9em9Fl8wAI3_ovoihrM5WLRcvxALHey3BpR__RoS88w
- ROUTERSPLOIT 3.0 : https://www.threat9.com/blog.html?fbclid=IwAR39pKseDz0IJsz-1Jau5J0bFCuy_SntGYlnoufBD7a0ue5w3IXpJMHoebM
- Firmware Analysis and Comparison Tool(FACT) : https://n0where.net/the-firmware-analysis-and-comparison-tool-fact?fbclid=IwAR17LssLsEfwNnur14XhmCtwUJELLdlVYO_anmZXbpHTHF-U-9-mdK-Iu4g
- Binwalk : https://gbhackers.com/analyzing-embedded-files-and-executable-code-with-frimware-images-binwalk/?fbclid=IwAR2XI1wBJv6VBratTL5Te5Zq4YSEtL3hhQyBio2DTD2XdGLJa33eyPzozqc
- ASTo (An IoT Network Security Analysis Tool and Visualizer) : https://www.kitploit.com/2017/07/asto-iot-network-security-analysis-tool.html?utm_source=dlvr.it&utm_medium=facebook&fbclid=IwAR3NEkJGl03gpMlivhOWKv53d9FrL9nJEJnQSXQcaR-UOywzKnYsGI1uGYI
- Mipsdis: MIPS disassembler in the browser : http://blog.loadzero.com/blog/announcing-mipsdis/?fbclid=IwAR0FExLw0fjCMIKIa8ePZEb2j3B1zZZQHz5IPbd26-dX5S8nqFrpxhuifJA

## 7. IoT Bonet Malware 보고서
Mirai 이후에 IoT에 다양한 Botnet 들이 발견되고 있습니다. (사실 Mirai 이전에도 많았습니다;) Mirai 외 다양한 IoT botnet 들을 정리한 자료이며, 연구용? 으로 샘플이 필요하신분은 직접 구하시길 바랍니다 :)

- The Moon : https://blog.netlab.360.com/file/TheMoon-botnet.pdf?fbclid=IwAR1qmdaMVChS6qLectoo5lwLk2kqWBSiRbEXLi2_YAvK9UjX15jgtEEa6Xs
- DoubleDoor(방화벽을 우회하는 기능을 탑재한 한국발) : https://blog.newskysecurity.com/doubledoor-iot-botnet-bypasses-firewall-as-well-as-modem-security-using-two-backdoor-exploits-88457627306d?fbclid=IwAR0skVFTKZPZ-V0m5SC-NtogHD6P_72JXEUsxxuwlk8f_n2jc2RL-logm3k
- HNS : https://www.zdnet.com/article/this-unsuaul-new-iot-botnet-is-spreading-rapidly-by-using-peer-to-peer-communication/?fbclid=IwAR2LnBVVnzVQQ2lKBx2LW9tdourXcg7fIF7sFloCm-bUaYssTpoj4zeta5c
- Satori(Huawei Home Routers in Botnet Recruitment) : https://research.checkpoint.com/good-zero-day-skiddie/?fbclid=IwAR3nzV1oxx3yM0Mvd_vH0JERUr6ixSPU_1wtqljKwp2fQjJfiUoD2VIWT0w
- Persirai : https://blog.trendmicro.com/trendlabs-security-intelligence/persirai-new-internet-things-iot-botnet-targets-ip-cameras/?fbclid=IwAR1JalIKaAJlW-6niRknOaSf-5CUwYkV7d5q6UShOh98M3z42Twwj0iBP-o
- Hajime : http://securityaffairs.co/wordpress/58415/malware/hajime-botnet.html?fbclid=IwAR35vEI_Il_qMgUASjiZtDzJ6nlcz6ZKnWBcQvOMK6P-Pr4lpMD-0zrgymg
- brickerbot : https://the01.jp/p0004886/?fbclid=IwAR0EzS6hAZbgZbjpVtnrvX1WoPQUuoBUhNups9K0moFskMLFe94P7SqHd_w
- Amnesia : https://unit42.paloaltonetworks.com/unit42-new-iotlinux-malware-targets-dvrs-forms-botnet/

### 앞으로 해당 게시글은 좋은 자료가 수집될 시 틈틈히 업데이트 하도록 하겠습니다.
