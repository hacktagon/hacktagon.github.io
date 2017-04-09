---
layout: post
title: "Algorithm One Line Tips"
headline: Algorithm
modified: 2017-04-02
categories: Algorithm XothSec
comments: true
featured: true
---

### 최하위 비트 알아내기
A & (-A)

### 숫자가 2^n 인지 체크하는 법
A == (A & (-A)) -> 0은 예외 처리 해줘야함.

### 두 숫자가 해밍 Path (1bit 차이) 인지 체크하는 법
X = Code[i] ^ Code[j]
X!=0 && X==(X & (-X))
