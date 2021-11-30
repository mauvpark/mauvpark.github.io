---
layout: post
title: "My SQL basic 메모"
date: 2021-11-30 20:42:00 +0900
parent: Sql
categories: sql, mysql, basic
nav_order: 2
comments: false
---

*2021-11-30 20:42 작성*

# My SQL basic 메모

## 1. SQL 정의
SQL은 Structured query language를 의미하고 다음의 세 가지를 포함한다.

1. 테이블, 뷰, 트리거, 저장된 절차 등을 의미하는 정의적 언어
2. 데이터 업데이트, 쿼리를 수행하는 조작적 언어
3. 유저가 특정 데이터베이스에 접근하는 것을 허용/불허용하는 통제적 언어

## 2. MySQL 명령어

### A. SELECT
| 명령어                       | 설명              |
| ---------------------------- | ---------------- |
| `SELECT 1 + 1`               | `2`              |
| `SELECT NOW()`               | `2021-11-30 15:54:24` |
| `SELECT CONCAT('John', ' ', 'Doe')` | `John Doe` |
| `SELECT select_list FROM dual` | `dual`은 dummy clause로 실제적 의미를 갖지 않는다. 실제 table을 지정하지 않고 `FROM`을 사용해야 할 경우 사용한다. |
| `SELECT expression AS column_alias;` = `SELECT expression column_alias;` (e.g. `SELECT CONCAT('Jane',' ','Doe') AS 'Full name';`, `SELECT CONCAT('John',' ','Doe') AS name;`) | `AS` 이하의 `column_alias`로 이름을 변경할 수 있다. |

### B. ORDER BY - 정렬
<p align="center">
  <img src="https://www.mysqltutorial.org/wp-content/uploads/2021/07/MySQL-ORDER-BY.svg" width="350" title="order by">
</p>

| 명령어                       | 설명              |
| ---------------------------- | ---------------- |
| `ORDER BY column1 ASC, column2 DESC;` | `column1`을 ascending 방식으로 정렬한 뒤 정렬값을 `column2`의 기준에 따라 다시 descending 방식으로 정렬(`column1`에서 정렬된 순서를 바꾸지 않으며 만약 동일값이 존재하면 그 안에서 `column2`의 기준을 적용해 정렬한다.) |
| `ORDER BY quantityOrdered * priceEach DESC;` | 연산 표현도 가능 |

~~~~sql
SELECT 
    contactLastname, 
    contactFirstname
FROM
    customers
ORDER BY 
	contactLastname DESC , 
	contactFirstname ASC;
~~~~

Output:
| contactLastname | contactFirstname |
|-----------------|------------------|
| Young           | Dorothy          |
| Young           | Jeff             |
| Young           | Julie            |
| Young           | Mary             |
| Yoshido         | Juri             |
| Walker          | Brydey           |
| Victorino       | Wendy            |
| Urs             | Braun            |
| Tseng           | Jerry            |
| Tonini          | Daniel           |
...

| 명령어                       | 설명              |
| ---------------------------- | ---------------- |
| `SELECT orderNumber, orderLineNumber, quantityOrdered * priceEach AS subtotal FROM orderdetails ORDER BY subtotal DESC;` | `AS`로 명칭이 변경된 경우 `ORDER BY`에서 사용 가능 |

<p align="center">
  <h3>FIELD()를 이용한 ORDER BY 👇</h3>
  <img src="https://www.mysqltutorial.org/wp-content/uploads/2009/12/orders_table.png" width="150" title="field">
</p>

| 명령어                       | 설명              |
| ---------------------------- | ---------------- |
| `FIELD('str', 'str1','str2','...');` e.g. `SELECT FIELD('B', 'A','B','C');` => `2` | `str1, str2, ...` 중 `str`의 위치값을 반환 |

~~~~sql
SELECT 
    orderNumber, status
FROM
    orders
ORDER BY FIELD(status,
        'In Process',
        'On Hold',
        'Cancelled',
        'Resolved',
        'Disputed',
        'Shipped');
~~~~

output:
| orderNumber | status     |
|-------------|------------|
|       10425 | In Process |
|       10421 | In Process |
|       10422 | In Process |
|       10420 | In Process |
|       10424 | In Process |
|       10423 | In Process |
|       10414 | On Hold    |
|       10401 | On Hold    |
|       10334 | On Hold    |
|       10407 | On Hold    |
...

각각의 위치값을 `ASC` 방식으로 출력하게 되면 위와 같은 결과값이 나온다.

**기타사항**
- `NULL`은 `non-NULL` 값보다 앞선다. 따라서 `ASC`로 정렬할 경우 `NULL`값이 가장 먼저 나열되고 `DESC`로 정렬할 경우 `NULL`값을 가진 데이터들은 가장 마지막에 위치한다.

### C. WHERE - Search condition

- `AND`, `OR` 그리고 `NOT`과 같은 논리 연산자를 사용해서 하나 혹은 여러개의 표현을 조합할 수 있다.
- `SELECT`외에도 `UPDATE` 혹은 `DELETE` 구문에서도 사용할 수 있다.
- 아래의 순서대로 진행된다.

<p align="center">
  <img src="https://www.mysqltutorial.org/wp-content/uploads/2021/07/MySQL-Where.svg" width="400" title="field">
</p>

~~~~sql
SELECT 
    lastname, 
    firstname, 
    jobtitle,
    officeCode
FROM
    employees
WHERE
    jobtitle = 'Sales Rep' AND 
    officeCode = 1;
~~~~

output: 
| lastname | firstname | jobtitle  | officeCode |
|----------|-----------|-----------|------------|
| Jennings | Leslie    | Sales Rep | 1          |
| Thompson | Leslie    | Sales Rep | 1          |
2 rows in set (0.00 sec)

- **`Between A AND B` 연산자**

~~~~sql
SELECT 
    firstName, 
    lastName, 
    officeCode
FROM
    employees
WHERE
    officeCode BETWEEN 1 AND 3
ORDER BY officeCode;
~~~~

output: 
| firstName | lastName  | officeCode |
|-----------|-----------|------------|
| Diane     | Murphy    | 1          |
| Mary      | Patterson | 1          |
| Jeff      | Firrelli  | 1          |
| Anthony   | Bow       | 1          |
| Leslie    | Jennings  | 1          |
| Leslie    | Thompson  | 1          |
| Julie     | Firrelli  | 2          |
| Steve     | Patterson | 2          |
| Foon Yue  | Tseng     | 3          |
| George    | Vanauf    | 3          |
10 rows in set (0.00 sec)

- `LIKE` 연산자: 특정 패턴에 `TRUE`를 반환하는 데이터를 추출

| Wildcard | Description      |
|----------|------------------|
| `%`      | 0 ~ 여러개의 문자 |
| `_`      | 1 개 문자        |

~~~~sql
SELECT 
    firstName, 
    lastName
FROM
    employees
WHERE
    lastName LIKE '%son'
ORDER BY firstName;
~~~~

output:
| firstName | lastName  |
|-----------|-----------|
| Leslie    | Thompson  |
| Mary      | Patterson |
| Steve     | Patterson |
| William   | Patterson |
4 rows in set (0.00 sec)

- `IN` 연산자: list 안에 있는 값들 중 한 개라도 매칭되는 데이터 추출

~~~~sql
SELECT 
    firstName, 
    lastName, 
    officeCode
FROM
    employees
WHERE
    officeCode IN (1, 2, 3)
ORDER BY 
    officeCode;
~~~~

output:
| firstName | lastName  | officeCode |
|-----------|-----------|------------|
| Diane     | Murphy    | 1          |
| Mary      | Patterson | 1          |
| Jeff      | Firrelli  | 1          |
| Anthony   | Bow       | 1          |
| Leslie    | Jennings  | 1          |
| Leslie    | Thompson  | 1          |
| Julie     | Firrelli  | 2          |
| Steve     | Patterson | 2          |
| Foon Yue  | Tseng     | 3          |
| George    | Vanauf    | 3          |
10 rows in set (0.00 sec)

- `IS NULL` 연산자: 값이 `NULL`인 데이터를 추출 (숫자 0이나 빈 문자열과는 다름)

~~~~sql
SELECT 
    lastName, 
    firstName, 
    reportsTo
FROM
    employees
WHERE
    reportsTo IS NULL;
~~~~

output:
| lastName | firstName | reportsTo |
|----------|-----------|-----------|
| Murphy   | Diane     |      NULL |
1 row in set (0.01 sec)

- 비교 연산자

| Operator     | Description |
|--------------|-------------|
| `=`          | Equal to. You can use it with almost any data type. |
| `<>` or `!=` | Not equal to |
| `<`          | Less than. You typically use it with numeric and date/time data types. |
| `>`          | Greater than. |
| `<=`         | Less than or equal to |
| `>=`         | Greater than or equal to |

~~~~sql
SELECT 
    lastname, 
    firstname, 
    jobtitle
FROM
    employees
WHERE
    jobtitle <> 'Sales Rep';
~~~~

output:
| lastname  | firstname | jobtitle             |
|-----------|-----------|----------------------|
| Murphy    | Diane     | President            |
| Patterson | Mary      | VP Sales             |
| Firrelli  | Jeff      | VP Marketing         |
| Patterson | William   | Sales Manager (APAC) |
| Bondur    | Gerard    | Sale Manager (EMEA)  |
| Bow       | Anthony   | Sales Manager (NA)   |
6 rows in set (0.00 sec)

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new-password';`: 비밀번호 변경

## 3. Error 해결

| 에러명                                                                                  | 해결                                                                |
| --------------------------------------------------------------------------------------- | -----------------------------------------------------------------  |
| ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)      | 비밀번호가 설정된 상태로 mysql 로그인 시 `-p` 태그를 붙여줘야 함       |

## 4. 주요 Terminal 명령어

`sudo systemctl status mysql`: mysql 서버 스타트
`sudo mysql -u root -p`: mysql 서버 연결
`show databases;`: 서버 연결 후 최근 서버에서의 모든 데이터베이스를 보여줌