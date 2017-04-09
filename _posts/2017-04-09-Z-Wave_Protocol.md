---
layout: post
title:  "Wireless Protocol Z-Wave Analysis"
tag:   Analysis
categories: Wireless Analysis Z-Wave persu
modified: 2017-04-09
comments: true
featured: true
---

# Z-Wave Protocol Analysis

## 1. Z-Wave란 ?

Wireless IoT Protocol으로 저전력, 저대역폭을 지원하여, IoT 디바이스를 제어하기 위한 무선 프로토콜

<img src="{{ site.url }}/images/persu/zz4.jpg" style="display: block; margin: auto;">


### Z-Wave 규격 및 특징

- 대역폭 : 9,600 bit/s 또는 40 kbit/s
- 변조방식 : GFSK(Gaussian Frequency Shift Keying) : 주파수 편이 변조(Frequency Shift Keying, FSK) 방식의 하나로 가우시안 필터(Gaussian Filter)를 사용하여 이진수1은 양의 편향으로, 이진수0은 음의 편향으로 변환한다.
- 네트워크 구성 : 최대 232개의 유닛
- 네트워크 토폴로지 : Mesh(망형) -> 하나의 노드는 모든 노드의 상태를 가지게 된다.
- 주파수 대역 : 920.9 MHz, 921.7 MHz, 923.1 MHz
- <b>Non-IP Protocol</b>


## 2. 프로토콜 분석
Z-Wave 는 아래 계층 부터 RF, MAC, Transfer, Routing, Application 5계층으로 구성된다.

<img src="{{ site.url }}/images/persu/zz3.jpg" style="display: block; margin: auto;">


### 2.1 RF Media 계층
RF 계층은 네트워크 데이터를 신호화 시키는 계층으로 GFSK, FSK 등 과 같은 방식으로 신호를 생성하여 전달하는 계층이다.

<img src="{{ site.url }}/images/persu/zz1.jpg" style="display: block; margin: auto;">

### 2.2 MAC 계층
데이터 스트림은 맨처스터 코딩 방식으로 Start of Frame(SOF), End of Frame(EOF) 로 구성 된다.
해당 계층에서 가장 중요한 <b>Collision avoidance</b>를 담당한다.

- Collision avoidance
다른 노드가 전송되는 동안 전송하는 시작 노드와 충돌을 방지하는 회피기능을 가지게 된다. 노드들은 프레임을 전송하지 않을 때 수신 모드 다음 수신기에서 데이터를 파악한다.

<img src="{{ site.url }}/images/persu/zz2.jpg" style="display: block; margin: auto;">

### 2.3 Transfer 계층
두 노드 사이의 데이터 전송을 제어하며, 재전송 및 체크섬 확인 및 승인을 포함하는 계층이다. 이 계층에서는 4가지 Frame를 전송하게 된다.

- Singlecast Frame Type

항상 하나의 특정 노드로 데이터를 전송하는 방식이며, 수신되었다고 프레임이 인정된다.
singlecast의 프레임 전송이 제대로 이루어 지지 않았을 경우 재전송한다. 이는 ACK 신호를 수신하는데 걸리는 시간을 통해 재전송 여부를 판단하게 된다.

<img src="{{ site.url }}/images/persu/z1.jpg" style="display: block; margin: auto;">

- Transfer Acknowledge Frame Type

Singlecast에 대한 응답을 가지는 신호

<img src="{{ site.url }}/images/persu/z2.jpg" style="display: block; margin: auto;">

- Multicatst Frame Type
1 ~ 232 노드에 프레임을 송신하는 프레임 타입

- Broadcast Frame Type
모든 노드들에게 데이터를 보내는 방식

### 2.4 Routing 계층
이 계층에서는 Z-Wave의 Home Network에 라우팅 테이블과 네트워크 토폴로지를 스캔하고 유지하는데 책임이 있다. 이 계층에서는 두가지 프레임 구성을 가지게 된다.
- Routed Singlecast Frame Type
Flow방식으로 노드가 다른 노드에게 프레임을 넘기면서 확인 신호를 옮기는 방식이다.

- Routed Acknowledge Frame Type
페이로드가 없는 라우팅일 경우 singlecast로 전송된다.

<img src="{{ site.url }}/images/persu/z3.jpg" style="display: block; margin: auto;">

- Routing Table
컨트롤러는 네트워크 토폴로지에 대한 노드의 정보를 유지하는 위치 라우팅 테이블을 가진다.

<img src="{{ site.url }}/images/persu/z4.jpg" style="display: block; margin: auto;">

- Route to Node
Routing을 하기 위해서는 노드의 정보를 얻고, 최적의 라우트 프로토콜을 사용해야하는데 이에 관한 문제점을 가지고 있다.

### 2.5 Application 계층
응용 계층은 홈 ID, 노드 ID의 컨트롤러 및 복제의 할당이다. 응용 계층은 개발자가 특정하게 구현을 해야 한다.

|Command Class|Description|
|-|-|
|00h-1Fh|Reserved for the Z-Wave Protocol |
|20h-FFh|Reserved for the Z-Wave Application|

Application은 해당 IoT의 기능을 나타나며 거기에 들어가는 Value값은 Command parameter로 구성한다.

<img src="{{ site.url }}/images/persu/z5.jpg" style="display: block; margin: auto;">

## 2. Z-Wave 구성
Z-Wave는 Controller과 Slave 두개로 구성

2.1 Controller : 제어 명령을 Slave 노드로 전송하며, Slave 노드의 제어 명령을 수신하는 라우팅 테이블을 가짐 Controller은 하나의 네트워크를 구성하며, 모든 노드에 관한 네트워크 환경을 통해 라우팅 테이블을 가지게 된다. 또한 이러한 특징을 통해 항상 최신의 네트워크 상태를 알게 된다.

- Portable Controller
- Static Controller(동글?) : 상태 메시지를 파악, 모든 네트워크의 위치를 알 수 있는 장점(보조제어기?)
- Installer Controller
- Bridge Controller : NAT와 같은 128개 가상 노드와 홈 네트워크를 연결

2.2 Slave

- Slave : 컨트롤러부터 받은 명령을 직접 수행하는 노드로써 송신받은 데이텅에 한해서만 수신을 하는 기능을 가지고 다른 노드또는 컨트롤러에게 독자적으로 송신 할 순 없다.(오직 받은 명령에 대한 처리 후 응답)
- Routing Slave : 각각 노드가 가져야될 라우팅 경로 번호에 대해서 정보를 전송하게 된다.
- Enhanced Slave(향상된 슬레이브) : 실시간 클록 및 애플리케이션 데이터를 저장하기 위한 EEPROM을 가지고 있다.


## 3. Z-Wave 식별
- Home ID : IP주소를 통해서 네트워크를 분리하는 TCP/IP체계와는 다르게 Z-Wave에서는 Home ID라는 식별자를 통해 네트워크를 분리하게 된다.(32bit/4byte)
- Node ID : Home ID를 가진 네트워크에서 각 노드들에게 부여되는 값이다. 8bit(1byte)로 구성되는 값

## 4. Z-Wave 동작 과정
Z-Wave 동작과정은 HomeID 를 가지고 있는 Master가 slave에게 HomeID, NodeID를 부여하는 Pairing 과정에서 부터 시작된다.

아래 그림과 같이 Master는 Slave를 찾기 위해 Broadcast를 통해 자신을 알리게 된다.
<img src="{{ site.url }}/images/persu/zz5.jpg" style="display: block; margin: auto;">

이후 Pairing 대상인 Slave는 여기서 응답을 주게 된다.
<img src="{{ site.url }}/images/persu/zz6.jpg" style="display: block; margin: auto;">

Master는 HomeID에 관한 NodeID를 부여하게 되는데, 이로써 Slave는 HomeID를 가진 디바이스간 통신을 할 수 있다.
<img src="{{ site.url }}/images/persu/zz7.jpg" style="display: block; margin: auto;">

이런 연결 과정이 끝나게 되면, Slave는 Master에게 자신의 정보를 전달하게 되고 Slave가 사용할 수 있는 명령 체계를 전달하게 된다.
<img src="{{ site.url }}/images/persu/zz8.jpg" style="display: block; margin: auto;">

이후에는 Master가 HomeID, NodeID 에 맞게 명령을 수행하게 된다.
