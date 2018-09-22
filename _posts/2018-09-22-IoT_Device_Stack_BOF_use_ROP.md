---
layout: post
title:  "MIPS based IoT Device Stack Buffer overflow"
tag:   Analysis
categories: persu IoT MIPS BOF ROP
modified: 2018-09-22
comments: true
featured: true
---

# 개요
해당 글은 MIPS 기반의 IoT Device에서 ROP를 이용한 Stack Buffer over flow 에 대한 내용입니다. 자세한 내용은 아래의 참고해 주시길 바랍니다.

[MIPS 기반의 IoT Device Hacking Bible]
- https://www.vantagepoint.sg/papers/MIPS-BOF-LyonYang-PUBLIC-FINAL.pdf
- https://www.exploit-db.com/docs/36806.pdf

위 두 URL은 개인적으로 MIPS 기반의 IoT Device의 Exploit을 위해 잘 설명된 문서라 생각하여, 참고하게 되었습니다.

이 문서는 제가 IoT Device를 실제로 분석하고 어떻게 Exploit을 하게 되었는지 설명하는 글입니다. vender 명 특정 String은 모자이크 처리를 하였으니, 참고 바랍니다.

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

아래 내용을 확인하면, websGetVar를 통해 입력 받은 page_type의 입력 값은 "00431B78" 의 sprintf의 $a2 레지스터가 된다.

```
.text:00431AE0 addiu   $a1, (aPage_type - 0x450000)  # "page_type"
.text:00431AE4 jalr    $t9 ; websGetVar

.text:00431B74 addiu   $a0, $sp, 0x18
.text:00431B78 jalr    $t9 ; sprintf    # ??
.text:00431B7C addiu   $a1, (aPostform_asp?f - 0x450000)  # "postform.asp?ftype=hns&fpage=%s&fcmd=de"...
.text:00431B80 lw      $gp, 0x10($sp)
.text:00431B84 move    $a0, $s1
.text:00431B88 la      $t9, websRedirect
.text:00431B8C nop
.text:00431B90 jalr    $t9 ; websRedirect
.text:00431B94 addiu   $a1, $sp, 0xF0+var_D8
.text:00431B98 lw      $gp, 0xF0+var_E0($sp)
.text:00431B9C lw      $ra, 0xF0+var_4($sp)
.text:00431BA0 lw      $s2, 0xF0+var_8($sp)
.text:00431BA4 lw      $s1, 0xF0+var_C($sp)
.text:00431BA8 lw      $s0, 0xF0+var_1
.text:00431BAC jr      $ra
.text:00431BB0 addiu   $sp, 0xF0
```

Stack Buffer 계산은 아래와 같이 간단한 파이썬 코드를 통해 할 수 있다.

```
python
hex(0xD8-0x1-len("postform.asp?ftype=commit&fpage=")) = '0xa8'
```
