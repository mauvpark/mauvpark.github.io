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

### D. SELECT DISTINCT - 중복된 열 제거

테이블에서 질의 데이터를 추출할 때 중복된 열이 발생하는데 `SELECT` 구문에서 `DISTINCT`를 사용함으로써 중복을 제거할 수 있다.

<p align="center">
  <img src="https://www.mysqltutorial.org/wp-content/uploads/2021/07/MySQL-Distinct.svg" width="450" title="field">
</p>

- 한 열에 여러 개의 `NULL`이 존재하는 경우 `DISTINCT`는 이 또한 중복으로 판단하기 때문에 중복된 `NULL`을 제거하고 한 개의 `NUlL`만 반환한다.
- 여러 개의 열이 주어지는 경우, `DISTINCT`는 여러 개의 열에서 중복되는 데이터를 종합적으로 고려하여 결과를 반환한다.

~~~~sql
SELECT DISTINCT
    state, city
FROM
    customers
WHERE
    state IS NOT NULL
ORDER BY 
    state, 
    city;
~~~~

output: 

| state         | city           |
|---------------|----------------|
| BC            | Tsawassen      |
| BC            | Vancouver      |
| CA            | Brisbane       |
| CA            | Burbank        |
| CA            | Burlingame     |
| CA            | Glendale       |
| CA            | Los Angeles    |
| CA            | Pasadena       |
| CA            | San Diego      |

...

위에서 중복된 state 값 중 고유의 city 값이 존재하는 경우 제거하지 않고 state와 city 두 개의 열에서 모두 중복되는 데이터만 제거한다. (49 rows => 37 rows)

### E. AND 연산자

MySQL은 내장된 Boolean type이 존재하지 않으므로 숫자 0을 `FALSE`로 그 외의 값을 `TRUE`로 사용한다. `AND` 연산자는 2 개 이상의 Boolean 표현을 통합해 `1`, `0` 혹은 `NULL`을 반환하는 논리 연산자이다.

- `A AND B`에서 `A`와 `B` 모두 `0`과 `NULL`이 아닐 경우 `1`을 반환한다.
- 만약 둘 중 하나가 `0`일 경우 `0`을 반환한다.
- 그 이외에는 `NULL`을 반환한다.
- 실사용에 있어서 `AND` 연산자는 `SELECT`, `UPDATE`, `DELETE` 구문의 `WHERE` 절에 조건을 다는 방식으로 사용된다.

|           | **TRUE**  | **FALSE** | **NULL**  |
|-----------|-----------|-----------|-----------|
| **TRUE**  | TRUE      | FALSE     | NULL      | 
| **FALSE** | FALSE     | FALSE     | FALSE     |
| **NULL**  | NULL      | FALSE     | NULL      |

~~~~sql
SELECT 1 AND 0, 0 AND 1, 0 AND 0, 0 AND NULL;
~~~~

output: 

| 1 AND 0 | 0 AND 1 | 0 AND 0 | 0 AND NULL |
|---------|---------|---------|------------|
|       0 |       0 |       0 |          0 |

~~~~sql
SELECT 1 = 0 AND 1 / 0 ;
~~~~

output: 

| 1 = 0 AND 1 / 0 |
|-----------------|
|               0 |

위에서 MySQL은 `1 = 0`부분만 검사하는데 그 이유는 이 부분만 검사해도 전체 값이 0이 될 것이라는 것을 알 수 있기 때문이다. (`1/0`에서 zero error가 발생함에도 불구하고: `1/0`만 실행할 경우 wanring 에러가 뜨며 `NULL`을 반환함)

### F. OR 연산자

|           | **TRUE**  | **FALSE** | **NULL**  |
|-----------|-----------|-----------|-----------|
| **TRUE**  | TRUE      | TRUE      | TRUE      | 
| **FALSE** | TRUE      | FALSE     | NULL      |
| **NULL**  | TRUE      | NULL      | NULL      |

~~~~sql
SELECT 1 OR NULL, 0 OR NULL, NULL or NULL;
~~~~

output:

| 1 OR NULL | 0 OR NULL | NULL or NULL |
|-----------|-----------|--------------|
|         1 |      NULL |         NULL |

**연산자 우선**

`AND`와 `OR` 연산자가 모두 포함되어 있는 표현구(expression)의 경우, `AND`가 `OR`보다 우선한다. 따라서 MySQL은 `AND` 연산자를 먼저 평가한 후에 `OR` 연산자를 평가한다.

~~~~sql
SELECT 1 OR 0 AND 0;
~~~~

output:

| 1 OR 0 AND 0 |
|--------------|
|            1 |

`OR` 연산자를 먼저 평가하고자 하는 경우 괄호()를 포함한다.

~~~~sql
SELECT (1 OR 0) AND 0;
~~~~

output:

| (1 OR 0) AND 0 |
|----------------|
|              0 |

~~~~sql
SELECT   
	customername, 
	country, 
	creditLimit
FROM   
	customers
WHERE(country = 'USA'
		OR country = 'France')
	  AND creditlimit > 100000;
~~~~

output: country가 `France`혹은 `USA`면서 creditlimit의 조건을 충족하는 데이터

| customername                 | country | creditLimit |
|------------------------------|---------|-------------|
| La Rochelle Gifts            | France  |   118200.00 |
| Mini Gifts Distributors Ltd. | USA     |   210500.00 |
| Land of Toys Inc.            | USA     |   114900.00 |
| Saveley & Henriot, Co.       | France  |   123900.00 |
| Muscle Machine Inc           | USA     |   138500.00 |
| Diecast Classics Inc.        | USA     |   100600.00 |
| Collectable Mini Designs Co. | USA     |   105000.00 |
| Marta's Replicas Co.         | USA     |   123700.00 |
| Mini Classics                | USA     |   102700.00 |
| Corporate Gift Ideas Co.     | USA     |   105000.00 |
| Online Diecast Creations Co. | USA     |   114200.00 |

~~~~sql
SELECT    
    customername, 
    country, 
    creditLimit
FROM    
    customers
WHERE 
    country = 'USA'
    OR country = 'France'
    AND creditlimit > 100000;
~~~~

output: country가 `USA`거나 creditLimit이 100,000 이상이면서 country가 `France`인 데이터

| customername                 | country | creditLimit |
|------------------------------|---------|-------------|
| Signal Gift Stores           | USA     |    71800.00 |
| La Rochelle Gifts            | France  |   118200.00 |
| Mini Gifts Distributors Ltd. | USA     |   210500.00 |
| Mini Wheels Co.              | USA     |    64600.00 |
| Land of Toys Inc.            | USA     |   114900.00 |
| Saveley & Henriot, Co.       | France  |   123900.00 |
| Muscle Machine Inc           | USA     |   138500.00 |
| Diecast Classics Inc.        | USA     |   100600.00 |
| Technics Stores Inc.         | USA     |    84600.00 |
| American Souvenirs Inc       | USA     |        0.00 |

...

### G. IN 연산자

`IN` 연산자는 임의의 값이 list의 값들과 매칭되는지를 확인한다.

- `value`값이 `NULL`일 경우 결과값은 `NULL`을 반환한다.
- `value`값이 list의 어떤 값과도 일치하지 않고, list에 `NULL`이 존재하는 경우 `NULL`을 반환한다.

~~~~sql
value IN (value1, value2, value3,...)
~~~~

위의 식에서 `value`가 `IN` 이하의 값들 중 어느 하나와 매칭되는 경우 1(true)을 반환하고 그렇지 않으면 0(false)을 반환한다. 

~~~~sql
SELECT 2 IN (1, 2, 3);
~~~~

output: 2의 값이 list 중 하나에 속하므로 true인 1일 반환한다.

| 2 IN (1,2,3) |
|--------------|
|            1 |

~~~~sql
SELECT 
    officeCode, 
    city, 
    phone, 
    country
FROM
    offices
WHERE
    country IN ('USA' , 'France');
~~~~

output: `country = 'USA' OR country = 'France'`과 동일한 결과를 반환한다.

| officeCode | city          | phone           | country |
|------------|---------------|-----------------|---------|
| 1          | San Francisco | +1 650 219 4782 | USA     |
| 2          | Boston        | +1 215 837 0825 | USA     |
| 3          | NYC           | +1 212 555 3000 | USA     |
| 4          | Paris         | +33 14 723 4404 | France  |

### H. NOT IN 연산자

`NOT IN` 연산자는 임의의 값이 list의 어떤 값들 과도 일치하지 않을 때 어느 하나를 반환하고 그렇지 않으면 0을 반환한다. 기술적으로 아래의 값과 같다.

~~~~sql
NOT (value = value1 OR value = value2 OR value = valu3)
~~~~

~~~~sql
SELECT 1 NOT IN (1,2,3);
~~~~

output: 

| 1 NOT IN (1,2,3) |
|------------------|
|                0 |

~~~~sql
SELECT 0 NOT IN (1,2,3,NULL);
SELECT NULL NOT IN (1,2,3,NULL);
SELECT NULL NOT IN (1,2,3);
~~~~

output: `NULL`

~~~~sql
SELECT 
    officeCode, 
    city, 
    phone
FROM
    offices
WHERE
    country NOT IN ('USA' , 'France')
ORDER BY 
    city;
~~~~

output:

| officeCode | city   | phone            |
|------------|--------|------------------|
| 7          | London | +44 20 7877 2041 |
| 6          | Sydney | +61 2 9264 2451  |
| 5          | Tokyo  | +81 33 224 5000  |

### I. BETWEEN 연산자

~~~~sql
value BETWEEN low AND high;
~~~~

`BETWEEN` 연산자는 임의의 값이 범위 안에 존재하는지 여부를 확인하는 논리 연산자다. 범위 안에 존재할 경우 1(true)을 반환하고 그렇지 않을 경우 0(false)을 반환한다. 만약, `value`, `low` 혹은 `high` 중 하나가 `NULL`일 경우 연산자는 `NULL`을 반환한다.

~~~~sql
SELECT 
   orderNumber,
   requiredDate,
   status
FROM 
   orders
WHERE 
   requireddate BETWEEN 
     CAST('2003-01-01' AS DATE) AND 
     CAST('2003-01-31' AS DATE);
~~~~

output:

| orderNumber | requiredDate | status  |
|-------------|--------------|---------|
|       10100 | 2003-01-13   | Shipped |
|       10101 | 2003-01-18   | Shipped |
|       10102 | 2003-01-18   | Shipped |

**NOT BETWEEN**

`BETWEEN` 연산자 앞에 `NOT`이 붙어 BETWEEN의 연산값의 역을 반환한다.

### J. LIKE 연산자

`LIKE` 연산자는 임의의 문자열이 특정 패턴을 포함하는지 여부를 테스트하는 논리 연산자다.

~~~~sql
expression LIKE pattern ESCAPE escape_character
~~~~

`expression`이 `pattern`에 일치하게 되면 `LIKE` 연산자는 `1(true)`를 반환하고 그렇지 않으면 `0(false)`를 반환한다.

| Wildcard | Description      |
|----------|------------------|
| `%`      | 0 ~ 여러개의 문자 |
| `_`      | 1 개 문자        |

예를 들어, `s%`는 `sun`과 `six`와 같이 `s`로 시작하는 어떤 문자열이라도 반환한다. `se_`는 `see`와 `sea`와 같이 `se` 다음에 하나의 추가 character가 오는 문자열을 반환한다. 

pattern이 와일드카드 character를 포함하고 있고 이것을 일반적 character로 다루길 원할 때는 `ESCAPE` 절을 사용할 수 있다.

일반적으로 `LIKE` 연산자는 `SELECT`, `DELETE` 그리고 `UPDATE` 구문의 `WHERE` 절에서 사용할 수 있다.

~~~~sql
SELECT 
    employeeNumber, 
    lastName, 
    firstName
FROM
    employees
WHERE
    lastname LIKE '%on%';
~~~~

output: `on` 부분 문자열을 포함하는 모든 데이터를 찾는다.

| employeeNumber | lastName  | firstName |
|----------------|-----------|-----------|
|           1056 | Patterson | Mary      |
|           1088 | Patterson | William   |
|           1102 | Bondur    | Gerard    |
|           1166 | Thompson  | Leslie    |
|           1216 | Patterson | Steve     |
|           1337 | Bondur    | Loui      |
|           1504 | Jones     | Barry     |

~~~~sql
SELECT 
    employeeNumber, 
    lastName, 
    firstName
FROM
    employees
WHERE
    firstname LIKE 'M_ry';
~~~~

output:

| employeeNumber | lastName  | firstName |
|----------------|-----------|-----------|
|           1056 | Patterson | Mary      |

**NOT LIKE 연산자**

특정 패턴에 매칭되는 데이터를 제외한 정보를 얻을 수 있다.

~~~~sql
SELECT 
    employeeNumber, 
    lastName, 
    firstName
FROM
    employees
WHERE
    lastName NOT LIKE 'B%';
~~~~

output: `B%` 패턴에 맞는 데이터를 제외한 정보를 제공한다.

| employeeNumber | lastName  | firstName |
|----------------|-----------|-----------|
|           1002 | Murphy    | Diane     |
|           1056 | Patterson | Mary      |
|           1076 | Firrelli  | Jeff      |
|           1088 | Patterson | William   |
|           1165 | Jennings  | Leslie    |
|           1166 | Thompson  | Leslie    |
|           1188 | Firrelli  | Julie     |
|           1216 | Patterson | Steve     |
|           1286 | Tseng     | Foon Yue  |
|           1323 | Vanauf    | George    |
|           1370 | Hernandez | Gerard    |
|           1401 | Castillo  | Pamela    |
|           1504 | Jones     | Barry     |
|           1611 | Fixter    | Andy      |
|           1612 | Marsh     | Peter     |
|           1619 | King      | Tom       |
|           1621 | Nishi     | Mami      |
|           1625 | Kato      | Yoshimi   |
|           1702 | Gerard    | Martin    |

**ESCAPE 절과 LIKE 연산자**

때때로 `10%`, `_20` 등과 같은 데이터를 처리해야 하는 순간이 있는데 해당 특수문자들은 모두 와일드카드로 쓰이고 있기 때문에 `ESCAPE` 처리를 해 일반 문자로 보이도록 변경해주어야 한다. 만약 escape 문자를 명시적으로 지정하지 않으면 백슬래시(`\`)가 기본 escape 문자가 된다.

예를 들어, 만약 `_20`을 포함하는 데이터를 찾을 경우에 `%\_20%`을 사용할 수 있다.

~~~~sql
SELECT 
    productCode, 
    productName
FROM
    products
WHERE
    productCode LIKE '%\_20%';
~~~~

output: 기본 escape 문자(`\`)

| productCode | productName                               |
|-------------|-------------------------------------------|
| S10_2016    | 1996 Moto Guzzi 1100i                     |
| S24_2000    | 1960 BSA Gold Star DBD34                  |
| S24_2011    | 18th century schooner                     |
| S24_2022    | 1938 Cadillac V-16 Presidential Limousine |
| S700_2047   | HMS Bounty                                |

~~~~sql
SELECT 
    productCode, 
    productName
FROM
    products
WHERE
    productCode LIKE '%$_20%' ESCAPE '$';
~~~~

output: 지정 escape 문자(`$`)

| productCode | productName                               |
|-------------|-------------------------------------------|
| S10_2016    | 1996 Moto Guzzi 1100i                     |
| S24_2000    | 1960 BSA Gold Star DBD34                  |
| S24_2011    | 18th century schooner                     |
| S24_2022    | 1938 Cadillac V-16 Presidential Limousine |
| S700_2047   | HMS Bounty                                |

## 3. Error 해결

| 에러명                                                                                  | 해결                                                                |
| --------------------------------------------------------------------------------------- | -----------------------------------------------------------------  |
| ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)      | 비밀번호가 설정된 상태로 mysql 로그인 시 `-p` 태그를 붙여줘야 함       |
| 비밀번호 설정이 안됐거나 새로 설정이 필요한 경우 | `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new-password';` |

## 4. 주요 Terminal 명령어

`sudo systemctl status mysql`: mysql 서버 스타트
`sudo mysql -u root -p`: mysql 서버 연결
`show databases;`: 서버 연결 후 최근 서버에서의 모든 데이터베이스를 보여줌