---
layout: post
title: "SQL injection을 하기 전에"
headline: Web
modified: 2017-04-09
categories: chinnie web sql injection
comments: true
featured: true
---

모의해킹 보고서에서 SQL injection 공격 방법을 따라해보던 중.. SQL 기본 문법을 그동안 잊고 있었음을 깨닫고 기본 문법 정리!

## 기본 Query 와 SQL injection을 위해 사용되는 주요 Query!

먼저, SQL은 DDL, DML, DCL을 하기 위한 언어로 나뉜다는 점~

- DDL(Data Definition Language)
- DML(Data Manipulation Language)
- DCL(Data Control Language)

DB 공부할 때, 학교에서 시험보려고 DDL은 CREATE, DROP~~ DML은 SELECT, FROM ~~ 외웠던 기억이 있지만 시험보고 나면 다 까먹는 것을 알고 있지. 그래서 실제 DB 생성할 때 순서대로 한번 적어보려고 함

### 1.1 기본 쿼리문
데이터를 DB에 어떤 형태로, 어떤 내용을 넣을지 먼저 정의해 줘야함.

- **CREATE** 로 하는 일.

Step1: database 이름을 붙여 만들어 준다.

    CREATE database 룰루랄라 -> 룰루랄라 라는 database가 만들어짐

Step2: database를 사용할게!

    use 룰루랄라

Step3: database 안에 table들을 만들자. table을 만들 때에는 table안에 들어갈 data들의 생김새를 정해줘야지.

    CREATE table 신난다 (username varchar(10) null)

    -> 신난다 table을 만들거야. 신난다 table에는 username이라는 항목이 있어. 여기에는 변하는 문자 형태로 최대 10자리까지 들어갈 수 있고, 아무런 값이 들어가지 않아도 되!)

Step4: 이제 database와 table이 만들어 졌고, table에 username을 넣을 수 있는 상태가 되었음!


- **INSERT** 로 하는 일.

Step1: 이제 형태가 정해졌으니, 실제 '값'을 넣기.

    INSERT into 신난다 (username) values ('윤똥이');
    -> 신난다 table에 있는 username 항목에 넣을거야. 윤똥이 라는 값을

- **UPDATE** 로 하는 일.

Step1: INSERT로 값을 넣었는데 수정하고 싶어!

    UPDATE 신난다 SET username='똥그리'
    -> 신난다 table에 있는 윤똥이를 똥그리로 바꿀거야.

- **DELETE** 로 하는 일.

Step1: '값' 삭제하기.

    DELETE FROM 신난다 WHERE username='똥그리';
    -> 이제 난 똥그리가 아니니까, 신난다 테이블에 username이 똥그리인 것을 삭제해줘!


- **WHERE** 로 하는 일.

Step1: 내가 원하는 조건의 값을 찾아줘!

    DELETE FROM 신난다 WHERE id>2000; -> 신난다 table에 id가 2000 이상인 내용을 삭제해줘!

Step2: WHERE은 LIKE와 함께 쓰이기도 하지! LIKE는 특정 문자 or 다수의 불특정 문자를 포함하는 것들을 찾아줘.

++특정 문자를 포함하는 경우++

    DELETE FROM 신난다 WHERE username='똥%';
    -> 신난다 table에 있는 username이 똥으로 시작하는 모든 data를 지워줘! 똥쟁이, 똥그리, 똥똥이..

++불특정 문자를 포함하는 경우++

    DELETE FROM 신난다 WHERE username='똥_';
    -> 신난다 table에 있는 username 이 똥으로 시작하는 두 글자 모든 data를 지워줘! 똥킥, 똥똥..

Step3: WHERE은 IN과 함께 쓰여. IN은 여러 개의 OR라고 보면 되.

    DELETE FROM 신난다 WHERE id IN (2009,2010,2011);
    -> 신난다 table에 id가 2009 이거나 2010 이거나 2011 인 data를 삭제해줘!

Step4: 우선 마지막! WHERE은 BETWEEN과 함께 쓰여. BETWEEN은 커플앱도 있듯이, '너와 나의 연결고리' 기능이야.

    DELETE FROM 신난다 WHERE id BETWEEN 2009 AND 2011; -> 신난다 table에 id가 2009~2011인 모든 data를 삭제해줘!


- **ALTER** 로 하는 일.

Step1: 이미 생성된 table의 구조를 바꿀 때 사용! table에 들어갈 항목 추가, 삭제, 항목 안의 유형 수정, 항목 이름 변경.

Step2: **ADD**, table에 들어갈 항목 추가!

    ALTER table 신난다 ADD (id varchar(10)); -> id라는 항목을 추가해줘!

Step3: **DROP COLUMN**, table에 있는 COLUMN을 삭제!

    ALTER table 신난다 DROP COLUMN username;


_DROP은 table삭제도 가능하기 때문에 table삭제와 COLUMN 삭제 잘 구분해서 사용하기!_

Step4: **MODIFY**, table에 있는 항목의 데이터 유형, Default 값, NOT NULL 제약조건에 대한 변경

    ALTER Table 신난다 MODIFY (username varchar(10) DEFAULT '2009' NOT NULL);

++MODIFY를 사용할 땐, 주의할 점이 있음++
1. 칼럼 크기를 늘릴 수 있지만, 줄일 수 없다는 점
2. 하지만, 칼럼에 NULL값만 가지고 있거나 행이 없다면 줄일 수 있지!
3. 칼럼이 NULL만 가지고 있다면 데이터 유형도 바꿀 수 있음!
4. 만약 DEFAULT 값을 바꾸면 변경 작업 이후 발생한 행에만 DEFAULT값이 들어감.
5. 칼럼에 NULL값이 들어가 있지 않다면, NOT NULL 조건을 추가할 수 있음.
