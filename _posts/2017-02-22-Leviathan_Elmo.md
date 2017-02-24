---
layout: post
title: "엘모쨩의 Leviathan 롸업☆"
headline: Leviathan
modified: 2017-02-23
categories: Wargame Elmo leviathan
comments: true
featured: true
---

# leviathan0

``` bash
leviathan0@melinda:~$ ls -al
total 24
drwxr-xr-x   3 root       root       4096 Nov 14  2014 .
drwxr-xr-x 172 root       root       4096 Jul 10  2016 ..
drwxr-x---   2 leviathan1 leviathan0 4096 Dec 12 15:03 .backup
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
leviathan0@melinda:~$ cat .backup/
cat: .backup/: Is a directory
leviathan0@melinda:~$ cd .backup/
leviathan0@melinda:~/.backup$ ls -al
total 140
drwxr-x--- 2 leviathan1 leviathan0   4096 Dec 12 15:03 .
drwxr-xr-x 3 root       root         4096 Nov 14  2014 ..
-rw-r----- 1 leviathan1 leviathan0 133259 Nov 14  2014 bookmarks.html
```

## solution

``` bash
leviathan0@melinda:~/.backup$ vi bookmarks.html

/leviathan

<DT><A HREF="http://leviathan.labs.overthewire.org/passwordus.html | This will be fixed later, the password for leviathan1 is rioGegei8m" ADD_DATE="1155384634" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">password to leviathan1</A>
```

## flag

```
rioGegei8m
```

# leviathan1

``` bash
leviathan1@melinda:~$ ls -al
total 28
drwxr-xr-x   2 root       root       4096 Nov 14  2014 .
drwxr-xr-x 172 root       root       4096 Jul 10  2016 ..
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
-r-sr-x---   1 leviathan2 leviathan1 7493 Nov 14  2014 check
leviathan1@melinda:~$ file check
check: setuid ELF 32-bit LSB  executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0d17ae20f672ebc7d440bb4562277561cc60f2d0, not stripped
leviathan1@melinda:~$ cat check                                                             E® 񳬇Ӏ»Ebuaˠ䳤°,,  lib/ld K㿆07?_ M2GNUGNU
                                  libc.so.6_IO_stdin_usedputs__stack_chk_failprintfgetcharsysxiistrcmp__libc_start_main__gmon_start__GLIBC_2.4GLIBC_2.0ii
 $雁»𿆀t.[Ŀ5ÿÿ%
              h꡿ÿÿÿ%ꑿÿÿÿ%hꁿÿÿÿ%h鱿ÿÿÿ%h 顿ÿÿÿ% h(鑿ÿÿÿ%$h0避ÿÿÿ%(h8豿ÿÿ1잉ᄤ󏓒hQVh鐿ÿÿ󦏦fffff$¦fffff¸7-4񵂃¸󿿴񕊥ꙇ$4ÿщÍ¶¸4-4¸Á蝁ё󂃺Ѵ񕊥ꘉD$ń$4ÿӉÉ򍺧=4uU.罿ÿÿĴʳ¦󿿴¸󿿴U儬ńÿщ躿ÿÿ贿ÿÿU儤䪰e¡D$,1GD$sexƄ$%secrfƄ$)etń$+Ƅ$godƄ$ loveń$$ń炾ÿÿ獾ÿÿD$焾ÿÿD$滾ÿÿD$ń$D$D$D$$龽ÿÿ󿿵ń澾ÿÿ
                                                                      셄栾ÿÿ¸T$,e3t鹽ÿÿʃfffUW1ÿVS禾ÿÿZ꜋l$0³
鴽ÿÿ[Cpassword: /bin/shWrong password, Good Bye ...(񾀿D}þÿÿh@ÿÿÿ°ÿÿÿzR|
```

아이고 이상하다 'ㅅ'
그래도 어쨌든 실행을 해보쟈

``` bash
leviathan1@melinda:~$ ./check
password: rioGegei8m
Wrong password, Good Bye ...
```

웅 굳베이...

## gdb
``` gdb
   0x0804857a <+77>:   call   0x80483c0 <printf@plt>
   0x0804857f <+82>:   call   0x80483d0 <getchar@plt>
   0x08048584 <+87>:   mov    BYTE PTR [esp+0x14],al
   0x08048588 <+91>:   call   0x80483d0 <getchar@plt>
   0x0804858d <+96>:   mov    BYTE PTR [esp+0x15],al
   0x08048591 <+100>:   call   0x80483d0 <getchar@plt>
   0x08048596 <+105>:   mov    BYTE PTR [esp+0x16],al
   0x0804859a <+109>:   mov    BYTE PTR [esp+0x17],0x0
   0x0804859f <+114>:   lea    eax,[esp+0x18]
   0x080485a3 <+118>:   mov    DWORD PTR [esp+0x4],eax
   0x080485a7 <+122>:   lea    eax,[esp+0x14]
   0x080485ab <+126>:   mov    DWORD PTR [esp],eax
   0x080485ae <+129>:   call   0x80483b0 <strcmp@plt>

Make breakpoint pending on future shared library load? (y or [n]) n
(gdb) b *main+129
Breakpoint 1 at 0x80485ae
(gdb) r
Starting program: /home/leviathan1/check
password: AAA

Breakpoint 1, 0x080485ae in main ()

(gdb) x/wx $esp
0xffffd690:   0xffffd6a4
(gdb)
0xffffd694:   0xffffd6a8
(gdb) x/s 0xffffd6a4
0xffffd6a4:   "AAA"
(gdb) x/s 0xffffd6a8
0xffffd6a8:   "sex"
```

ㅇㅅㅇ...? 동공지진잼

## solution

``` bash
leviathan1@melinda:~$ ./check
password: sex
$ whoami
leviathan2
$ id
uid=12001(leviathan1) gid=12001(leviathan1) euid=12002(leviathan2) groups=12002(leviathan2),12001(leviathan1)
$ cat /etc/leviathan_pass/leviathan2   
ougahZi8Ta
```

## flag

``` bash
ougahZi8Ta
```


# leviathan2

``` bash
leviathan2@melinda:~$ file ./printfile  
./printfile: setuid ELF 32-bit LSB  executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=d765b4023e214e3fbfe71aa63554713a57e39520, not stripped
leviathan2@melinda:~$ ./printfile
*** File Printer ***
Usage: ./printfile filename
leviathan2@melinda:~$ ./printfile elmo
You cant have that file...
```

## gdb

``` gdb
Dump of assembler code for function main:
   0x0804852d <+0>:   push   ebp
   0x0804852e <+1>:   mov    ebp,esp
   0x08048530 <+3>:   and    esp,0xfffffff0
   0x08048533 <+6>:   sub    esp,0x230
   0x08048539 <+12>:   mov    eax,DWORD PTR [ebp+0xc]
   0x0804853c <+15>:   mov    DWORD PTR [esp+0x1c],eax
   0x08048540 <+19>:   mov    eax,gs:0x14
   0x08048546 <+25>:   mov    DWORD PTR [esp+0x22c],eax
   0x0804854d <+32>:   xor    eax,eax
   0x0804854f <+34>:   cmp    DWORD PTR [ebp+0x8],0x1
   0x08048553 <+38>:   jg     0x804857e <main+81>
   0x08048555 <+40>:   mov    DWORD PTR [esp],0x8048690
   0x0804855c <+47>:   call   0x80483d0 <puts@plt>
   0x08048561 <+52>:   mov    eax,DWORD PTR [esp+0x1c]
   0x08048565 <+56>:   mov    eax,DWORD PTR [eax]
   0x08048567 <+58>:   mov    DWORD PTR [esp+0x4],eax
   0x0804856b <+62>:   mov    DWORD PTR [esp],0x80486a5
   0x08048572 <+69>:   call   0x80483b0 <printf@plt>
   0x08048577 <+74>:   mov    eax,0xffffffff
   0x0804857c <+79>:   jmp    0x80485e8 <main+187>
   0x0804857e <+81>:   mov    eax,DWORD PTR [esp+0x1c]
   0x08048582 <+85>:   add    eax,0x4
   0x08048585 <+88>:   mov    eax,DWORD PTR [eax]
   0x08048587 <+90>:   mov    DWORD PTR [esp+0x4],0x4
   0x0804858f <+98>:   mov    DWORD PTR [esp],eax
   0x08048592 <+101>:   call   0x8048420 <access@plt>
   0x08048597 <+106>:   test   eax,eax
   0x08048599 <+108>:   je     0x80485ae <main+129>
   0x0804859b <+110>:   mov    DWORD PTR [esp],0x80486b9
   0x080485a2 <+117>:   call   0x80483d0 <puts@plt>
   0x080485a7 <+122>:   mov    eax,0x1
   0x080485ac <+127>:   jmp    0x80485e8 <main+187>
   0x080485ae <+129>:   mov    eax,DWORD PTR [esp+0x1c]
   0x080485b2 <+133>:   add    eax,0x4
   0x080485b5 <+136>:   mov    eax,DWORD PTR [eax]
   0x080485b7 <+138>:   mov    DWORD PTR [esp+0xc],eax
   0x080485bb <+142>:   mov    DWORD PTR [esp+0x8],0x80486d4
   0x080485c3 <+150>:   mov    DWORD PTR [esp+0x4],0x1ff
   0x080485cb <+158>:   lea    eax,[esp+0x2c]
   0x080485cf <+162>:   mov    DWORD PTR [esp],eax
   0x080485d2 <+165>:   call   0x8048410 <snprintf@plt>
   0x080485d7 <+170>:   lea    eax,[esp+0x2c]
   0x080485db <+174>:   mov    DWORD PTR [esp],eax
   0x080485de <+177>:   call   0x80483e0 <system@plt>
   0x080485e3 <+182>:   mov    eax,0x0
   0x080485e8 <+187>:   mov    edx,DWORD PTR [esp+0x22c]
   0x080485ef <+194>:   xor    edx,DWORD PTR gs:0x14
   0x080485f6 <+201>:   je     0x80485fd <main+208>
   0x080485f8 <+203>:   call   0x80483c0 <__stack_chk_fail@plt>
   0x080485fd <+208>:   leave  
   0x080485fe <+209>:   ret    
End of assembler dump.
(gdb) b *main
Breakpoint 1 at 0x804852d
(gdb) r
Starting program: /home/leviathan2/printfile

Breakpoint 1, 0x0804852d in main ()
(gdb) x/s 0x80486b9
0x80486b9:   "You cant have that file..."
(gdb) b *main+101
Breakpoint 2 at 0x8048592
```

http://forum.falinux.com/zbxe/index.php?document_srl=412987&mid=C_LIB
int access(const char *pathname, int mode);
access()는 프로세스가 지정한 파일이 존재하는지, 읽거나 쓰거나 실행이 가능한 지를 확인하는 함수입니다.
만일 지정한 파일이 심볼릭 링크라면 링크의 원본을 체크합니다.
access()의 첫 번째 인자는 파일이나 디렉토리의 전체 이름이며, 두 번째 인자는 체크할 내용을 지정하게 됩니다.
반환 0   가능 또는 파일이 존재함
반환 -1   mode 에 대해 하나 이상 거절되었거나 에러가 있음. 자세한 내용은 errno에 세팅됨

``` gdb
(gdb) r asdf
Starting program: /home/leviathan2/printfile asdf

Breakpoint 1, 0x0804852d in main ()
(gdb) c
Continuing.

Breakpoint 2, 0x08048592 in main ()
(gdb) x/wx $esp
0xffffd480:   0xffffd8a1 //첫번째 인자 주소
(gdb)
0xffffd484:   0x00000004 //두번째 인자
(gdb) x/s 0xffffd8a1
0xffffd8a1:   "asdf" //넣은거
(gdb) ni
0x08048597 in main ()
(gdb) i r
eax            0xffffffff   -1 //반환
ecx            0xffffffbc   -68
edx            0xf7fca000   -134438912
ebx            0xf7fca000   -134438912
esp            0xffffd480   0xffffd480
ebp            0xffffd6b8   0xffffd6b8
esi            0x0   0
edi            0x0   0
eip            0x8048597   0x8048597 <main+106>
eflags         0x286   [ PF SF IF ]
cs             0x23   35
ss             0x2b   43
ds             0x2b   43
es             0x2b   43
fs             0x0   0
gs             0x63   99
(gdb) c
Continuing.
You cant have that file...
[Inferior 1 (process 30920) exited with code 01]
(gdb) q                   
```

``` bash
leviathan2@melinda:~$ ls
printfile
leviathan2@melinda:~$ ls -asl /etc/leviathan_pass/
total 40
4 drwxr-xr-x   2 root       root       4096 Mar 21  2015 .
4 drwxr-xr-x 108 root       root       4096 Jan 27 00:44 ..
4 -r--------   1 leviathan0 leviathan0   11 Nov 14  2014 leviathan0
4 -r--------   1 leviathan1 leviathan1   11 Nov 14  2014 leviathan1
4 -r--------   1 leviathan2 leviathan2   11 Nov 14  2014 leviathan2
4 -r--------   1 leviathan3 leviathan3   11 Mar 21  2015 leviathan3
4 -r--------   1 leviathan4 leviathan4   11 Nov 14  2014 leviathan4
4 -r--------   1 leviathan5 leviathan5   11 Nov 14  2014 leviathan5
4 -r--------   1 leviathan6 leviathan6   11 Nov 14  2014 leviathan6
4 -r--------   1 leviathan7 leviathan7   11 Nov 14  2014 leviathan7
leviathan2@melinda:~$ ./printfile /etc/leviathan_pass/leviathan3
You cant have that file...
leviathan2@melinda:~$ ./printfile /etc/leviathan_pass/leviathan2
/bin/cat: /etc/leviathan_pass/leviathan2: Permission denied
leviathan2@melinda:~$ ./printfile /etc/leviathan_pass/leviathan1
You cant have that file...

leviathan2@melinda:~$ mkdir /tmp/elmo2
leviathan2@melinda:~$ cd /tmp/elmo2
leviathan2@melinda:/tmp/elmo2$ ls
leviathan2@melinda:/tmp/elmo2$ cat > asdf
hihihi
leviathan2@melinda:/tmp/elmo2$ cd ~
leviathan2@melinda:~$ ./printfile /tmp/elmo2/asdf
hihihi
```

c일때
eax = [ebp+0xc]

*arr = malloc(13);
*arr, arr[0]
*(arr+1), arr[1]

*(arr+1), arr[1]

어셈일때  
arr
add 4 //int가 4바이트니까
[]

int main(int argc, char **argv, char ** envp)
{
}

함수 프롤로그: 이전함수 ebp를 저장하기위함
push ebp
mov ebp, esp

에필로그: 이전함수 ebp를 다시 꺼내오기위함
mov esp, ebp
pop ebp

[ebp+0xc] = argv
eax = argv


argv는 더블포인터 = 값A를 가지고있는 변수

주소A        주소B         주소C
 값A          값B           값C



값A랑 주소B는 같음
값B랑 주소C랑 같음

&argv = ? 주소 A
argv = ? 값 A
*argv = ? 값 B
**argv = ? 값 C

int *pointer;
int a=3;

pointer = &a;
printf("%d",*pointer); => 3

[] => *

``` gdb
mov    eax,DWORD PTR [ebp+0xc] // eax = argv
mov    DWORD PTR [esp+0x1c],eax // [esp+0x1c] = argv
mov    eax,gs:0x14 // stack canary, rand num 4byte
mov    DWORD PTR [esp+0x22c],eax  // [esp+0x22c] = stack canary
xor    eax,eax // eax = 0
cmp    DWORD PTR [ebp+0x8],0x1 // cmp argc, 1
jg     0x804857e <main+81> // argc > 1 면 main+81로 점프
mov    DWORD PTR [esp],0x8048690 // "*** File Printer ***"
call   0x80483d0 <puts@plt> // puts("*** File Printer ***");
mov    eax,DWORD PTR [esp+0x1c] // eax = argv
mov    eax,DWORD PTR [eax] // eax = [eax] = [argv] = argv[0]
mov    DWORD PTR [esp+0x4],eax // [esp+0x4] = argv[0]
mov    DWORD PTR [esp],0x80486a5 // [esp] = 0x80486a5
call   0x80483b0 <printf@plt> // printf("Usage: %s filename\n", argv[0]); // 실행하는 바이너리 이름 (./printfile, /home/leviathan2/printfile 심볼릭 링크로 변경 가능)
mov    eax,0xffffffff // eax = -1
cmp    0x80485e8 <main+187> // return -1


mov    eax,DWORD PTR [esp+0x1c] // eax = argv
add    eax,0x4 // eax += 4, eax = argv + 4
mov    eax,DWORD PTR [eax] eax = [eax] = [argv + 4] = argv[1]
mov    DWORD PTR [esp+0x4],0x4 // [esp+0x4] = 0x4
mov    DWORD PTR [esp],eax // [esp] = argv[1]
call   0x8048420 <access@plt> // access(argv[1], 0x4); // argv[1]은 윤서가 넣어주는 값
test   eax,eax // eax = access(argv[1], 0x4)의 리턴 값.
// test A, B -> A&B zeroflag, test A,A -> A&A zeroflag
// A = 0 -> zeroflag set
// A !=0 -> zeroflag clr
je     0x80485ae <main+129> // je -> jump equal -> zeroflag가 set되어있으면 점프.
// 즉 eax = access(argv[1], 0x4)가 0을 리턴하면 저플 main+129


// access(argv[1],0x4)!=0
mov    DWORD PTR [esp],0x80486b9
call   0x80483d0 <puts@plt>
mov    eax,0x1
jmp    0x80485e8 <main+187>
// puts("You cant have that file...");
// return 1;


mov    eax,DWORD PTR [esp+0x1c] // eax = argv
add    eax,0x4 // eax = argv + 4
mov    eax,DWORD PTR [eax] eax = [argv + 4] = argv[1]
mov    DWORD PTR [esp+0xc],eax // [esp+0xc] = argv[1]
mov    DWORD PTR [esp+0x8],0x80486d4 // [esp+8] = 0x80486d4
mov    DWORD PTR [esp+0x4],0x1ff // [esp+4] = 0x1ff
lea    eax,[esp+0x2c] // eax = esp+0x2c
mov    DWORD PTR [esp],eax // [esp] = esp + 0x2c
call   0x8048410 <snprintf@plt> // snprintf(esp+0x2c, 0x1ff, 0x80486d4, argv[1]);
// snprintf(buf, 0x1ff, "/bin/cat %s", argv[1]); // argv[1]은 윤서가 넣은거고 buf = esp + 0x2c

lea    eax,[esp+0x2c] // eax = buf
mov    DWORD PTR [esp],eax // [esp] = eax
call   0x80483e0 <system@plt> // system(buf);
mov    eax,0x0 // return 0;
mov    edx,DWORD PTR [esp+0x22c]
xor    edx,DWORD PTR gs:0x14
je     0x80485fd <main+208>
call   0x80483c0 <__stack_chk_fail@plt>
// canary check
leave  
ret    
```

이걸 c로 다시 쓰면
``` c
int main(int argc, char **argv)
{
   char buf[512];
   if(argc<=1)
   {
      puts("~~");
      printf("~~~");
      return -1;
   }
   if(access(argv[1],4))
   {
      puts(~~);
      return 1;
   }
   snprintf(buf, 0x1ff, "/bin/cat %s",argv[1]);
   system(buf);
   return 0;
}
```
뀨


``` bash
cat
"a;cat</etc/leviathan_pass/leviathan3"
cat a

"a;cat "
a;cat<
```

## solution

``` bash
leviathan2@melinda:/tmp/elmo2$ touch a
leviathan2@melinda:/tmp/elmo2$ ls
a
leviathan2@melinda:/tmp/elmo2$ mkdir "a;cat<"
leviathan2@melinda:/tmp/elmo2$ cd a\;cat\</
leviathan2@melinda:/tmp/elmo2/a;cat<$ mkdir etc
leviathan2@melinda:/tmp/elmo2/a;cat<$ cd etc/
leviathan2@melinda:/tmp/elmo2/a;cat</etc$ mkdir leviathan_pass
leviathan2@melinda:/tmp/elmo2/a;cat</etc$ cd leviathan_pass/
leviathan2@melinda:/tmp/elmo2/a;cat</etc/leviathan_pass$ touch leviathan3
leviathan2@melinda:/tmp/elmo2/a;cat</etc/leviathan_pass$ cd /tmp/lemo2
-bash: cd: /tmp/lemo2: No such file or directory
leviathan2@melinda:/tmp/elmo2/a;cat</etc/leviathan_pass$ cd /tmp/elmo2
leviathan2@melinda:/tmp/elmo2$ ~/printfile "a;cat</etc/leviathan_pass/leviathan3"
Ahdiemoo1j
```

## flag

``` bash
Ahdiemoo1j
```

# leviathan3

``` bash
leviathan3@melinda:~$ ls
level3
leviathan3@melinda:~$ file level3
level3: setuid ELF 32-bit LSB  executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=9ee1bc84cc2a39de9df95a77cb807136b1ba6db2, not stripped
leviathan3@melinda:~$ ./level3
Enter the password> AAAAAAAA
bzzzzzzzzap. WRONG
```

## gdb

``` gdb
main()
   0x08048607 <+9>:   mov    eax,gs:0x14 //eax = rand 4byte
   0x0804860d <+15>:   mov    DWORD PTR [esp+0x4c],eax //[esp+0x4c] = rand 4byte
   0x08048611 <+19>:   xor    eax,eax // eax = 0
   0x08048613 <+21>:   mov    DWORD PTR [esp+0x23],0x626d6f62 //[esp+0x23] = 0x626d6f62
   0x0804861b <+29>:   mov    WORD PTR [esp+0x27],0x6461 //[esp+0x27] = 0x6461
   0x08048622 <+36>:   mov    BYTE PTR [esp+0x29],0x0 //[esp+0x29] = 0x0
   0x08048627 <+41>:   mov    DWORD PTR [esp+0x38],0x732e2e2e //[esp+0x38] = 0x732e2e2e
   0x0804862f <+49>:   mov    DWORD PTR [esp+0x3c],0x33726333 //[esp+0x3c] = 0x33726333
   0x08048637 <+57>:   mov    WORD PTR [esp+0x40],0x74 //[esp+0x40] = 0x74
   0x0804863e <+64>:   mov    DWORD PTR [esp+0x2a],0x6f6e3068 //[esp+0x2a] = 0x6f6e3068
   0x08048646 <+72>:   mov    WORD PTR [esp+0x2e],0x3333 //[esp+0x2e] = 0x3333
   0x0804864d <+79>:   mov    BYTE PTR [esp+0x30],0x0 //[esp+0x30] = 0x0
   0x08048652 <+84>:   mov    DWORD PTR [esp+0x31],0x616b616b //[esp+0x31] = 0x616b616b
   0x0804865a <+92>:   mov    WORD PTR [esp+0x35],0x616b //[esp+0x35] = 0x616b
   0x08048661 <+99>:   mov    BYTE PTR [esp+0x37],0x0 //[esp+0x37] = 0x0
   0x08048666 <+104>:   mov    DWORD PTR [esp+0x42],0x2e32332a //[esp+0x42] = 0x2e32332a
   0x0804866e <+112>:   mov    DWORD PTR [esp+0x46],0x785b2a32 //[esp+0x46] = 0x785b2a32
   0x08048676 <+120>:   mov    WORD PTR [esp+0x4a],0x5d //[esp+0x4a] = 0x5d
   0x0804867d <+127>:   lea    eax,[esp+0x31] //eax = 0x616b616b
   0x08048681 <+131>:   mov    DWORD PTR [esp+0x4],eax //[esp+0x4] = 0x616b616b
   0x08048685 <+135>:   lea    eax,[esp+0x2a] //eax = 0x6f6e3068
   0x08048689 <+139>:   mov    DWORD PTR [esp],eax //[esp] = eax
   0x0804868c <+142>:   call   0x80483d0 <strcmp@plt> //strcmp(0x6f6e3068, 0x616b616b);
   0x08048691 <+147>:   test   eax,eax //둘이 같으면 zeroflag set, 다르면 clr
   0x08048693 <+149>:   jne    0x804869d <main+159> //jump if not equal -> zeroflag clr면 점프

//같으면
   0x08048695 <+151>:   mov    DWORD PTR [esp+0x1c],0x1 //[esp+0x1c] = 0x1

//0x6f6e3068과 0x616b616b가 다르면
   0x0804869d <+159>:   mov    DWORD PTR [esp],0x804878f //[esp] = "Enter the password> "
   0x080486a4 <+166>:   call   0x80483e0 <printf@plt> //printf("Enter the password> ");
   0x080486a9 <+171>:   call   0x804854d <do_stuff> //do_stuff();
   0x080486ae <+176>:   mov    eax,0x0 //eax = 0
   0x080486b3 <+181>:   mov    edx,DWORD PTR [esp+0x4c] //edx=[esp+0x4c]
   0x080486b7 <+185>:   xor    edx,DWORD PTR gs:0x14 //stack canary check
   0x080486be <+192>:   je     0x80486c5 <main+199>
   0x080486c0 <+194>:   call   0x8048400 <__stack_chk_fail@plt>
   0x080486c5 <+199>:   leave  
   0x080486c6 <+200>:   ret    




do_stuff()

   0x0804855c <+15>:   mov    DWORD PTR [ebp-0xc],eax //[ebp-0xc] = eax
   0x0804855f <+18>:   xor    eax,eax // eax = 0
   0x08048561 <+20>:   mov    DWORD PTR [ebp-0x117],0x706c6e73 //[ebp-0x117] = 0x706c6e73
   0x0804856b <+30>:   mov    DWORD PTR [ebp-0x113],0x746e6972 //[ebp-0x113] = 0x746e6972
   0x08048575 <+40>:   mov    WORD PTR [ebp-0x10f],0xa66 //[ebp-0x10f] = 0xa66
   0x0804857e <+49>:   mov    BYTE PTR [ebp-0x10d],0x0 //[ebp-0x10d] = 0x0
   0x08048585 <+56>:   mov    eax,ds:0x804a03c //eax = stdin
   0x0804858a <+61>:   mov    DWORD PTR [esp+0x8],eax //[esp+0x8] = stdin
   0x0804858e <+65>:   mov    DWORD PTR [esp+0x4],0x100 //[esp+0x4] = 0x100
   0x08048596 <+73>:   lea    eax,[ebp-0x10c] //eax = ebp-0x10c
   0x0804859c <+79>:   mov    DWORD PTR [esp],eax //[esp] = ebp-0x10c
   0x0804859f <+82>:   call   0x80483f0 <fgets@plt> fgets()
   //char * fgets ( char * str, int num, FILE * stream );
   //str 읽어들인 문자열을 저장할 char 배열을 가리키는 포인터.
   //num 마지막 NULL 문자를 포함하여, 읽어들일 최대 문자 수. 다시 말해 이 값이 10 이면 컴퓨터는 최대 9 문자를 입력 받는다.
   //stream 문자열을 읽어들일 스트림의 FILE 객체를 가리키는 포인터. 특히, 표준 입력(stdin) 에서 입력을 받으려면 여기에 stdin 을 써주면 된다. (예를 들어 fgets (str, 100, stdin); 과 같이)
   //성공적으로 읽어들였다면 함수는 str 을 리턴한다.
   //fgets(ebp-0x10c,256,stdin)
   //ebp-0x10c는 내가 쓴거
   0x080485a4 <+87>:   lea    eax,[ebp-0x117] //eax = ebp-0x117
   0x080485aa <+93>:   mov    DWORD PTR [esp+0x4],eax // [esp+0x4] = ebp-0x117
   0x080485ae <+97>:   lea    eax,[ebp-0x10c] eax = ebp-0x10c
   0x080485b4 <+103>:   mov    DWORD PTR [esp],eax [esp] = ebp-0x10c
   0x080485b7 <+106>:   call   0x80483d0 <strcmp@plt> //strcmp(ebp-0x10c, ebp-0x117);
   0x080485bc <+111>:   test   eax,eax
   0x080485be <+113>:   jne    0x80485da <do_stuff+141>

   //둘이 같으면
   0x080485c0 <+115>:   mov    DWORD PTR [esp],0x8048760 //[esp] = 0x8048760
   0x080485c7 <+122>:   call   0x8048410 <puts@plt> //puts("[You've got shell]!");
   0x080485cc <+127>:   mov    DWORD PTR [esp],0x8048774 //[esp] = 0x8048774
   0x080485d3 <+134>:   call   0x8048420 <system@plt> //system("/bin/sh");
   0x080485d8 <+139>:   jmp    0x80485e6 <do_stuff+153>

   //둘이 다르면
   0x080485da <+141>:   mov    DWORD PTR [esp],0x804877c //[esp] = 0x804877c
   0x080485e1 <+148>:   call   0x8048410 <puts@plt> //puts("bzzzzzzzzap. WRONG");

   0x080485e6 <+153>:   mov    eax,0x0
   0x080485eb <+158>:   mov    edx,DWORD PTR [ebp-0xc]
```

## solution
``` bash
(gdb) x/s 0x8048760
0x8048760:   "[You've got shell]!"
(gdb) x/s 0x804877c
0x804877c:   "bzzzzzzzzap. WRONG"

(gdb) b *do_stuff+106
Breakpoint 2 at 0x80485b7: file level3.c, line 12.
(gdb) cont
Continuing.
Enter the password> AAAA

Breakpoint 2, 0x080485b7 in do_stuff () at level3.c:12
12   in level3.c
(gdb) x/s $ebp-0x10c
0xffffd55c:   "AAAA\n"
(gdb) x/s $ebp-0x117
0xffffd551:   "snlprintf\n"

leviathan3@melinda:~$ ./level3
Enter the password> snlpritf
bzzzzzzzzap. WRONG
leviathan3@melinda:~$ ./level3
Enter the password> snlprintf
[You've got shell]!
$ cat /etc/leviathan_pass/leviathan4
vuH0coox6m
```

## flag

``` bash
vuH0coox6m
```

# leviathan4

``` bash
leviathan4@melinda:~$ ls -al
total 24
drwxr-xr-x   3 root root       4096 Nov 14  2014 .
drwxr-xr-x 172 root root       4096 Jul 10  2016 ..
-rw-r--r--   1 root root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root root        675 Apr  9  2014 .profile
dr-xr-x---   2 root leviathan4 4096 Nov 14  2014 .trash
leviathan4@melinda:~$ cd .trash
leviathan4@melinda:~/.trash$ ls -al
total 16
dr-xr-x--- 2 root       leviathan4 4096 Nov 14  2014 .
drwxr-xr-x 3 root       root       4096 Nov 14  2014 ..
-r-sr-x--- 1 leviathan5 leviathan4 7425 Nov 14  2014 bin
leviathan4@melinda:~/.trash$ file ./bin
./bin: setuid ELF 32-bit LSB  executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=3be0f50b480c24c4223fc43cca29ab377e140fc7, not stripped
leviathan4@melinda:~/.trash$ ./bin
01010100 01101001 01110100 01101000 00110100 01100011 01101111 01101011 01100101 01101001 00001010
```

## gdb

``` gdb
(gdb) disas main
Dump of assembler code for function main:

   0x080484d7 <+10>:   mov    DWORD PTR [esp+0x4],0x8048650 //[esp+0x4] = "r"
   0x080484df <+18>:   mov    DWORD PTR [esp],0x8048654 //[esp] ="/etc/leviathan_pass/leviathan5"
   0x080484e6 <+25>:   call   0x80483b0 <fopen@plt>
   //FILE * fopen ( const char * filename, const char * mode );
   //filename C 문자열로 열을 파일의 이름
   //mode C 문자열로 파일 접근 모드를 설정 (r, w, a)
   //만일 파일이 성공적으로 열렸다면 fopen 함수는 FILE 객체에 대한 포인터를 리턴할 것이다. 이 포인터는 나중에 스트림 관련 작업시에 스트림을 구분하기 위해 자주 사용된다. 그렇지 않을 경우 널 포인터가 리턴된다.
   //fp = fopen ("/etc/leviathan_pass/leviathan5", "r")

   0x080484eb <+30>:   mov    DWORD PTR [esp+0x1c],eax //[esp+0x1c] = eax //[esp+0x1c] = fp
   0x080484ef <+34>:   cmp    DWORD PTR [esp+0x1c],0x0 //cmp fp, 0x0
   0x080484f4 <+39>:   jne    0x8048500 <main+51>

   //fp가 0이면
   0x080484f6 <+41>:   mov    eax,0xffffffff //eax = 0xffffffff
   0x080484fb <+46>:   jmp    0x80485ae <main+225> //return

   //fp가 0이 아니면
   0x08048500 <+51>:   mov    eax,DWORD PTR [esp+0x1c] //eax = fp
   0x08048504 <+55>:   mov    DWORD PTR [esp+0x8],eax //[esp+0x8] = fp
   0x08048508 <+59>:   mov    DWORD PTR [esp+0x4],0x100 //[esp+0x4] = 256
   0x08048510 <+67>:   mov    DWORD PTR [esp],0x804a060 //[esp] = buf
   0x08048517 <+74>:   call   0x8048370 <fgets@plt>
   //fgets(buf,256,fp)

   //file *fp = fopen("inout","r");
   //fgets(buf,256,fp)

   0x0804851c <+79>:   mov    DWORD PTR [esp+0x14],0x0 //[esp+0x14] = 0x0
   //i=
   0x08048524 <+87>:   jmp    0x8048589 <main+188>

   //1 < strlen(buf)
   0x08048526 <+89>:   mov    eax,DWORD PTR [esp+0x14] //eax = 1
   0x0804852a <+93>:   add    eax,0x804a060 //eax = 1 + buf
   0x0804852f <+98>:   movzx  eax,BYTE PTR [eax] //eax = [1+buf]
   0x08048532 <+101>:   mov    BYTE PTR [esp+0x13],al //[esp+0x13] = al
   0x08048536 <+105>:   mov    DWORD PTR [esp+0x18],0x0 //[esp+0x18] = 0x0
   0x0804853e <+113>:   jmp    0x8048571 <main+164>

   //[esp+0x18] <= 0x7
   0x08048540 <+115>:   cmp    BYTE PTR [esp+0x13],0x0 // cmp [esp+0x13], 0x0
   0x08048545 <+120>:   jns    0x8048555 <main+136> //부호비트가 0일때
   0x08048547 <+122>:   mov    DWORD PTR [esp],0x31 //[esp] = 0x31
   0x0804854e <+129>:   call   0x80483c0 <putchar@plt> //putchar("1");
   0x08048553 <+134>:   jmp    0x8048561 <main+148>

   0x08048555 <+136>:   mov    DWORD PTR [esp],0x30 //[esp]=0x30
   0x0804855c <+143>:   call   0x80483c0 <putchar@plt> //putchar("0");

   0x08048561 <+148>:   movsx  eax,BYTE PTR [esp+0x13] //eax = [esp+0x13]
   0x08048566 <+153>:   add    eax,eax //eax = [esp+0x13] + [esp+0x13]
   0x08048568 <+155>:   mov    BYTE PTR [esp+0x13],al //[esp+0x13] = al
   0x0804856c <+159>:   add    DWORD PTR [esp+0x18],0x1 //[esp+0x18] = [esp+0x18] + 0x1

   0x08048571 <+164>:   cmp    DWORD PTR [esp+0x18],0x7 //[esp+0x18] <= 0x7
   0x08048576 <+169>:   jle    0x8048540 <main+115>

   //[esp+0x18] >= 0x7
   0x08048578 <+171>:   mov    DWORD PTR [esp],0x20 //[esp] = 0x20
   0x0804857f <+178>:   call   0x80483c0 <putchar@plt> //putchar(" "); //space
   0x08048584 <+183>:   add    DWORD PTR [esp+0x14],0x1 //[esp+0x14] = [esp+0x14] + 0x1

   0x08048589 <+188>:   mov    ebx,DWORD PTR [esp+0x14] //ebx = [esp+0x14] //ebx = 1
   //(gdb) x/u $esp+0x14
   //0xffffd6d0:   1

   0x0804858d <+192>:   mov    DWORD PTR [esp],0x804a060 //[esp] = buf
   0x08048594 <+199>:   call   0x8048390 <strlen@plt> //eax = strlen(buf)
   0x08048599 <+204>:   cmp    ebx,eax //cmp 1 < eax // 1 < strlen(buf)
   0x0804859b <+206>:   jb     0x8048526 <main+89> //jump if below

   //1 > strlen(buf)
   0x0804859d <+208>:   mov    DWORD PTR [esp],0xa //[esp] = 0xa
   0x080485a4 <+215>:   call   0x80483c0 <putchar@plt> //putchar("\n");
   0x080485a9 <+220>:   mov    eax,0x0 //eax = 0x0
   0x080485ae <+225>:   mov    ebx,DWORD PTR [ebp-0x4] //ebx = [ebp-0x4]
   0x080485b1 <+228>:   leave  
   0x080485b2 <+229>:   ret    
```

## solution

``` bash
01010100 01101001 01110100 01101000 00110100 01100011 01101111 01101011 01100101 01101001 00001010
to ascii
```

## flag

``` bash
Tith4cokei
```

# leviathan5

``` bash
leviathan5@melinda:~$ ls
leviathan5
leviathan5@melinda:~$ file leviathan5
leviathan5: setuid ELF 32-bit LSB  executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=c71b3ae0d0395851879ee6fb8ede92e1676ecf71, not stripped
leviathan5@melinda:~$ ./leviathan5
Cannot find /tmp/file.log
```

## gdb

``` gdb
leviathan5@melinda:~$ gdb -q ./leviathan5
Reading symbols from ./leviathan5...(no debugging symbols found)...done.
(gdb) set disassembly-flavor intel
(gdb) disas main
Dump of assembler code for function main:

   0x080485f6 <+9>:   mov    DWORD PTR [esp+0x4],0x8048720
   0x080485fe <+17>:   mov    DWORD PTR [esp],0x8048722
   0x08048605 <+24>:   call   0x80484b0 <fopen@plt>
   //fopen("/tmp/file.log", "r")

   0x0804860a <+29>:   mov    DWORD PTR [esp+0x1c],eax //[esp+0x1c] = eax
   0x0804860e <+33>:   cmp    DWORD PTR [esp+0x1c],0x0 //cmp eax, 0
   0x08048613 <+38>:   jne    0x804862d <main+64>

   //fp not found -> puts("Cannot find /tmp/file.log"); exit();
   0x08048615 <+40>:   mov    DWORD PTR [esp],0x8048730
   0x0804861c <+47>:   call   0x8048460 <puts@plt>
   0x08048621 <+52>:   mov    DWORD PTR [esp],0xffffffff
   0x08048628 <+59>:   call   0x8048480 <exit@plt>

   //fp found
   0x0804862d <+64>:   mov    eax,DWORD PTR [esp+0x1c] //eax = fp
   0x08048631 <+68>:   mov    DWORD PTR [esp],eax //[esp] = fp
   0x08048634 <+71>:   call   0x80484d0 <fgetc@plt> //fgetc(fp);
   //int fgetc ( FILE * stream ); 스트림(stream) 에서 문자 하나를 읽어온다
   //인자로 전달한 stream 의 파일 위치 지정자가 가리키는 문자를 리턴한다. 이 때 파일 위치 지정자는 그 다음 문자를 가리키게 된다.
   0x08048639 <+76>:   mov    BYTE PTR [esp+0x1b],al //[esp+0x1b] = al
   0x0804863d <+80>:   mov    eax,DWORD PTR [esp+0x1c] //eax = fp
   0x08048641 <+84>:   mov    DWORD PTR [esp],eax //[esp] = fp
   0x08048644 <+87>:   call   0x8048490 <feof@plt> //feof(fp);
   //int feof( FILE *stream); 파일의 끝이면 0이 아닌 값을, 끝이 아니면 0
   0x08048649 <+92>:   test   eax,eax
   0x0804864b <+94>:   je     0x804864f <main+98>

   //파일 끝이면
   0x0804864d <+96>:   jmp    0x804865e <main+113>

   //파일 끝이 아니면
   0x0804864f <+98>:   movsx  eax,BYTE PTR [esp+0x1b] //eax = [esp+0x1b]
   0x08048654 <+103>:   mov    DWORD PTR [esp],eax //[esp] = [esp+0x1b]
   0x08048657 <+106>:   call   0x80484c0 <putchar@plt> //putchar("");
   0x0804865c <+111>:   jmp    0x804862d <main+64>

   //파일 끝이면 이리로 점프
   0x0804865e <+113>:   mov    eax,DWORD PTR [esp+0x1c] //eax = fp
   0x08048662 <+117>:   mov    DWORD PTR [esp],eax //[esp] = fp
   0x08048665 <+120>:   call   0x8048430 <fclose@plt> //fclose(fp);
   0x0804866a <+125>:   call   0x8048440 <getuid@plt> //getuid();
   //getuid() returns the real user ID of the calling process.
   0x0804866f <+130>:   mov    DWORD PTR [esp],eax //[esp] = uid
   0x08048672 <+133>:   call   0x80484e0 <setuid@plt> //setuid(uid)
   0x08048677 <+138>:   mov    DWORD PTR [esp],0x8048722 //[esp] = "/tmp/file.log"
   0x0804867e <+145>:   call   0x8048450 <unlink@plt> = unlink("/tmp/file.log"); //링크를 삭제
   0x08048683 <+150>:   mov    eax,0x0 //return 0

End of assembler dump.
```

``` bash
leviathan5@melinda:~$ vi /tmp/file.log
leviathan5@melinda:~$ cat /tmp/file.log
ABCD
leviathan5@melinda:~$ ./leviathan5
ABCD
```

ln [옵션] 원본파일 대상파일(대상디렉토리)

## solution

``` bash
leviathan5@melinda:~$ ln /etc/leviathan_pass/leviathan6 /tmp/file.log
ln: failed to create hard link '/tmp/file.log' => '/etc/leviathan_pass/leviathan6': Operation not permitted
leviathan5@melinda:~$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
leviathan5@melinda:~$ ./leviathan5
UgaoFee4li
```

## flag

``` bash
UgaoFee4li
```

# leviathan6

``` bash
leviathan6@melinda:~$ ls
leviathan6
leviathan6@melinda:~$ ./leviathan6
usage: ./leviathan6 <4 digit code>
leviathan6@melinda:~$ ./leviathan6 1234
Wrong
```

## gdb

``` gdb
Dump of assembler code for function main:

   0x08048516 <+9>:   mov    DWORD PTR [esp+0x1c],0x1bd3 //[esp+0x1c] = 0x1bd3
   0x0804851e <+17>:   cmp    DWORD PTR [ebp+0x8],0x2 // cmp [ebp+0x8] == 0x2
   0x08048522 <+21>:   je     0x8048545 <main+56>

   //[ebp+0x8] != 0x2 argc가 두개가 아니면 사용법 출력 usage: ./leviathan6 <4 digit code>
   0x08048524 <+23>:   mov    eax,DWORD PTR [ebp+0xc] //eax = argv
   0x08048527 <+26>:   mov    eax,DWORD PTR [eax] //eax = [argv]
   0x08048529 <+28>:   mov    DWORD PTR [esp+0x4],eax //[esp+0x4] = [argv]
   0x0804852d <+32>:   mov    DWORD PTR [esp],0x8048620
   0x08048534 <+39>:   call   0x8048390 <printf@plt>
   0x08048539 <+44>:   mov    DWORD PTR [esp],0xffffffff
   0x08048540 <+51>:   call   0x80483e0 <exit@plt>

   //[ebp+0x8] == 0x2
   0x08048545 <+56>:   mov    eax,DWORD PTR [ebp+0xc] //eax = [ebp+0xc] // eax = argv
   0x08048548 <+59>:   add    eax,0x4 //eax = argv + 0x4 = argv[1] // eax = 4digitcode
   0x0804854b <+62>:   mov    eax,DWORD PTR [eax] // eax = [4digitcode]
   0x0804854d <+64>:   mov    DWORD PTR [esp],eax //[esp] = [4digitcode]
   0x08048550 <+67>:   call   0x8048400 <atoi@plt> //eax = atoi([4digitcode]);
   0x08048555 <+72>:   cmp    eax,DWORD PTR [esp+0x1c] //cmp atoi([4digitcode]), 0x1bd3


   //atoi() : 10진 정수 문자열을 정수로 변환합니다.
문자열에서 10진 정수 숫자 문자 뒤의 일반 문자는 취소되며, 10진 정수 숫자 문자까지만 숫자로 변환됩니다.
10진 정수 숫자 문자 앞의 공백문자는 자동 제거되어 10진 정수 숫자 문자까지만 숫자로 변환됩니다.
공백 및 10진 정수 문자가 아닌 문자로 시작하면 0을 반환합니다.

   0x08048559 <+76>:   jne    0x8048575 <main+104>

   //atoi([4digitcode]) == 0x1bd3
   0x0804855b <+78>:   mov    DWORD PTR [esp],0x3ef
   0x08048562 <+85>:   call   0x80483a0 <seteuid@plt>
   0x08048567 <+90>:   mov    DWORD PTR [esp],0x804863a
   0x0804856e <+97>:   call   0x80483c0 <system@plt>
   0x08048573 <+102>:   jmp    0x8048581 <main+116>

   //if atoi([4digitcode]) != 0x1bd3 -> puts("Wrong")
   0x08048575 <+104>:   mov    DWORD PTR [esp],0x8048642
   0x0804857c <+111>:   call   0x80483b0 <puts@plt>

End of assembler dump.
```

## solution

0x1bd3 == decimal 7123

``` bash
leviathan6@melinda:~$ ./leviathan6 7123
$ whoami           
leviathan7
$ cat /etc/leviathan_pass/leviathan7
ahy7MaeBo9
```

## flag

``` bash
ahy7MaeBo9
```

# leviathan7

``` bash
leviathan7@melinda:~$ ls
CONGRATULATIONS
leviathan7@melinda:~$ cat CONGRATULATIONS
Well Done, you seem to have used a *nix system before, now try something more serious.
(Please don't post writeups, solutions or spoilers about the games on the web. Thank you!)
```

롸업 쓰지말랬는데 써부럿네 ㅇㅁㅇ..
