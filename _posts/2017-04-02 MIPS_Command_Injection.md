---
layout: post
title:  "MIPS Command Injection"
tag:   Analysis
categories: mips reversing persu
modified: 2017-04-02
comments: true
featured: true
---

# MIPS Command Injection
MIPS 기반의 디바이스에서 command Injection을 찾는 나름의 방법을 정리해 보았습니다.

<b>Command Injection</b>은 사용자가 입력한 인자 값을 통해 명령어가 실행되는 취약점으로,
system, execv, ececle, ececve, execlp, ececvp, popen, open, fork 이런 함수에서 사용되게 됩니다.

MIPS는 IDA를 통해 Hexray(어셈블리어 -> C 코드 번역 기능)가 지원되지 않습니다.


- https://retdec.com
