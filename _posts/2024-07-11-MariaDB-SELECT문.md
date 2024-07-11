---
title: "MariaDB SELECT문"
author: hcbak
date: 2024-07-11 17:42:00 +0900
categories: [DBMS, MariaDB]
---

- 테스트 환경  
Ubuntu 24.04  
MariaDB 10.11.8

## SELECT
```sql
-- SELECT: FROM 테이블의 컬럼을 조회
SELECT category FROM menu;

-- '*': 모든 컬럼
SELECT * FROM menu;


-- DISTINCT: 중복 제거
SELECT DISTINCT category FROM menu;


-- LIMIT: 출력 행 조절
SELECT * FROM menu LIMIT 10;


-- SELECT문 단독 사용
SELECT 1 + 2;
SELECT 3 * 4;

-- 내장 함수 사용 가능
SELECT NOW();
SELECT CONCAT('CON', 'CAT');

-- FIELD 
SELECT FIELD('A', 'A', 'B', 'C'); 
```

## ORDER BY
```sql
-- ORDER BY: SELECT문의 결과를 정렬

-- category를 기준으로 오름차순 정렬
SELECT * FROM menu ORDER BY category;

-- 내림차순 정렬
SELECT * FROM menu ORDER BY category DESC;

-- -(하이픈): NULL값과의 상관관계가 있는데 그냥 넣었다 뺐다 해보자...
SELECT * FROM menu ORDER BY -category;
```

## WHERE
```sql
-- WHERE: 조건에 맞는 결과 출력

-- 특정 column 값 지정
SELECT * FROM menu WHERE category = 'main';
-- main 테이블 중에 category가 main인 데이터 출력

-- 특정 column 값 이외의 값
SELECT * FROM menu WHERE category != 'main';
SELECT * FROM menu WHERE category <> 'main';

-- 숫자 비교
SELECT * FROM menu WHERE ca_index > 10;

-- 논리 연산자
SELECT * FROM menu WHERE category = 'main' AND ca_index > 10;
SELECT * FROM menu WHERE category = 'main' OR ca_index > 10;


-- BETWEEN _ AND: 두 값 사이의 값 지정
SELECT * FROM menu WHERE ca_index BETWEEN 5 AND 10;
SELECT * FROM menu WHERE ca_index NOT BETWEEN 5 AND 10; -- 선택 반전


-- LIKE: 문자열 패턴 검색
SELECT * FROM menu WHERE category like 'ma%';
-- %는 아무거나 있던지 말던지
-- _는 하나의 빈 자리 허용 (ma_n으로 main 찾아짐)
SELECT * FROM menu WHERE category NOT like '%ma%'; -- 선택 반전


-- IN: 괄호 안의 값 중에 일치하는지 확인
SELECT * FROM menu WHERE ca_index IN (1, 2, 3);
SELECT * FROM menu WHERE ca_index NOT IN (1, 2, 3); -- 선택 반전


-- IS NULL: NULL값 찾기
SELECT * FROM menu WHERE ca_index IS NULL;
SELECT * FROM menu WHERE ca_index IS NOT NULL; -- 선택 반전
```