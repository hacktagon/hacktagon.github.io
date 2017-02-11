---
layout: post
title: "Leviathan"
headline: Leviathan
modified: 2017-02-11
categories: Wargame Leviathan Pwnable Misc XothSec
comments: true
featured: true
---

# Leviathan00
ssh leviathan0@leviathan.labs.overthewire.org // password leviathan0

### Solution
홈 디렉토리에서 ls -asl 명령어를 실행하면 숨겨진 디렉토리 .backup 을 찾을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan00_01.png" style="display: block; margin: auto;">

.backup 디렉토리 내에서 ls 명령어를 실행하면 bookmarks.html을 발견할 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan00_02.png" style="display: block; margin: auto;">

이 html 파일을 cat 명령어를 통해 읽고 grep 명령어를 통해 password 를 찾아보면 leviathan1의 비밀번호를 얻을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan00_03.png" style="display: block; margin: auto;;">

# Leviathan01
ssh leviathan1@leviathan.labs.overthewire.org // password _________

### Solution
홈 디렉토리에서 ls -l 명령어를 실행하면 leviathan2 권한으로 setuid가 걸린 바이너리 check 을 찾을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan01_01.png" style="display: block; margin: auto;">

실행을 하면 password: 라는 문자열을 출력하고 사용자에게 입력을 받는다. 아무 문자열이나 입력하면 Wrong password, Good Bye ... 를 출력하고 종료된다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan01_02.png" style="display: block; margin: auto;">

이 password를 맞추는 것이 중요할 것이라는 것을 예상하며 gdb를 실행해보자.

set disassembly-flavor intel
disassemble main
을 통해 이 문제의 핵심 부분을 살펴보면

<img src="{{ site.url }}/images/2017-02-11/Leviathan01_03.png" style="display: block; margin: auto;">

strcmp 명령어를 통해 두 문자열이 다르면 main+152로 점프하여 puts(0x8048693)를 실행하고 같다면 system(0x804868b)를 실행하고 종료한다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan01_04.png" style="display: block; margin: auto;">

즉 처음 실행했을 때 password로 입력해준 값을 맞추어서 높은 권한의 쉘을 획득하는 것이 목표일 것이다.

strcmp를 call하는 main+129에 BP (Break Point) 를 걸고 실행하여 strcmp로 넘어가는 두 개의 인자가 어떤 것인지 확인해보자.

<img src="{{ site.url }}/images/2017-02-11/Leviathan01_05.png" style="display: block; margin: auto;">

strcmp를 호출하기 직전 스택상황을 생각해보면 첫번째 인자는 esp에 두번째 인자는 esp+4에 있을 것이다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan01_06.png" style="display: block; margin: auto;">

내가 입력해준 문자열과 어떤 문자열을 인자로 하여 strcmp를 실행하는 것을 알 수 있다. 따라서 password를 해당 문자열로 입력해주면 leviathan02 권한의 쉘을 얻을 수 있고 /etc/leviathan_pass/leviathan2 파일을 읽는 것으로 leviathan2의 password를 알아낼 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan01_07.png" style="display: block; margin: auto;">

# Leviathan02
ssh leviathan2@leviathan.labs.overthewire.org // password _________

### Solution
홈 디렉토리에서 ls -l 명령어를 실행하면 leviathan3 권한으로 setuid가 걸린 바이너리 check 을 찾을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_01.png" style="display: block; margin: auto;">

실행을 하면 Usage를 출력하고 종료된다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_02.png" style="display: block; margin: auto;">

gdb를 통해 분석해보자

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_03_1.png" style="display: block; margin: auto;">

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_03_2.png" style="display: block; margin: auto;">

snprintf(buf, 0x1ff, 0x80486d4, argv[1]);
system(buf);
가 실행되는 것을 확인할 수 있다.

여기서 0x80486d4는 "/bin/cat %s" 이다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_04.png" style="display: block; margin: auto;">

즉 Command Injection이 가능할 것이다. argv[1]를 ";/bin/bash"로 하여 실행해보자. 하지만 "You cant have that file...
" 라는 오류가 발생하고 종료된다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_05.png" style="display: block; margin: auto;">

gdb를 다시 살펴보면 access함수를 통해 파일의 유무를 체크하여 해당 에러 메시지를 출력해주는 것을 알 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_06_1.png" style="display: block; margin: auto;">

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_06_2.png" style="display: block; margin: auto;">

이를 우회하기 위해 ";/bin/bash" 가 의미있는 파일이어야한다. 먼저 /tmp에 tmplev2 디렉토리를 만들어서 이를 우회하기 위한 작업을 해보자.

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_07.png" style="display: block; margin: auto;">

이 디렉토리에서 상대 주소로 ";/bin/bash"에 읽을 수 있는 파일이 있으면 된다. 즉 ; 디렉토리를 만들고 그 안에 bin 디렉토리를 만들고 bash라는 파일을 만들어주면 된다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_08.png" style="display: block; margin: auto;">

그런 다음 이전의 명령어를 다시 실행하면 높은 권한의 쉘을 얻을 수 있을 것이라 생각했다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_09.png" style="display: block; margin: auto;">

문제는 "/bin/cat ;/bin/bash"를 실행하면 cat이 종료되지 않아 쉘이 시작되지 않는 것이다. cat이 종료되기 위해 cat 명령어 뒤에 읽을 파일을 주는 것이 필요하다. "/bin/cat file;/bin/bash" 가 실행되도록 수정해보자.

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_10.png" style="display: block; margin: auto;">

이런 다음 argv[1]이 "file;/bin/bash"가 되도록 실행하면 높은 레벨의 쉘을 얻을 수 있다고 생각했는데 그래도 안된다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_11.png" style="display: block; margin: auto;">

이유는 정확히 모르겠으나 setuid가 bash에는 적용이 안된다고 한다.
bash가 아닌 sh로 하면 높은 권한의 쉘을 얻을 수 있고, "cat /etc/leviathan_pass/leviathan3" 명령어를 통해 leviathan3의 password를 얻을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan02_12.png" style="display: block; margin: auto;">

# Leviathan03
ssh leviathan3@leviathan.labs.overthewire.org // password _________

### Solution
홈 디렉토리에서 ls -l 명령어를 실행하면 leviathan4 권한으로 setuid가 걸린 바이너리 level3 을 찾을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan03_01.png" style="display: block; margin: auto;">

실행하면 "Enter the password>" 가 출력되고 사용자에게 입력을 받는다. 아무 문자열이나 입력하면 "bzzzzzzzzap. WRONG"가 출력되고 종료된다. 레벨 1과 비슷한 느낌을 받을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan03_02.png" style="display: block; margin: auto;">

gdb를 통해 level3 바이너리를 분석해보자. 두 개의 문자열을 비교하여 같으면 [esp+0x1c]를 1로 바꾸고 다르면 바꾸지 않고 printf(0x804878f)를 실행하고 do_stuff()를 실행하다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan03_03_1.png" style="display: block; margin: auto;">

<img src="{{ site.url }}/images/2017-02-11/Leviathan03_03_2.png" style="display: block; margin: auto;">

비교하는 두 문자열은 각각 "h0no33", "kakaka"로 고정된 문자열이다. 따라서 [esp+0x1c]는 1이 되지 않을 것이다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan03_04.png" style="display: block; margin: auto;">

do_stuff 함수 내부로 들어가면 ebp-0x10c을 buf로 사용하여 fgets를 통해 사용자 입력을 받고 strcmp를 통해 두 문자열이 같은 경우 puts(0x8048760), system(0x8048774)를 실행하고 다른 경우 puts(0x804877c)를 실행하는 것을 볼 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan03_05_1.png" style="display: block; margin: auto;">

<img src="{{ site.url }}/images/2017-02-11/Leviathan03_05_2.png" style="display: block; margin: auto;">

strcmp에 BP를 걸고 어떤 문자열을 비교하는지 알아보자.

<img src="{{ site.url }}/images/2017-02-11/Leviathan03_06_1.png" style="display: block; margin: auto;">

<img src="{{ site.url }}/images/2017-02-11/Leviathan03_06_2.png" style="display: block; margin: auto;">

level3 바이너리를 실행하고 해당 문자열을 입력해주면 level4 권한의 쉘을 얻을 수 있고 "cat /etc/leviathan_pass/leviathan4" 명령어를 통해 leviathan4의 password를 얻을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan03_07.png" style="display: block; margin: auto;">

# Leviathan04
ssh leviathan4@leviathan.labs.overthewire.org // password _________

### Solution
홈 디렉토리에서 ls -al 명령어를 실행하면 숨은 디렉토리 .trash를 찾을 수 있다.
.trash 디렉토리에서 ls -l 명령어를 실행하면 leviathan5 권한으로 setuid가 걸린 바이너리 bin을 찾을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan04_01.png" style="display: block; margin: auto;">

bin을 실행하면 0과 1로 이루어진 어떤 문자열이 출력되고 종료된다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan04_02.png" style="display: block; margin: auto;">

01010100 01101001 01110100 01101000 00110100 01100011 01101111 01101011 01100101 01101001 00001010

이 문자열을 보면 각각 8글자이고 왼쪽에서 두번째 글자가 1인 경우가 많은 것으로 보아 Ascii 문자라고 생각할 수 있다.
binary-to-ascii-text-converter라는 기능을 제공해주는 홈페이지에서 이를 번역하면 leviathan5의 password를 알 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan04_03.png" style="display: block; margin: auto;">

# Leviathan05
ssh leviathan5@leviathan.labs.overthewire.org // password _________

### Solution
홈 디렉토리에서 ls -l 명령어를 실행하면 leviathan6 권한으로 setuid가 걸린 바이너리 leviathan5을 찾을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan05_01.png" style="display: block; margin: auto;">

leviathan5를 실행해보면 "Cannot find /tmp/file.log"라는 오류메시지가 출력되고 종료된다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan05_02.png" style="display: block; margin: auto;">

gdb로 leviathan5를 분석해보자.

<img src="{{ site.url }}/images/2017-02-11/Leviathan05_03.png" style="display: block; margin: auto;">

오류메시지와 같이 연관지어 생각해보면 /tmp/file.log의 파일을 읽어주는 역활을 하는 것을 알 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan05_04.png" style="display: block; margin: auto;">

그런 다음 해당 파일을 삭제한다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan05_05.png" style="display: block; margin: auto;">

간단하게 다음 레벨의 password 파일을 /tmp/file.log로 "symbolic link" 하는 것으로 이 문제를 해결할 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan05_06.png" style="display: block; margin: auto;">

# Leviathan06
ssh leviathan6@leviathan.labs.overthewire.org // password _________

### Solution
홈 디렉토리에서 ls -l 명령어를 실행하면 leviathan7 권한으로 setuid가 걸린 바이너리 leviathan6을 찾을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan06_01.png" style="display: block; margin: auto;">

leviathan6를 실행하면 usage를 확인할 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan06_02.png" style="display: block; margin: auto;">

usage에 따라 argv[1]을 임의의 4자리 digit code로 하여 실행하면 Wrong이라는 오류메시지가 출력된다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan06_03.png" style="display: block; margin: auto;">

gdb를 통해 leviathan6를 분석해보자.
atoi 함수를 통해 내가 넣어준 숫자열을 숫자로 변환하여 [esp+0x1c]의 숫자와 비교하여 같으면 system(0x804863a)을 다르면 puts(0x8048642)를 실행하고 종료하는 것을 알 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan06_04_1.png" style="display: block; margin: auto;">

<img src="{{ site.url }}/images/2017-02-11/Leviathan06_04_2.png" style="display: block; margin: auto;">

해당 숫자와 같은 숫자를 넣는다면 leviathan7 권한의 쉘을 얻을 수 있고 "cat /etc/leviathan_pass/leviathan7" 명령어를 통해 leviathan7의 password를 얻을 수 있다.

<img src="{{ site.url }}/images/2017-02-11/Leviathan06_05.png" style="display: block; margin: auto;">

# Leviathan07
ssh leviathan7@leviathan.labs.overthewire.org // password _________

### CONGRATULATIONS
Well Done, you seem to have used a *nix system before, now try something more serious.
