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
MIPS 기반의 디바이스에서 command Injection을 찾는 나름의 방법을 정리해 보았다.

<b>Command Injection</b>은 사용자가 입력한 인자 값을 통해 명령어가 실행되는 취약점으로, system, execv, ececle, ececve, execlp, ececvp, popen, open, fork 이런 함수에서 사용되게 된다.

MIPS는 IDA를 통해 Hexray(어셈블리어 -> C 코드 번역 기능)가 지원되지 않는다.
최근 redtoc 플러그인 사용 시 일부 C 코드로 변환 할 수 있다. 아래 URL을 통해 확인 할 수 있으며, 바이너리 파일을 업로드하면 C 코드로 다운받을 수 있다.

- https://retdec.com


위 취약함수중 바이너리에 자주 사용되는 함수는 system, execl, popen, execv 순으로 등장하게 된다.

<b>이 블로그에서는 system, popen 함수에 대해서 command injection 방법을 포스팅한다.</b>

위에서 자주 사용되는 함수가 실제 데몬에서 어떻게 사용되는지 살펴 보겠다. 취약점 분석을 하다 보니 MIPS 어셈블리만이 가지고 있는 특정 규칙이 있다.

이것을 함수 호출 규약이라고 하죠! calling convention

 - 함수 호출 규약 1. 모든 함수의 인자 값은 a0, a1, a2, a3 레지스터가 입력됩니다. 함수의 인자 값이 4개를 초과할 경우는 임시 레지스터인 $t0, $t1 … 가 사용됩니다.
(현재까지 $t0 레지스터가 사용되는 경우는 보지 못함)
 - 함수 호출 규약 2. Jar opcode로 아래 한 줄을 실행시키고 자신을 실행 시키는 로직을 가지고 있습니다.
 - 함수 호출 규약 3. 함수가 실행되고 리턴 값은 $vx 레지스터에 저장된다.
 - 함수 호출 규약 4. ra 는 return address를 의미하고 있음

## 1. system 함수를 이용한 command injection
system 함수는 /bin/sh -c string를 호출하여 string에 지정된 명령어를 실행하고, 명령어가 끝난후 반환한다.

### 1.1 사용법

```
#include<stdlib.h>
int system(const char * string);
```

### 1.2 반환값

/bin/sh를 실행시키기 위한 execve()의 호출이 실패했다면 127이 리턴되며, 다른 에러가 있다면 -1, 그렇지 않다면 명령어의 리턴코드가 반환됩니다.

string값이 NULL이고, system()이 shell을 이용할 수 있다면 0이 아닌 값을 그렇지 않다면 0을 반환합니다.

[출처] https://www.joinc.co.kr/w/man/3/system

### 1.3 예제

MIPS 디바이스에 실제로 사용되는 system 함수는 3가지 방식이 있습니다. (어디까지나 개인적인 의견)

1.3.1 string 값이 Define되어 있는 경우

해당 경우 Command Injection이 발생 할 확률은 0%에 가깝다. 그 이유는 소스를 참고해보겠습니다.

```
#include<stdio.h>
#include<stdlib.h>

int main(void)
{
	system("/bin/sh");
	return 0;
}
```

string 값이 이미 system 함수 내 정의되어 있기 때문에 공격자가 변경을 할 수 없는 로직입니다. gdb를 이용하여 살펴 보도록 하겠습니다.
* 여기서는 MIPS 어셈블리의 에필로그, 프롤로그 등 기초적인 어셈블리 설명은 제외하겠습니다. 왜냐면 저도 아직 잘 모르거든요 ^-' (공부중)


```
0x00400650 <+16>:	lui	v0,0x40
0x00400654 <+20>:	addiu	a0,v0,2064
0x00400658 <+24>:	jal	0x400500 <system@plt>
0x0040065c <+28>:	move	at,at
```

main+24 부분에 system 함수가 실행되는데, 함수 호출 규약에서 system에 들어갈 인자 값인 string 은 $a0 레지스터에 저장이 됩니다.

```
(gdb) x/s $a0
0x400810:	 "/bin/sh"
```

이 경우에는 공격을 할 수 가 없겠죠?... 혹시 할 수 있는 방법이 있다면 알려주세요.

1.3.2 string 값이 spritf, snprintf를 통해 얻어 오는 경우

1.3.2 일 경우에 많은 취약점이 발견된다.

```
sprintf  : (char *buffer, const char *format, ...)
snprintf : (char *buffer, int buf_size, const char *format, ...)
```

sprintf, snprintf는 buffer 변수에 형식에 따라 만들어진 문자열이 저장된다.

```
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

int main(int argc, char *argv[])
{
        char input_buf[50];
        char command_buf[50];
        int snprintf_ret;

        if(argc < 2)
        {
                printf("argv error\n");
                exit(0);
        }
        strncpy(input_buf,argv[1],50);
        printf("command argv[1] : %s\n",input_buf);
        snprintf(command_buf,50,"cat %s",input_buf);
        printf("command_buf = %s\n\n",command_buf);
        system(command_buf);
        return 0;
}
```

간단한 소스를 짜보았는데 ... 우선 취약함수인 strcpy, sprintf는 사용하지 않고 오직 Command injection만 일어날 수 있는 소스 코드 인데..
arvg[1]로 입력한 인자 값이 snprintf 함수를 통해 cat argv[1] 형태로 command_buf로 변환이 되고 최종적으로 system 함수를 통해 command_buf 내 있는 string이 실행되게 된다. command injection이 일어나기 위해서는 \$ \` \&\& \|\| \; 이런 특수문자를 이용하여 하는 것은 기본 전제로 깔고 들어가겠다.

지금부터는 gdb를 통해 자세하게 파악해 보겠다.

```
0x004007dc <+140>:	addiu	a0,s8,80
0x004007e0 <+144>:	addiu	v0,s8,28
0x004007e4 <+148>:	li	a1,50
0x004007e8 <+152>:	move	a2,v1
0x004007ec <+156>:	move	a3,v0
0x004007f0 <+160>:	jal	0x400600 <snprintf@plt>
0x004007f4 <+164>:	move	at,at
0x004007f8 <+168>:	sw	v0,24(s8)
0x004007fc <+172>:	lui	v0,0x40
0x00400800 <+176>:	addiu	v1,v0,2572
0x00400804 <+180>:	addiu	v0,s8,80
0x00400808 <+184>:	move	a0,v1
0x0040080c <+188>:	move	a1,v0
0x00400810 <+192>:	jal	0x4005c0 <printf@plt>
0x00400814 <+196>:	move	at,at
0x00400818 <+200>:	addiu	v0,s8,80
0x0040081c <+204>:	move	a0,v0
0x00400820 <+208>:	jal	0x4005e0 <system@plt>
```

printf 함수와 strncpy 부분은 제외하였다.

snprintf 함수는 4개의 인자 값이 들어 간다. $a0,$a1,$a2,$a3 이 값이 순차적으로 스택에 쌓이게 될 것이며, 인자 값으로 들어가게 될 것이다.

```
(gdb) x/s $a0
0x7fff6c88:	 "w\374yp"
(gdb) x/s $a1
0x32:	 <Address 0x32 out of bounds>
(gdb) x/x $a1
0x32:	Cannot access memory at address 0x32
(gdb) x/s $a2
0x400a04:	 "cat %s"
(gdb) x/s $a3
0x7fff6c54:	 "1.c"
```

\$a 레지스터를 확인하면, 실제로 입력하였던 1.c 까지 잘 들어 가있는 것을 볼 수 있다. \$a2 레지스터는 0x32 값이 50임을 알 수 있다. 뭐... $a2에는 50이 들어가있다는 소리

그렇다면 breakpoint를 system 함수 쪽으로 옮기고 $a0에는 어떤 값이 들어가는지 확인해보자.

```
(gdb) x/s $a0
0x7fff6c88:	 "cat 1.c"
(gdb) n
#include<stdio.h>
#include<stdlib.h>
int main(void)
{
	system("/bin/sh");
	return 0;
}
```

\$a0 레지스터에는 snprintf의 결과 값인(리턴 값 아님) <b>cat 1.c</b> 가 들어가게 된다. system 함수는 이를 실행하여 1.c의 내용을 출력하게 된다.

이런 소스 코드 및 어셈블리는 command injection 취약점을 가지고 있다.
실제 테스트를 하기 위해 argv[1]에 1.c && ls 를 넣어 보겠다.

```
root@debian-mips:/home/user# ./2 1.c && ls
command argv[1] : 1.c
command_buf = cat 1.c

#include<stdio.h>
#include<stdlib.h>

int main(void)
{
	system("/bin/sh");
	return 0;
}

1  1.c	2  -2  2.c  mips
```

결과적으로는 command injection 성공이다. 실제로 명령어 삽입이 가능한 특수문자에 대한 필터링이 걸려 있지 않기 때문이다. gdb를 통해 확실히 파악해보겠다.

```
(gdb) x/s $a0
0x7fff6c78:	 "w\374yp"
(gdb) x/s $a1
0x32:	 <Address 0x32 out of bounds>
(gdb) x/s $a2
'0x400a04:	 "cat %s"
(gdb) x/s $a3
0x7fff6c44:	 "1.c && ls"
```

snprintf의 인자 값으로는 위 내용들이 포함되게 된다. 여기서 중요한 건 $a3 레지스터다.

```
(gdb) x/s $a0
0x7fff6c78:	 "cat 1.c && ls"
```

system 함수에서 breakpoint를 걸고 확인한 $a0 레지스터는 <b> cat 1.c && ls </b> command injection이 가능하게 되는 것이다.

1.3.3 string 값이 flash memory를 통해 얻어 오는 경우

system함수에 인가 값인 string이 위의 경우처럼 사용자 입력 값, 절대 값이 코드영역에서 처리되는 것이 아닌 flash memory에 저장되어 있는 string을 사용할 때가 있다.
이유는 MIPS를 쓰는 대부분의 디바이스들은 소형 임베디드 장비로 초기 설정 값들이 flash memory(비 휘발성 메모리 : NVRAM)에 저장되야 하기 때문이다. 초기 계정, 비밀번호, MAC주소, IP주소 등 ...

해당 경우에는 결국 flash memory에 있는 값들 또한 spritf, snprintf 함수에 사용되기 마련이다. 사용자가 접근 할 수 있는 웹 페이지, 포트 등 을 통해서 해당 flash memory 데이터가 변경이 가능하고 변경 가능한 데이터가 system 인자로 사용될 경우 취약점이 발생할 수 있다. 이 부분은 마땅한 예제를 들 수 가 없어 실제 장비 분석하면서 나오는 경우 추가하도록 하겠다.


## 2. popen 함수를 이용한 command injection

popen 은 command 를 shell(:12)을 가동시켜서 열고 pipe(2)로 연결한다. pipe 는 기본적으로 단방향으로만 정의 되어 있음으로, 읽기전용 혹은 쓰기전용 으로만 열수 있으며, type 로 정의된다. popen 은 command 를 실행시키고 pip 연결을 위해서 내부적으로 fork() 와 pipe() 를 사용한다.
command 는 실행쉘인 /bin/sh 에 -c 옵션을 사용하여서 전달되게 된다.
pclose(2) 함수는 종료되는 관련 프로세스를 기다리며 wait(2) 가 반환하는 것처럼 명령어의 종료 상태를 반환한다.

### 2.1 사용법

```
#include <stdio.h>

FILE *popen(const char *command, const char *type);
int pclose();
```

### 2.2 반환값

popen 은 실패할경우 NULL 을 반환한다. pclose 는 종료되는 관련 프로세스를 기다리며 명령어의 종료 상태를 반환한다. 에러가 발견될경우 -1 을 리턴한다.

### 2.3 예제

popen도 string 값이 정해져 있으냐, 사용자 입력 값으로부터 입력되느냐에 따라 나눠지는데
정해져 있는 경우 system 함수에서 언급하여 넘어가도록 하겠다.

```
int main(int argc, char *argv[])
{
	char input_buf[50];
	char snprintf_ret[50];

	snprintf(snprintf_ret,50,"cat %s",argv[1]);
	printf("snprintf_ret %s \n",snprintf_ret);
	FILE *fp;
	fp = popen(snprintf_ret,"r");

	if (fp == NULL)
	{
		perror("error : ");
		exit(0);
	}

	while(fgets(input_buf, 50, fp) != NULL)
	{
		printf("%s",input_buf);
	}
	pclose(fp);
}
```

위 예제 소스 코드는 snprintf를 통해 입력 받은 snprintf_ret 값이 popen 함수의 인자 값으로 사용되게 된다.
/bin/sh -c | cat 2.c 이런 형식으로 shell을 쓰게 되고 pipe를 쓰는건가?!
특이 하게 popen 일 경우 command injection에 사용되는 문구는 "command"로 입력 값을 줘야 한다.

좀더 상세하게 gdb로 살펴 보기로 하겠다.

```
0x004007cc <+44>:	addiu	a0,s8,80
0x004007d0 <+48>:	li	a1,50
0x004007d4 <+52>:	move	a2,v1
0x004007d8 <+56>:	move	a3,v0
0x004007dc <+60>:	jal	0x400640 <snprintf@plt>

(gdb) x/s $a0
0x7fff6c88:	 "w\374yp"
(gdb) x/s $a1
0x32:	 <Address 0x32 out of bounds>
(gdb) x/s $a2
0x400a40:	 "cat %s"
(gdb) x/s $a3
0x7fff6e9f:	 "2.c"
```

snprintf 부분에서는 인자 값이 정상적으로 들어가게 된다. 공격자가 입력한 값은 <b>2.c</b> cat 2.c 로 만들어진 값은 아래의 popen 함수에 입력된다.

```
0x00400800 <+96>:	addiu	v0,s8,80
0x00400804 <+100>:	move	a0,v0
0x00400808 <+104>:	lui	v0,0x40
0x0040080c <+108>:	addiu	a1,v0,2652
0x00400810 <+112>:	jal	0x400650 <popen@plt>

(gdb) x/s $a1
0x400a5c:	 "r"
(gdb) x/s $a0
0x7fff6c88:	 "cat 2.c"
```

popen(cat 2.c, r)로 함수가 실행되면 2.c를 읽어 온다.

지금 부터는 command injection을 넣어 보겠다.

큰 따옴표 방식이 아닌 그냥 command를 입력 값을 주었을 시 어떻게 되는지 보자.

```
(gdb) r 2.c;ls
Starting program: /home/user/3 2.c;ls

Breakpoint 1, 0x004007dc in main ()
(gdb) x/s $a0
0x7fff6c88:	 "w\374yp"
(gdb) x/s $a1
0x32:	 <Address 0x32 out of bounds>
(gdb) x/s $a2
0x400a40:	 "cat %s"
(gdb) x/s $a3
0x7fff6e9f:	 "2.c"
```

2.c;ls 중 2.c만 $a3레지스터에 남게 된다. 왜죠? ...무튼...

"command" 방식으로 입력 값을 주었을 시 아래 와 같다.

```
(gdb) r "2.c;ls"
Starting program: /home/user/3 2.c ; ls
Breakpoint 1, 0x004007dc in main ()
(gdb) x/s $a0
0x7fff6c78:	 "w\374yp"
(gdb) x/s $a1
0x32:	 <Address 0x32 out of bounds>
(gdb) x/s $a2
0x400a40:	 "cat %s"
(gdb) x/s $a3
0x7fff6e9c:	 "2.c;ls"
```

$a3 레지스터가 "2.c;ls"로 설정되어 있다.

```
0x00400800 <+96>:	addiu	v0,s8,80
0x00400804 <+100>:	move	a0,v0
0x00400808 <+104>:	lui	v0,0x40
0x0040080c <+108>:	addiu	a1,v0,2652
0x00400810 <+112>:	jal	0x400650 <popen@plt>
(gdb) x/s $a0
0x7fff6c78:	 "cat 2.c;ls"
(gdb) x/s $a1
0x400a5c:	 "r"
```

popen함수에 snpritf값인 cat 2.c;ls가 넘어오게 되고, command injection이 실행되게 된다! 야호! 성공 :)
