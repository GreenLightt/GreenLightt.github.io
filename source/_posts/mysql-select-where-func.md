---
title: Mysql 用在 select 和 where 子句中的函数
date: 2018-01-08 01:09:41
description: 整理 Mysql 用在 select 和 where 子句中的函数
tags:
- Mysql
categories:
---

转自 [MySQL中文参考手册 7.4 节 用在SELECT和WHERE子句中的函数](http://www.yesky.com/imagesnew/software/mysql/manual_Reference.html#Functions)

# 算术函数

**加法**

```
select 3+5;  // 输出 8
```


**减法**

```
select 3-5;  // 输出 -2
```

**乘法**
```
select 3*5;  // 输出 15

select 18014398509481984*18014398509481984.0;  // 输出 324518553658426726783156020576256.0

select 18014398509481984*18014398509481984;  // 输出 0 因为 整数乘积的结果超过用BIGINT计算的 64 位范围
```

**除法**

```
select 3/5;  // 输出 0.6

select 102/(1-1);  // 输出 NULL
```


# 位函数

**位与**

```
select 29 & 15;  // 输出 13
```

**位或**

```
select 29 | 15;  // 输出 31
```

**左移位一个长(BIGINT)数字。**

```
select 1 << 2;  // 输出 4
```

**右移位一个长(BIGINT)数字。**

```
select 4 >> 2;  // 输出 1
```

**颠倒所有的位。**

```
select 5 & ~1;  // 输出 4
```

**二进制数中包含1的个数**

```
select BIT_COUNT(29);  // 输出 4
```

# 逻辑运算

**逻辑与**
如果任何一个参数是 0 或 `NULL`，返回0，否则返回1。
```
select 1 && NULL;  // 输出 0
select 1 && 0;  // 输出 0
```

**逻辑或**
如果任何一个参数不是 0 并且不 `NULL`，返回 1。
```
select 1 || 0;  // 输出 1
select 1 || NULL;  // 输出 1
select 0 || 0;  // 输出 0
```

**逻辑非**
如果参数是 0，返回 1，否则返回 0。例外： `NOT NULL` 返回 `NULL`。
```
select NOT 1;  // 输出 0
select NOT NULL;  // 输出 NULL
select ! (1+1);  // 输出 0
select ! 1+1;  // 输出 1
```

# 比较运算
`MySQL` 使用下列规则执行比较：

- 如果一个或两个参数是 `NULL`，比较的结果是 `NULL`，除了 `<=>` 操作符。
- 如果在比较中操作的两个参数是字符串，他们作为字符串被比较。
- 如果两个参数是整数，他们作为整数被比较。
- 十六进制的值如果不与一个数字比较，则被当作二进制字符串。
- 如果参数之一是一个 `TIMESTAMP` 或 `DATETIME` 列而其他参数是一个常数，在比较执行前，常数被转换为一个时间标记。这样做是为了对 `ODBC` 更友好。
- 在所有其他的情况下，参数作为浮点(实数)数字被比较。

**等于**

```
select 1 = 0;  // 输出 0
select '0' = 0; // 输出 1
select '0.0' = 0; // 输出 1
select '0.01' = 0; // 输出 0
select '.01' = 0.01; // 输出 1
select 1 = null; // 输出 null

```

**安全等于**

和 `=` 不一样， `null` 会被纳入可比较；

```
select 1 = null; // 输出 0
```

**不等于**

```
select '.01' <> '0.01'; // 输出 1
select .01 <> '0.01'; // 输出 0
select 'zapp' <> 'zappp'; // 输出 1
```

**小于或等于**

```
select 0.1 <= 2; // 输出 1
```

**小于**

```
select 2 < 2; // 输出 0
```

**大于或等于**

```
select 2 >= 2; // 输出 1
```


**大于**

```
select 2 > 2; // 输出 0
```


**NULL 和 非 NULL**

```
select 1 IS NULL, 0 IS NULL, NULL IS NULL:
// 输出 0 0 1

select 1 IS NOT NULL, 0 IS NOT NULL, NULL IS NOT NULL;
// 输出 1 1 0

select ISNULL(1+1); // 输出 0
select ISNULL(1/0); // 输出 1
```

**BETWEEN AND**

这等价于表达式(`min <= expr AND expr <= max`)。

```
select 1 BETWEEN 2 AND 3;  // 输出 0
select 'b' BETWEEN 'a' AND 'c';  // 输出 1
select 2 BETWEEN 2 AND '3';  // 输出 1
select 2 BETWEEN 2 AND 'x-3';  // 输出 0
```

**IN 和 NOT IN**

```
select 2 IN (0,3,5,'wefwf'); // 输出 0
select 'wefwf' IN (0,3,5,'wefwf'); // 输出 1 
```

**COALESCE(list)**

返回 `list` 中第一个非 `NULL` 的单元。

```
select COALESCE(NULL,1); // 输出  1
select COALESCE(NULL,NULL,NULL); // 输出 NULL
```

**INTERVAL(N,N1,N2,N3,...)**

如果 `N< N1`，返回 0，如果 `N< N2`，返回 1 等等。所有的参数被当作整数。为了函数能正确地工作，它要求 `N1<N2<N3< ...<Nn`。这是因为使用二进制搜索(很快)。

```
select INTERVAL(23, 1, 15, 17, 30, 44, 200); // 输出 3
select INTERVAL(10, 1, 10, 100, 1000); // 输出 2
select INTERVAL(22, 23, 30, 44, 200); // 输出 0  
```

# 字符串比较函数

**LIKE 与 NOT LIKE**

- `%`: 匹配任何数目的字符，甚至零个字符
- `_`: 精确匹配一个字符

```
select 'David!' LIKE 'David_'; // 输出  1
select 'David!' LIKE '%D%v%'; // 输出  1
```

转义字符 `\`

```
select 'David!' LIKE 'David\_';  // 输出  0

select 'David_' LIKE 'David\_';  // 输出  1
```

或者指定不同的转义字符

```
select 'David_' LIKE 'David|_' ESCAPE '|'; // 输出  1
```

**REGEXP 和 NOT REGEXP**

```
select 'Monty!' REGEXP 'm%y%%'; // 输出  0
select 'Monty!' REGEXP '.*';  // 输出  1
select 'new*\n*line' REGEXP 'new\\*.\\*line'; // 输出   1
select "a" REGEXP "A", "a" REGEXP BINARY "A"; // 输出   1  0
```

**STRCMP(expr1,expr2)**

如果字符串相同，`STRCMP()` 返回 0，如果第一参数根据当前的排序次序小于第二个，返回 -1，否则返回 1。

```
select STRCMP('text', 'text2'); // 输出  -1
select STRCMP('text2', 'text'); // 输出  1
select STRCMP('text', 'text'); // 输出  0
```

# 类型转换运算符
`BINARY` 操作符强制跟随它后面的字符串为一个二进制字符串。即使列没被定义为 `BINARY` 或 `BLOB`，这是一个强制列比较区分大小写的简易方法。

```
select "a" = "A"; // 输出 1
select BINARY "a" = "A"; // 输出 0
```

# 控制流函数
**IFNULL(expr1,expr2)**
如果 `expr1` 不是 `NULL`，`IFNULL()` 返回 `expr1`，否则它返回 `expr2`。`IFNULL()` 返回一个数字或字符串值，取决于它被使用的上下文环境。

```
select IFNULL(1,0); // 输出  1
select IFNULL(0,10); // 输出  0
select IFNULL(1/0,10); // 输出 10
select IFNULL(1/0,'yes'); // 输出 'yes'
```

**IF(expr1,expr2,expr3)**

如果 `expr1` 是 `TRUE` (`expr1 <>0且expr1<>NULL`)，那么IF()返回expr2，否则它返回expr3。IF()返回一个数字或字符串值，取决于它被使用的上下文。

```
select IF(1>2,2,3); // 输出 3

select IF(1<2,'yes','no'); // 输出 'yes'

select IF(strcmp('test','test1'),'yes','no'); // 输出 'yes'
```

**CASE value WHEN [compare-value] THEN result [WHEN [compare-value] THEN result ...] [ELSE result] END**

```
SELECT CASE 1 WHEN 1 THEN "one" WHEN 2 THEN "two" ELSE "more" END; // 输出 "one"
```

**CASE WHEN [condition] THEN result [WHEN [condition] THEN result ...] [ELSE result] END**

```
SELECT CASE WHEN 1>0 THEN "true" ELSE "false" END; // 输出   "true"

SELECT CASE BINARY "B" when "a" then 1 when "b" then 2 END;  // 输出  NULL
```


# 数学函数

所有的数学函数在一个出错的情况下返回 `NULL`。

`-`
单目减。改变参数的符号。

```
select - 2; // 输出 -2
```

**ABS(X)**
返回`X`的绝对值。

```
select ABS(2); // 输出  2
select ABS(-32); // 输出  32
```


**SIGN(X)**
返回参数的符号，为 -1、0 或 1，取决于 `X` 是否是负数、零或正数。

```
select SIGN(-32); // 输出 -1
select SIGN(0); // 输出 0
select SIGN(234); // 输出 1
```


**MOD(N,M) 或 %**
模 (类似 `C` 中的%操作符)。返回 `N` 被 `M` 除的余数。

```
select MOD(234, 10); // 输出  4
select 253 % 7; // 输出 1
select MOD(29,9); // 输出  2
```

**FLOOR(X)**
返回不大于 `X` 的最大整数值。

```
select FLOOR(1.23); // 输出 1
select FLOOR(-1.23); // 输出  -2
```

**CEILING(X)**
返回不小于 `X` 的最小整数值。

```
select CEILING(1.23);  // 输出 2
select CEILING(-1.23);  // 输出 -1
```

**ROUND(X) 和 ROUND(X,D)**

返回参数 `X` 的四舍五入的一个整数。

```
select ROUND(-1.23);// 输出 -1
select ROUND(-1.58);// 输出 -2
select ROUND(1.58); // 输出 2

select ROUND(1.298, 1); // 输出   1.3
select ROUND(1.298, 0); // 输出  1
```


**EXP(X)**
返回值 `e`（自然对数的底）的 `X` 次方。

```
select EXP(2); // 输出  7.389056
select EXP(-2); // 输出 0.135335
```

**LOG(X)**
返回 `X` 的自然对数。

```
select LOG(2); // 输出  0.693147
select LOG(-2); // 输出  NULL
```

**LOG10(X)**
返回 `X` 的以10为底的对数。

```
select LOG10(2); // 输出  0.301030
select LOG10(100); // 输出  2.000000
select LOG10(-100); // 输出   NULL
```

**POW(X,Y)**
和 `POWER(X,Y)` 等价
```
select POW(2,2); // 输出  4.000000
select POW(2,-2); // 输出  0.250000
```

**SQRT(X)**
返回非负数 `X` 的平方根。

```
select SQRT(4); // 输出   2.000000
select SQRT(20); // 输出  4.472136
```

**PI()**
返回 `PI` 的值（圆周率）。

```
select PI(); // 输出 3.141593
```

**COS(X)**
返回 `X` 的余弦, 在这里 `X` 以弧度给出。

```
select COS(PI()); // 输出 -1.000000
```

**SIN(X)**
返回 `X` 的正弦值，在此 `X` 以弧度给出。

```
select SIN(PI()); // 输出 0.000000
```

**TAN(X)**
返回 `X` 的正切值，在此 `X` 以弧度给出。

```
select TAN(PI()+1); // 输出 1.557408
```

**ACOS(X)**
返回 `X` 反余弦，即其余弦值是 `X`。如果 `X` 不在-1到1的范围，返回 `NULL`。

```
select ACOS(1); // 输出  0.000000
select ACOS(1.0001); // 输出  NULL
select ACOS(0); // 输出  1.570796
```

**ASIN(X)**
返回 `X` 反正弦值，即其正弦值是 `X`。L如果 `X` 不在-1到1的范围，返回 `NULL`。

```
select ASIN(0.2); //  输出   0.201358
select ASIN('foo'); // 输出   0.000000
```

**ATAN(X)**
返回 `X` 的反正切值，即其正切值是`X`。

```
select ATAN(2); // 输出 1.107149
select ATAN(-2); // 输出 -1.107149
```

**ATAN2(X,Y)**
返回 2 个变量 `X` 和 `Y` 的反正切。它类似于计算 `Y/X` 的反正切，除了两个参数的符号被用来决定结果的象限。

```
select ATAN(-2,2); // 输出  -0.785398
select ATAN(PI(),0); // 输出  1.570796 
```

**COT(X)**
返回 `X` 的余切。

```
select COT(12); // 输出   -1.57267341
select COT(0); // 输出   NULL
```

**RAND() 和 RAND(N)**
返回在范围 0 到 1.0 内的随机浮点值。如果一个整数参数 `N` 被指定，它被用作种子值。

```
select RAND();  // 输出  0.5925
select RAND(20); // 输出  0.1811
select RAND(20); // 输出  0.1811
select RAND(); // 输出 0.2079
select RAND(); // 输出  0.7888
```

**LEAST(X,Y,...)**

有 2 和 2 个以上的参数，返回最小(最小值)的参数。

```
select LEAST(2,0); // 输出 0
select LEAST(34.0,3.0,5.0,767.0); // 输出 3.0
select LEAST("B","A","C"); // 输出 "A"
```

**GREATEST(X,Y,...)**
返回最大(最大值)的参数。

```
select GREATEST(2,0); // 输出  2
select GREATEST(34.0,3.0,5.0,767.0);  // 输出 767.0
select GREATEST("B","A","C"); // 输出  "C"
```

**DEGREES(X)**
返回参数 `X`，从弧度变换为角度。

```
select DEGREES(PI()); // 输出 180.000000
```

**RADIANS(X)**
返回参数 `X`，从角度变换为弧度。

```
select RADIANS(90); // 输出   1.570796
```

**TRUNCATE(X,D)**
返回数字 `X`，截断为 `D` 位小数。如果 `D` 为 0，结果将没有小数点或小数部分。

```
select TRUNCATE(1.223,1); // 输出  1.2
select TRUNCATE(1.999,1); // 输出  1.9
select TRUNCATE(1.999,0); // 输出  1
```

# 字符串函数

对于针对字符串位置的操作，第一个位置被标记为 1。

**ASCII(str)**
返回字符串 `str` 的最左面字符的 `ASCII` 代码值。如果 `str` 是空字符串，返回 0。如果 `str` 是 `NULL`，返回 `NULL`。

```
select ASCII('2'); // 输出    50
select ASCII(2); // 输出  50
select ASCII('dx'); // 输出   100
```

**ORD(str)**
如果字符串 `str` 最左面字符是一个多字节字符，通过以格式((`first byte ASCII code)*256+(second byte ASCII code))[*256+third byte ASCII code...`]返回字符的 `ASCII` 代码值来返回多字节字符代码。如果最左面的字符不是一个多字节字符。返回与 `ASCII()` 函数返回的相同值。

```
select ORD('2'); // 输出  50
```

**CONV(N,from_base,to_base)**
在不同的数字基之间变换数字。返回数字 `N` 的字符串数字，从 `from_base` 基变换为 `to_base` 基，如果任何参数是`NULL`，返回 `NULL`。

```
select CONV("a",16,2); // 输出   '1010'
select CONV("6E",18,8); // 输出  '172'
select CONV(-17,10,-18); // 输出  '-H'
select CONV(10+"10"+'10'+0xa,10,10); // 输出  '40'
```

**BIN(N)**
返回二进制值 `N` 的一个字符串表示，在此 `N` 是一个长整数(`BIGINT`)数字，这等价于`CONV(N,10,2)`。如果 `N` 是 `NULL`，返回 `NULL`。

```
select BIN(12); // 输出 '1100'
```

**OCT(N)**
返回八进制值 `N` 的一个字符串的表示，在此N是一个长整型数字，这等价于 `CONV(N,10,8)`。如果 `N` 是 `NULL`，返回 `NULL`。

```
select OCT(12); // 输出  '14'
```

**HEX(N)**
返回十六进制值 `N` 一个字符串的表示，在此 `N` 是一个长整型(`BIGINT`)数字，这等价于 `CONV(N,10,16)` 。如果 `N` 是 `NULL`，返回 `NULL`。

```
select HEX(255); // 输出 'FF'
```

**CHAR(N,...)**
`CHAR()` 将参数解释为整数并且返回由这些整数的 `ASCII` 代码字符组成的一个字符串。`NULL` 值被跳过。

```
select CHAR(77,121,83,81,'76'); // 输出  'MySQL'
select CHAR(77,77.3,'77.3'); // 输出  'MMM'
```

**CONCAT(str1,str2,...)**
返回来自于参数连结的字符串。如果任何参数是 `NULL`，返回 `NULL`。

```
select CONCAT('My', 'S', 'QL'); // 输出   'MySQL'
select CONCAT('My', NULL, 'QL');// 输出    NULL
select CONCAT(14.3); // 输出   '14.3'
```

**LENGTH(str)**
等同于 `OCTET_LENGTH(str)` `CHAR_LENGTH(str)` `CHARACTER_LENGTH(str)` 

返回字符串str的长度。

```
select LENGTH('text'); // 输出   4
select OCTET_LENGTH('text'); // 输出  4
```

**LOCATE(substr,str)**
等同于 `POSITION(substr IN str)`

返回子串 `substr` 在字符串 `str` 第一个出现的位置，如果 `substr` 不是在 `str` 里面，返回 0.

```
select LOCATE('bar', 'foobarbar'); // 输出  4
select LOCATE('xbar', 'foobar'); // 输出  0
```


**LOCATE(substr,str,pos)**
返回子串 `substr` 在字符串 `str` 第一个出现的位置，从位置 `pos` 开始。如果 `substr` 不是在 `str` 里面，返回 0。

```
select LOCATE('bar', 'foobarbar',5); // 输出  7
```

**INSTR(str,substr)**
返回子串 `substr` 在字符串 `str` 中的第一个出现的位置。这与有 2 个参数形式的 `LOCATE()` 相同，除了参数被颠倒。

```
select INSTR('foobarbar', 'bar'); // 输出   4
select INSTR('xbar', 'foobar'); // 输出  0
```

**LPAD(str,len,padstr)**
返回字符串 `str`，左面用字符串 `padstr` 填补直到 `str` 是 `len` 个字符长。

```
select LPAD('hi',4,'??'); // 输出  '??hi'
```

**RPAD(str,len,padstr)**
返回字符串`str`，右面用字符串 `padstr` 填补直到 `str` 是 `len` 个字符长。  

```
select RPAD('hi',5,'??'); // 输出  'hi???'
```

**LEFT(str,len)**
返回字符串 `str` 的最左面 `len` 个字符。

```
select LEFT('foobarbar', 5); // 输出   'fooba'
```

**RIGHT(str,len)**
返回字符串 `str` 的最右面 `len` 个字符。

```
select RIGHT('foobarbar', 4); // 输出  'rbar'
```

**SUBSTRING(str,pos,len)**
等同于 `SUBSTRING(str FROM pos FOR len)` `MID(str,pos,len)`

从字符串 `str` 返回一个 `len` 个字符的子串，从位置 `pos` 开始。

```
select SUBSTRING('Quadratically',5,6); // 输出  'ratica'
```

**SUBSTRING(str,pos)**
等同于 `SUBSTRING(str FROM pos)`

从字符串 `str` 的起始位置 `pos` 返回一个子串。

```
select SUBSTRING('Quadratically',5); // 输出  'ratically'
select SUBSTRING('foobarbar' FROM 4); // 输出  'barbar'
```

**SUBSTRING_INDEX(str,delim,count)**
返回从字符串 `str` 的第 `count` 个出现的分隔符 `delim` 之后的子串。如果 `count` 是正数，返回最后的分隔符到左边(从左边数) 的所有字符。如果 `count` 是负数，返回最后的分隔符到右边的所有字符(从右边数)。

```
select SUBSTRING_INDEX('www.mysql.com', '.', 2); // 输出  'www.mysql'
select SUBSTRING_INDEX('www.mysql.com', '.', -2); // 输出 'mysql.com'
```

**LTRIM(str)**
返回删除了其前置空格字符的字符串 `str`。

```
select LTRIM('  barbar'); // 输出  'barbar'
```

**RTRIM(str)**
返回删除了其拖后空格字符的字符串 `str`。

```
select RTRIM('barbar   '); // 输出   'barbar'
```

**TRIM([[BOTH | LEADING | TRAILING] [remstr] FROM] str)**
返回字符串 `str`，其所有 `remstr` 前缀或后缀被删除了。如果没有修饰符 `BOTH`、`LEADING` 或 `TRAILING` 给出，`BOTH` 被假定。如果 `remstr` 没被指定，空格被删除。


```
select TRIM('  bar   '); // 输出   'bar'
select TRIM(LEADING 'x' FROM 'xxxbarxxx'); // 输出   'barxxx'
select TRIM(BOTH 'x' FROM 'xxxbarxxx'); // 输出   'bar'
select TRIM(TRAILING 'xyz' FROM 'barxxyz'); // 输出   'barx'
```

**SOUNDEX(str)**
返回 `str` 的一个同音字符串。听起来“大致相同”的 2 个字符串应该有相同的同音字符串。一个“标准”的同音字符串长是 4 个字符，但是 `SOUNDEX()` 函数返回一个任意长的字符串。你可以在结果上使用 `SUBSTRING()` 得到一个“标准”的 同音串。所有非数字字母字符在给定的字符串中被忽略。所有在 `A-Z` 之外的字符国际字母被当作元音。

```
select SOUNDEX('Hello');  // 输出 'H400'
select SOUNDEX('Quadratically');  // 输出  'Q36324'
```

**SPACE(N)**
返回由 `N` 个空格字符组成的一个字符串。

```
select SPACE(6); // 输出  '      '
```

**REPLACE(str,from_str,to_str)**
返回字符串 `str`，其字符串 `from_str` 的所有出现由字符串 `to_str` 代替。

```
select REPLACE('www.mysql.com', 'w', 'Ww'); // 输出  'WwWwWw.mysql.com'
```

**REPEAT(str,count)**
返回由重复 `countTimes` 次的字符串 `str` 组成的一个字符串。如果 `count <= 0`，返回一个空字符串。如果 `str` 或 `count` 是 `NULL`，返回 `NULL`。

```
select REPEAT('MySQL', 3); // 输出 'MySQLMySQLMySQL'
```

**REVERSE(str)**
返回颠倒字符顺序的字符串 `str`

```
select REVERSE('abc'); // 输出  'cba
```

**INSERT(str,pos,len,newstr)**
返回字符串 `str`，在位置 `pos` 起始的子串且 `len` 个字符长得子串由字符串 `newstr` 代替。

```
select INSERT('Quadratic', 3, 4, 'What'); // 输出 'QuWhattic'
```

**ELT(N,str1,str2,str3,...)**
如果 `N= 1`，返回 `str1`，如果 `N= 2`，返回 `str2`，等等。如果 `N `小于 1 或大于参数个数，返回 `NULL`。`ELT()` 是 `FIELD()` 反运算。

```
select ELT(1, 'ej', 'Heja', 'hej', 'foo'); // 输出  'ej'
select ELT(4, 'ej', 'Heja', 'hej', 'foo'); // 输出  'foo'
```

**FIELD(str,str1,str2,str3,...)**
返回 `str` 在 `str1` , `str2`, `str3`, ...清单的索引。如果 `str` 没找到，返回 0。`FIELD()` 是 `ELT()` 反运算。

```
select FIELD('ej', 'Hej', 'ej', 'Heja', 'hej', 'foo'); // 输出  2
select FIELD('fo', 'Hej', 'ej', 'Heja', 'hej', 'foo'); // 输出  0
```

**FIND_IN_SET(str,strlist)**
如果字符串 `str` 在由 `N` 子串组成的表 `strlist` 之中，返回一个 1 到 `N` 的值。一个字符串表是被“,”分隔的子串组成的一个字符串。

```
 SELECT FIND_IN_SET('b','a,b,c,d'); // 输出   2
```

**MAKE_SET(bits,str1,str2,...)**
返回一个集合 (包含由“,”字符分隔的子串组成的一个字符串)，由相应的位在 `bits` 集合中的的字符串组成。`str1` 对应于位 0，`str2` 对应位 1，等等。在 `str1`, `str2`, ...中的 `NULL` 串不添加到结果中。

```
SELECT MAKE_SET(1,'a','b','c'); // 输出  'a'
SELECT MAKE_SET(1 | 4,'hello','nice','world'); // 输出  'hello,world'
SELECT MAKE_SET(0,'a','b','c'); // 输出  ''
```

**EXPORT_SET(bits,on,off,[separator,[number_of_bits]])**
返回一个字符串，在这里对于在“ `bits` ”中设定每一位，你得到一个“`on`”字符串，并且对于每个复位(` reset `)的位，你得到一个“ `off` ”字符串。每个字符串用“ `separator` ”分隔(缺省“,”)，并且只有“ `bits` ”的“`number_of_bits` ” (缺省64)位被使用。

```
select EXPORT_SET(5,'Y','N',',',4)  // 输出 Y,N,Y,N 
```

**LCASE(str)**
等同于 `LOWER(str)`

返回字符串 `str`，根据当前字符集映射(缺省是`ISO-8859-1 Latin1`)把所有的字符改变成小写。


```
select LCASE('QUADRATICALLY'); // 输出 'quadratically'
```



**UCASE(str)**
等同于 `UPPER(str)`

返回字符串  `str`，根据当前字符集映射(缺省是 `ISO-8859-1 Latin1` )把所有的字符改变成大写。

```
select UCASE('Hej'); // 输出  'HEJ'
```

**LOAD_FILE(file_name)**
读入文件并且作为一个字符串返回文件内容。文件必须在服务器上，你必须指定到文件的完整路径名，而且你必须有 `file` 权限。文件必须所有内容都是可读的并且小于 `max_allowed_packet` 。如果文件不存在或由于上面原因之一不能被读出，函数返回 `NULL`。

```
 UPDATE table_name
 SET blob_column=LOAD_FILE("/tmp/picture")
 WHERE id=1;
```

# 日期和时间函数

**DAYOFWEEK(date)**
返回日期 `date` 的星期索引(1=星期天，2=星期一, ……7=星期六)。

```
select DAYOFWEEK('1998-02-03'); // 输出 3
```

**WEEKDAY(date)**
返回 `date` 的星期索引(0=星期一，1=星期二, ……6= 星期天)。

```
select WEEKDAY('1997-10-04 22:23:00'); // 输出 5
select WEEKDAY('1997-11-05'); // 输出 2
```

**DAYOFMONTH(date)**
返回date的月份中日期，在1到31范围内。

```
select DAYOFMONTH('1998-02-03'); // 输出  3
```

**DAYOFYEAR(date)**
返回 `date` 在一年中的日数, 在 1 到 366 范围内。

```
select DAYOFYEAR('1998-02-03');  // 输出 34
```


**MONTH(date)**
返回 `date` 的月份，范围1到12。

```
select MONTH('1998-02-03'); // 输出  2
```

**DAYNAME(date)**
返回 `date` 的星期名字。

```
select DAYNAME("1998-02-05"); // 输出  'Thursday'
```

**MONTHNAME(date)**
返回 `date` 的月份名字。

```
select MONTHNAME("1998-02-05"); // 输出  'February'
```

**QUARTER(date)**
返回 `date` 一年中的季度，范围 1 到 4。

```
select QUARTER('98-04-01'); // 输出  2
```

**WEEK(date) 和 WEEK(date,first)**
对于星期天是一周的第一天的地方，有一个单个参数，返回 `date` 的周数，范围在 0 到 52。2 个参数形式 `WEEK()` 允许你指定星期是否开始于星期天或星期一。如果第二个参数是 0，星期从星期天开始，如果第二个参数是 1，从星期一开始。

```
select WEEK('1998-02-20'); // 输出   7
select WEEK('1998-02-20',0); // 输出   7
select WEEK('1998-02-20',1); // 输出  8
```

**YEAR(date)**
返回 `date` 的年份，范围在 1000 到 9999。

```
select YEAR('98-02-03'); // 输出  1998
```

**HOUR(time)**
返回 `time` 的小时，范围是 0 到 23。

```
select HOUR('10:05:03'); // 输出  10
```

**MINUTE(time)**
返回 `time` 的分钟，范围是 0 到 59。

```
select MINUTE('98-02-03 10:05:03'); // 输出   5
```

**SECOND(time)**
回来 `time` 的秒数，范围是0到59。

```
select SECOND('10:05:03'); // 输出   3
```

**PERIOD_ADD(P,N)**
增加 `N` 个月到阶段 `P`（以格式 `YYMM` 或 `YYYYMM`)。以格式 `YYYYMM` 返回值。注意阶段参数 `P` 不是日期值。

```
select PERIOD_ADD(9801,2);  // 输出  199803
```

**PERIOD_DIFF(P1,P2)**
返回在时期 `P1` 和 `P2` 之间月数，`P1` 和 `P2` 应该以格式 `YYMM` 或 `YYYYMM` 。注意，时期参数 `P1` 和 `P2` 不是日期值。

```
select PERIOD_DIFF(9802,199703); // 输出   11
```

**DATE_ADD(date,INTERVAL expr type) 和 DATE_SUB(date,INTERVAL expr type)**
等同于 `ADDDATE(date,INTERVAL expr type)` 和 `SUBDATE(date,INTERVAL expr type)`

你可以使用 `+` 和 `-` 而不是 `DATE_ADD()` 和 `DATE_SUB()`。（见例子） `date` 是一个指定开始日期的 `DATETIME` 或 `DATE` 值，`expr` 是指定加到开始日期或从开始日期减去的间隔值一个表达式， `expr` 是一个字符串；它可以以一个“`-`”开始表示负间隔。`type` 是一个关键词，指明表达式应该如何被解释。

下表显示了type和expr参数怎样被关联：

| `type` 值  |  含义 | 期望的 `expr` 格式  | 
|: --- :|: --- :|: --- :|
| `SECOND` | 秒 | `SECONDS` |
| `MINUTE` | 分钟 | `MINUTES` |
| `HOUR`  | 时间  | `HOURS` |
| `DAY`  | 天 | `DAYS` |
| `MONTH`  | 月 | `MONTHS` |
| `YEAR`  | 年 | `YEARS` |
| `MINUTE_SECOND`  | 分钟和秒  | `MINUTES:SECONDS` |
| `HOUR_MINUTE`  | 小时和分钟 | `HOURS:MINUTES` |
| `DAY_HOUR`  | 天和小时  | `DAYS HOURS` |
| `YEAR_MONTH`  | 年和月 | `YEARS-MONTHS` |
| `HOUR_SECOND`  | 小时, 分钟， | `HOURS:MINUTES:SECONDS` |
| `DAY_MINUTE`  | 天, 小时, 分钟 | `DAYS HOURS:MINUTES` |
| `DAY_SECOND`  | 天, 小时, 分钟, 秒 | `DAYS HOURS:MINUTES:SECONDS` |



```
SELECT "1997-12-31 23:59:59" + INTERVAL 1 SECOND; // 输出  1998-01-01 00:00:00
SELECT INTERVAL 1 DAY + "1997-12-31"; // 输出  1998-01-01
SELECT "1998-01-01" - INTERVAL 1 SECOND; // 输出  1997-12-31 23:59:59 
SELECT DATE_ADD("1997-12-31 23:59:59", INTERVAL 1 SECOND); // 输出  1998-01-01 00:00:00
SELECT DATE_ADD("1997-12-31 23:59:59", INTERVAL 1 DAY); // 输出  1998-01-01 23:59:59
SELECT DATE_ADD("1997-12-31 23:59:59", INTERVAL "1:1" MINUTE_SECOND); // 输出  1998-01-01 00:01:00
SELECT DATE_SUB("1998-01-01 00:00:00", INTERVAL "1 1:1:1" DAY_SECOND); // 输出  1997-12-30 22:58:59
SELECT DATE_ADD("1998-01-01 00:00:00", INTERVAL "-1 10" DAY_HOUR); // 输出  1997-12-30 14:00:00
SELECT DATE_SUB("1998-01-02", INTERVAL 31 DAY); // 输出  1997-12-02
SELECT EXTRACT(YEAR FROM "1999-07-02"); // 输出  1999
SELECT EXTRACT(YEAR_MONTH FROM "1999-07-02 01:02:03"); // 输出  199907
SELECT EXTRACT(DAY_MINUTE FROM "1999-07-02 01:02:03"); // 输出  20102
```

如果你指定太短的间隔值(不包括 `type` 关键词期望的间隔部分)，`MySQL` 假设你省掉了间隔值的最左面部分。例如，如果你指定一个 `type` 是 `DAY_SECOND`，值 `expr` 被希望有天、小时、分钟和秒部分。如果你象"1:10"这样指定值，`MySQL` 假设日子和小时部分是丢失的并且值代表分钟和秒。换句话说，"1:10" `DAY_SECOND` 以它等价于"1:10" `MINUTE_SECOND` 的方式解释，

**TO_DAYS(date)**
给出一个日期 `date`，返回一个天数(从 0 年的天数)。

```
select TO_DAYS(950501); // 输出  728779
mysql> select TO_DAYS('1997-10-07'); // 输出  729669
```

**FROM_DAYS(N)**
给出一个天数 `N`，返回一个 `DATE` 值。

```
select FROM_DAYS(729669); // 输出  '1997-10-07'
```

**DATE_FORMAT(date,format)**
根据 `format` 字符串格式化 `date` 值。下列修饰符可以被用在 `format` 字符串中：

| 修饰符 | 描述 |
|: --- :|: --- :|
|  `%M` | 月名字(`January……December`) |
|  `%W` | 星期名字(`Sunday……Saturday`) |
|  `%D` | 有英语前缀的月份的日期(`1st`, `2nd`, `3rd`, 等等。） |
|  `%Y` | 年, 数字, 4 位 |
|  `%y` | 年, 数字, 2 位 |
|  `%a` | 缩写的星期名字(`Sun……Sat`) |
|  `%d` | 月份中的天数, 数字(00……31) |
|  `%e` | 月份中的天数, 数字(0……31) |
|  `%m` | 月, 数字(01……12) |
|  `%c` | 月, 数字(1……12) |
|  `%b` | 缩写的月份名字(`Jan……Dec`) |
|  `%j` | 一年中的天数(001……366) |
|  `%H` | 小时(00……23) |
|  `%k` | 小时(0……23) |
|  `%h` | 小时(01……12) |
|  `%I` | 小时(01……12) |
|  `%l` | 小时(1……12) |
|  `%i` | 分钟, 数字(00……59) |
|  `%r` | 时间, `12` 小时(`hh:mm:ss [AP]M`)|
|  `%T` | 时间, `24` 小时(`hh:mm:ss`) |
|  `%S` | 秒(00……59) |
|  `%s` | 秒(00……59) |
|  `%p` | `AM` 或 `PM` |
|  `%w` | 一个星期中的天数(0=Sunday ……6=Saturday ）|
|  `%U` | 星期(0……52), 这里星期天是星期的第一天|
|  `%u` | 星期(0……52), 这里星期一是星期的第一天 |
|  `%%` | 一个文字“%”。 |

```
select DATE_FORMAT('1997-10-04 22:23:00', '%W %M %Y'); // 输出   'Saturday October 1997'
mysql> select DATE_FORMAT('1997-10-04 22:23:00', '%H:%i:%s'); // 输出   '22:23:00'
mysql> select DATE_FORMAT('1997-10-04 22:23:00', '%D %y %a %d %m %b %j'); // 输出   '4th 97 Sat 04 10 Oct 277'
mysql> select DATE_FORMAT('1997-10-04 22:23:00', '%H %k %I %r %T %S %w'); // 输出 '22 22 10 10:23:00 PM 22:23:00 00 6'
```

**TIME_FORMAT(time,format)**
这象上面的 `DATE_FORMAT()` 函数一样使用，但是 `format` 字符串只能包含处理小时、分钟和秒的那些格式修饰符。其他修饰符产生一个 `NULL` 值或0。


**CURDATE() 和 CURRENT_DATE**
以'`YYYY-MM-DD`'或 `YYYYMMDD` 格式返回今天日期值，取决于函数是在一个字符串还是数字上下文被使用。

```
select CURDATE(); // 输出  '1997-12-15'
select CURDATE() + 0; // 输出  19971215
```

**CURTIME() 和 CURRENT_TIME**
以'`HH:MM:SS`'或 `HHMMSS` 格式返回当前时间值

```
select CURTIME(); // 输出  '23:50:26'
select CURTIME() + 0; // 输出   235026
```

**NOW()**
等同于 `SYSDATE()` `CURRENT_TIMESTAMP`

以'`YYYY-MM-DD HH:MM:SS`'或 `YYYYMMDDHHMMSS` 格式返回当前的日期和时间，取决于函数是在一个字符串还是在数字的上下文被使用。

```
select NOW(); // 输出  '1997-12-15 23:50:26'
select NOW() + 0; // 输出  19971215235026
```

**UNIX_TIMESTAMP() 和 UNIX_TIMESTAMP(date)**
如果没有参数调用，返回一个 `Unix` 时间戳记(从'`1970-01-01 00:00:00`' `GMT` 开始的秒数)。如果 `UNIX_TIMESTAMP()` 用一个 `date` 参数被调用，它返回从'`1970-01-01 00:00:00`' `GMT` 开始的秒数值。`date` 可以是一个 `DATE` 字符串、一个 `DATETIME` 字符串、一个 `TIMESTAMP` 或以 `YYMMDD` 或 `YYYYMMDD` 格式的本地时间的一个数字。

```
select UNIX_TIMESTAMP(); // 输出    882226357
select UNIX_TIMESTAMP('1997-10-04 22:23:00'); // 输出   875996580
```

**FROM_UNIXTIME(unix_timestamp)**
以'`YYYY-MM-DD HH:MM:SS`'或 `YYYYMMDDHHMMSS` 格式返回 `unix_timestamp` 参数所表示的值

```
select FROM_UNIXTIME(875996580); // 输出  '1997-10-04 22:23:00'
select FROM_UNIXTIME(875996580) + 0; // 输出   19971004222300
```

**FROM_UNIXTIME(unix_timestamp,format)**
返回表示 `Unix` 时间标记的一个字符串，根据 `format` 字符串格式化。

```
select FROM_UNIXTIME(UNIX_TIMESTAMP(), '%Y %D %M %h:%i:%s %x'); // 输出  '1997 23rd December 03:43:30 x'
```

**SEC_TO_TIME(seconds)**
返回 `seconds` 参数，变换成小时、分钟和秒，值以'`HH:MM:SS`'或 `HHMMSS` 格式化

```
select SEC_TO_TIME(2378); // 输出   '00:39:38'
select SEC_TO_TIME(2378) + 0;  // 输出  3938
```

**TIME_TO_SEC(time)**
返回 `time` 参数，转换成秒。

```
select TIME_TO_SEC('22:23:00'); // 输出  80580
select TIME_TO_SEC('00:39:38');  // 输出  2378
```


# 其他函数

**DATABASE()**
返回当前的数据库名字。

```
select DATABASE();  // 输出 'test'
```

**USER()**
同 `SYSTEM_USER()` `SESSION_USER()`

返回当前 `MySQL` 用户名。

```
select USER(); // 输出  'davida@localhost'

select substring_index(USER(),"@",1); // 输出 'davida'
```

**PASSWORD(str)**
从纯文本口令 `str` 计算一个口令字符串。该函数被用于为了在 `user` 授权表的 `Password` 列中存储口令而加密 `MySQL` 口令。  `PASSWORD()` 加密是非可逆的。`PASSWORD()` 不以与 `Unix` 口令加密的相同的方法执行口令加密。

```
select PASSWORD('badpwd'); // 输出 '7f84554057dd964b'
```

**ENCRYPT(str[,salt])**
使用 `Unix crypt()` 系统调用加密 `str`。`salt` 参数应该是一个有 2 个字符的字符串

```
select ENCRYPT("hello"); // 输出  'VxuFAJXVARROc'
```

如果 `crypt()` 在你的系统上不可用，`ENCRYPT()` 总是返回 `NULL`。`ENCRYPT()` 只保留 `str` 起始 8 个字符而忽略所有其他，至少在某些系统上是这样。这将由底层的 `crypt()` 系统调用的行为决定。

**ENCODE(str,pass_str)**
使用 `pass_str` 作为口令加密 `str`。为了解密结果，使用 `DECODE()`。结果是一个二进制字符串，如果你想要在列中保存它，使用一个 `BLOB` 列类型。


**DECODE(crypt_str,pass_str)**
使用 `pass_str` 作为口令解密加密的字符串 `crypt_str`。`crypt_str` 应该是一个由 `ENCODE()` 返回的字符串。


**MD5(string)**
对字符串计算 `MD5` 校验和。值作为一个 32 长的十六进制数字被返回可以，例如用作哈希(`hash`)键。

```
select MD5("testing"); // 输出  'ae2b1fca515949e5d54fb22b8ed95575'
```

**LAST_INSERT_ID([expr])**
返回被插入一个 `AUTO_INCREMENT` 列的最后一个自动产生的值。

```
select LAST_INSERT_ID();  // 输出  195
```

**FORMAT(X,D)**
格式化数字 `X` 为类似于格式'`#,###,###.##`'，四舍五入到 `D` 为小数。如果 `D` 为 0，结果将没有小数点和小数部分。

```
select FORMAT(12332.123456, 4);  // 输出 '12,332.1235'
select FORMAT(12332.1,4);  // 输出   '12,332.1000'
select FORMAT(12332.2,0);  // 输出  '12,332'
```

**VERSION()**
返回表明 `MySQL` 服务器版本的一个字符串。

```
select VERSION(); // 输出  '3.22.19b-log'
```

**GET_LOCK(str,timeout)**
试图获得由字符串 `str` 给定的一个名字的锁定，第二个 `timeout` 为超时。如果锁定成功获得，返回 1，如果尝试超时了，返回 0，或如果发生一个错误，返回 `NULL` (例如从存储器溢出或线程用 `mysqladmin kill` 被杀死)。当你执行 `RELEASE_LOCK()` 时、执行一个新的 `GET_LOCK()` 或线程终止时，一个锁定被释放。该函数可以用来实现应用锁或模拟记录锁，它阻止其他客户用同样名字的锁定请求；赞成一个给定的锁定字符串名字的客户可以使用字符串执行子协作建议的锁定。

```
select GET_LOCK("lock1",10); // 输出  1
select GET_LOCK("lock2",10); // 输出  1
select RELEASE_LOCK("lock2"); // 输出  1
select RELEASE_LOCK("lock1"); // 输出 NULL
```

注意，第二个 `RELEASE_LOCK()` 调用返回 `NULL`，因为锁" `lock1` "自动地被第二个 `GET_LOCK()` 调用释放。

**RELEASE_LOCK(str)**
释放字符串 `str` 命名的通过 `GET_LOCK()` 获得的锁。如果锁被释放，返回 1，如果锁没被这个线程锁定(在此情况下锁没被释放)返回 0，并且如果命名的锁不存在，返回 `NULL`。如果锁从来没有通过调用 `GET_LOCK()` 获得或如果它已经被释放了，锁将不存在。

**BENCHMARK(count,expr)**

`BENCHMARK()` 函数重复 `countTimes` 次执行表达式 `expr`，它可以用于计时 `MySQL` 处理表达式有多快。结果值总是 0。意欲用于 `mysql` 客户，它报告查询的执行时间。

```
select BENCHMARK(1000000,encode("hello","goodbye"));
```

报告的时间是客户端的经过时间，不是在服务器端的 `CPU` 时间。执行 `BENCHMARK()` 若干次可能是明智的，并且注意服务器机器的负载有多重来解释结果。




# 与 GROUP BY 子句一起使用的函数

如果你在不包含 `GROUP BY` 子句的一个语句中使用聚合函数，它等价于聚合所有行。

**COUNT(expr)**
返回由一个 `SELECT` 语句检索出来的行的非 `NULL` 值的数目。

```
select student.student_name,COUNT(*)
from student,course
where student.student_id=course.student_id
GROUP BY student_name;
```

**COUNT(DISTINCT expr,[expr...])**
返回一个不同值的数目。

```
select COUNT(DISTINCT results) from student;
```

**AVG(expr)**
返回 `expr` 的平均值。

```
select student_name, AVG(test_score)
from student
GROUP BY student_name;
```

**MIN(expr) 和 MAX(expr)** 
返回 `expr` 的最小或最大值。`MIN()` 和 `MAX()` 可以有一个字符串参数；在这种的情况下，他们返回最小或最大的字符串值。

```
select student_name, MIN(test_score), MAX(test_score)
from student
GROUP BY student_name;
```

**SUM(expr)**
返回 `expr` 的和。注意，如果返回的集合没有行，它返回 `NULL`！

**STD(expr) 和 STDDEV(expr)**
返回 `expr` 标准差(`deviation`)。这是对 `ANSI SQL` 的扩展。该函数的形式`STDDEV()` 是提供与 `Oracle` 的兼容性。

**BIT_OR(expr)**
返回 `expr` 里所有位的位或。计算用 64 位(`BIGINT`)精度进行。

**BIT_AND(expr)**

返回 `expr` 里所有位的位与。计算用 64 位(`BIGINT`)精度进行。
