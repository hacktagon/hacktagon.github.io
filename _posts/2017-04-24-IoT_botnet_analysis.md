---
layout: post
title:  "IoT Botnet Bricker_Bot"
tag:   Analysis
categories: IoT Analysis bot persu
modified: 2017-04-26
comments: true
featured: true
---

## Bricker botnet


본 글은 최근 이슈가 되고 있는 IoT Botnet인 Bricker에 대해 조사해 보았다.

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
BrickerBot 감염을 막기 위해서는 가장 기본적인 대응방안이 필요하다. 각 디바이스는 Telnet, SSH와 같은 원격 단말 관리 서비스를 비활성화 시켜야 하며, 필요에 의해 활성화 시킬 경우 각 디바이스 별로 유니크한 패스워드 정책을 수립해야 한다. 특히 SSH 사용 시 로그인 방식이 아닌 키교환을 통해 로그인할 수 있도록 구성헤야 한다.
