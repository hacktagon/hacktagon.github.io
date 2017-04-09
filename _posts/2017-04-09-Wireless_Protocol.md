---
layout: post
title:  "Wireless Protocol 802.11 Analysis"
tag:   Analysis
categories: Wireless Analysis persu
modified: 2017-04-09
comments: true
featured: true
---


# Wireless Protocol 802.11 Analysis

- 유선으로 이루어진 과거 네트워크 시스템에는 기기간의 유선의 한계를 뛰어넘기 위해 무선 기술이 개발되었고, 그에 따라 개발된 근거리 통신망을 구축하기 위한 무선 네트워크에 대한 표준이 802.11 규격이다.



## 1. 이동통신(LTE, 3G) vs 무선프로토콜(Wi-Fi)

- 이동통신은 무선 프로토콜은 동일한 무선 통신을 하지만 왜 나눠서 구별을 할까 ?

이유는 이동성(Mobility) 에 이유가 있다. 무선 프로토콜 제작 당시에는 무선 기기 들의 동작 반경은 한 사무실 내에 정도였지만 이동통신은 휴대전화가 생기면서 발전하였고 통화 중인 사용자는 기지국 내 반경 (Cell) 을 벗어 나더라도 통화를 유지해야하기 때문에 (이러한 기술을 Hand off / Hand over 이라고 한다.) 이동통신을 구성하는 프로토콜 스택 구조가 복잡하게 된다.

- 어쩌면 이러한 이동통신 프로토콜 스택 구조 덕분에 베터리 소모가 많이 들지 않을까 ?

실제로 무선 프로토콜 보다 이동통신이 베터리 소모를 많이 잡아 먹는 이유는 프로토콜 복잡성에 대한 것도 있지만 이동통신은 기지국 내에서 멀리 있을 수록 신호의 세기를 위해 고 전력을 사용한다.

<img src="{{ site.url }}/images/persu/0.jpg" style="display: block; margin: auto;">

## 2. 802.11 무선 프로토콜 용어

네트워크 구축 방식에는 Infra structured, non-infra structured 방식이 있다. 아래 그림만 봐도 알 수 있듯이 구조적인 특징을 가지고 있는 방식이 infra 구조 방식이지만 ad-hoc 같이 무선 디바이스 간의 통신을 하는 구조를 non-infra 구조 방식이라고 한다.

<img src="{{ site.url }}/images/persu/1.jpg" style="display: block; margin: auto;">

- AP VS ad-hoc

AP 방식은 무선 기기들을 인터넷 통신을 하기 위해 Gateway 역활을 하는 기기이다. 외국에서는 홈 라우터라고도 불리며 실제로 사설망(사설 IP) 로 구성되며 외부 네트워크로 통신을 할때는 NAT 를 통해 통신을 한다. 하지만 ad-hoc 같은 방식은 무선 기기간의 릴레이를 해주는 역활을 하는 구조다. 릴레이란 서로 같은 네트워크 안에 있는 기기들간의 통신을 하기위해 도움을 주는 기술이다. (리피터, 브릿지)


## 3. CSMA/CA (Carrier-sense Multi-access with / Collision Avoidance)

버스 구조의 이더넷 방식은 한 PC 가 데이터 전송시 같은 네트워크에 있는 PC들은 통신을 하지 않는다. 네트워크의 신호를 통해 '아 다른 PC가 사용 중이구나' 라고 알 수 있기 때문이다. 하지만 두 PC 가 우연하게도 동시에 신호 전송을 발생 시키면 어느 한 부분에서 충돌하여 잡음이 되어 두 PC는 서로 다른 시간 값을 가지고 우선 순위를 기다리게 된다. 이러한 방식을 CSMA/CD (D : Detect) 이라고 한다. 즉 서로 충돌하는 신호를 통해 판단을 할 수 있다는 것이다.

하지만 무선에는 D 가 아닌 A 를 쓴다. 회피인 것이다. 이 CSMA/CA 방식은 4 단계로 구성 되는데

1. 송신단 -> 수신단 RTS(Request to send)

2. 송신단 <- 수신단 CTS(Clear to send)

3. 송신단 -> 수신단 Data 전송

4. 송신단 -> 수신단 ACK

이와 같은 방식을 사용하며 CA 방식의 최대 목적은 Data 전송에 대한 충돌을 회피 하겠다는 목적이 있다. 아래 그림 처럼 AP 반경에는 A, B 스테이션이 존재하지만 A의 반경에는 B의 존재를 모른다. B 역시 마찬가지이다. 이러한 문제를 hidden node problem 이라고 하며 이 문제 때문에 송신단(Station) 은 수신단(AP) 에게 RTS (Request to Send) 메세지를 보냄으로써 연결 의사를 묻는 것이다. CTS(Clear to send) 역시 마찬가지이다. 수신단은 현재 다른 송신단과 통신하지 않고 있다 라는 메세지를 보냄으로 RTS 를 보낸 송신자와 수신자만의 통신이 이뤄지며 CTS 메세지는 모든 송신단에게 알려지기 때문에 NAV Time 동안은 다른 송신자는 No carrier sensing 을 하게 되면서 프로세스 사용을 줄일 수 가 있다. 이러한 방식이면 데이터는 충돌을 최소화(회피) 하면서 전송될 수 있다는 장점을 가진다.

<img src="{{ site.url }}/images/persu/2.jpg" style="display: block; margin: auto;">



## 4. 802.11 무선 프로토콜의 동작 방식

### 4.1 Passive mode

1. 첫 번째 동작 : Beacon Frme Broad cast
- Station <- AP Beacon Frame



2. 두 번째 동작 : Auth
- Station -> AP Authentication Request
- Station <- AP Authentication Response



3. 세 번째 동작 : Association
- Station -> AP Association Request
- Station <- AP Association Response

우리가 평소 Wi-Fi 를 사용 하는 모습이다. AP 를 찾고 인증을 거치는것까지 우리는 알고 있다. 하지만 3번 Association(결합) 과정은 우리가 잘 알지 못한다. 3번 과정이 있는 이유는 무엇일까 ?

즉 인증을 받은 AP 와 1:1 을 통신을 하기 위함이다. 만약 여러 인증을 거친 AP 와 통신을 하게 된다고 생각하면 보내고자 하는 데이터는 최종 목적지에 도착하면 연결한 AP 만큼 데이터가 도착할 것이며 최종 목적지에서 데이터를 보내려고 할때는 어느 AP를 목적지로 해서 보내야 할지 의문이 생기기 때문이다. 그렇기 때문에 BSSID 라는 AP의 MAC 주소를 통해 결합과정이 이루어지며 AP 와 Station 간의 1:1 통신을 하게 된다.

if 데이터를 전송 후 응답이 오기 전에 연결된 AP와 멀어져 다른 AP와 접속을 하게 된다면 ?

이러한 경우에는 응답 메세지는 드롭이 되며 Station 은 다른 AP 와의 통신을 통해 다시 데이터를 목적지에게 전송하게 된다.

### 4.2 Active mode

1. 첫 번째 동작 : Probe 메세지
- Station -> Probe Request
- Station <- Probe Response



2. 두 번째 동작 : Auth
- Station -> AP Authentication Request
- Station <- AP Authentication Response

3. 세 번째 동작 : Association
- Station -> AP Association Request
- Station <- AP Association Response

Passive mode 와 다른점은 station 이 먼저 AP 와의 연결을 묻는 것이다. Passive mode 의 Beacon Frame 주기는 3초~5초 정도 되지만 이러한 Active 모드는 이러한 주기 보다 빨리 연결을 하기 위해 존재한다. 이유는 컴퓨터의 입장에서는 1초가 상당한 시간이며 시간을 단축 시키기 위함이라고 생각한다.

## 5. 802.11 네트워크 동작

802.11 네트워크 동작으로는 9가지의 동작 방식이 있다. 이런 많은 프로토콜 동작이 있는 이유는 무선이라는 특징을 가지기 때문이다. 1,2 계층으로 구성된 802.11 프로토콜은 오류 제어를 해야한다. (마치 TCP 처럼...)

분산, 통합, 결합, 재결합, 결합해제, 인증, 인증해제, 프라이버시, 데이터전달 <- 총 9가지 해당 9가지 동작방식은 802.11 Protocol Header 부분에서 Type Header에 따라 달라지게 된다.

<img src="{{ site.url }}/images/persu/3.jpg" style="display: block; margin: auto;">

- 출처 : http://www.wildpackets.com/resources/compendium/wireless_lan/wlan_packets

802.11 MAC Header 부분에서 Frame Control의 Type, Subtype 부분을 통해 Frame의 동작을 파악 할 수 있다.
