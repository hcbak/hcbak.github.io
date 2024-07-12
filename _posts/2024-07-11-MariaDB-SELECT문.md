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

-- menu 테이블의 category 컬럼 조회
SELECT category FROM menu;

-- '*': 모든 컬럼
SELECT * FROM menu;

-- AS: 별칭
SELECT category AS cat FROM menu;


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

## JOIN
```sql
-- JOIN: 서로 다른 테이블을 중복되는 컬럼으로 결합

-- INNER JOIN: ON 뒤에 기준이 되는 컬럼을 지정하고 일치하는 컬럼을 기준으로 결합한다.
SELECT * FROM menu INNER JOIN sub_menu ON menu.category = sub_menu.category;
SELECT * FROM menu JOIN sub_menu ON menu.category = sub_menu.category; -- INNER 생략 가능

-- 컬럼 명이 같은 경우 USING으로 묶을 수 있다.
SELECT * FROM menu JOIN sub_menu USING (category);


-- LEFT JOIN: 왼쪽 테이블을 기준으로 오른쪽 테이블을 결합한다.
SELECT * FROM menu LEFT JOIN sub_menu ON menu.category = sub_menu.category;


-- RIGHT JOIN: 오른쪽 테이블을 기준으로 왼쪽 테이블을 결합한다.
SELECT * FROM menu RIGHT JOIN sub_menu ON menu.category = sub_menu.category;


-- SELF JOIN: 같은 테이블 결합
SELECT a.category, b.category FROM menu a JOIN menu b USING (category);
```

## GROUP BY
```sql
-- GROUP BY: 컬럼을 같은 값 끼리 그룹화한다.

SELECT category FROM menu GROUP BY category;

-- 다양한 함수로 어떻게 그룹화했는지 확인할 수 있다.
SELECT category, COUNT(*) FROM menu GROUP BY category; -- 그룹화한 갯수
SELECT category, SUM(*) FROM menu GROUP BY category;   -- 그룹화한 숫자의 합
SELECT category, AVG(*) FROM menu GROUP BY category;   -- 그룹화한 숫자의 평균


-- HAVING: 그룹화 한 결과에 조건을 걸 수 있다.
SELECT category, COUNT(*) FROM menu GROUP BY category HAVING COUNT(*) > 5;


-- WITH ROLLUP: 마지막 행에 중간 집계를 넣어준다.
SELECT category, COUNT(*) FROM menu GROUP BY category WITH ROLLUP; -- 맨 아래 행에 전체 COUNT 열의 합이 포함된 행을 추가한다.
```

## SubQuery
```sql
-- SubQuery: 쿼리로 가공한 테이블을 다른 쿼리에서 사용하고 싶은 경우에 쓰는 기법이다.

SELECT * FROM (SELECT * FROM menu) AS a
-- 해석: menu 테이블 전체를 출력한 테이블을 전체 출력
-- SubQuery는 다양한 곳에서 사용할 수 있으나 FROM 절에서 사용한 경우 AS로 별칭 지정 필수


```

## SET OPERATOR
```sql


```