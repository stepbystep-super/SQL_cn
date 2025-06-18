# SQL_cn

📊 SQL 执行顺序（核心记忆点）
- FROM
- WHERE（对原始数据进行过滤）
- GROUP BY（分组）
- HAVING（对分组后的结果过滤）
- SELECT（最终选择输出列）
- ORDER BY（排序）
  
| 分类    | 常用语法                                   | 说明                       |
| ----- | -------------------------------------- | ------------------------ |
| 基础查询  | `SELECT ... FROM ...`                  | 基本查询                     |
| 条件筛选  | `WHERE col = val`                      | 过滤原始数据                   |
| 分组统计  | `GROUP BY col`                         | 与 `COUNT()`, `AVG()` 一起用 |
| 排序    | `ORDER BY col DESC`                    | 支持多个字段排序                 |
| 去重    | `SELECT DISTINCT col`                  | 去重查询                     |
| 联表    | `INNER / LEFT JOIN`                    | 多表连接                     |
| 子查询   | `SELECT ... WHERE col IN (SELECT ...)` | 嵌套查询                     |
| 窗口函数  | `RANK() OVER (PARTITION ON/ORDER BY...)`                    | 排名、分组等                   |
| 条件列显示 | `CASE WHEN ... THEN ... ELSE ... END`  | 动态判断逻辑                   |

✅  实用函数 & 数据处理技巧
- 字符串处理：CONCAT(), SUBSTRING(), TRIM()

- 时间处理：DATE_PART(), NOW(), DATEDIFF()

- 条件判断：CASE WHEN, COALESCE(), NULLIF()

✅  常用窗口函数模板

-- ① 按字段分组内部排名
RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS dept_rank

-- ② 累计汇总
SUM(sales) OVER (PARTITION BY region ORDER BY month) AS running_total

-- ③ 判断唯一性
COUNT(*) OVER (PARTITION BY some_value) AS value_freq

| 类别       | 内容                                                  |
| -------- | --------------------------------------------------- |
| SQL 聚合函数 | `COUNT`, `SUM`, `AVG`, `CEIL`, `FLOOR`, `ROUND` 等使用 |
| 字符串处理    | `REPLACE`, `LOWER`, `SUBSTR`, `TRIM`, `CAST` 用法     |
| 思路转换     | 用表达式构造“错的工资”与“对的工资”之间的差异                            |
| 输出格式控制   | 拼接句子、加小写、加句号，符合输出要求                                 |
| 排序 & 分组  | `GROUP BY`, `ORDER BY COUNT(*)`, 处理输出顺序             |

| 功能    | 推荐函数                        |   |               |
| ----- | --------------------------- | - | ------------- |
| 平均值   | `AVG()`                     |   |               |
| 拼接    | 'concatenate'                |   | `或`CONCAT()\` |
| 类型转换  | `CAST(... AS INTEGER/CHAR)` |   |               |
| 替换字符  | `REPLACE()`                 |   |               |
| 字符串截取 | `SUBSTR(col, start, len)`   |   |               |
| 去掉空格  | `TRIM()`                    |   |               |
| 向上取整  | `CEIL()`                    |   |               |
举例说明下拼接/concatenate:
```sql
'... ' || col || '...'
```
比如 我要打出这一句话： There are a total of 3 doctors. 
用|| 而不是+

```sql
SELECT 
  'There are a total of ' || COUNT(*) || ' ' || LOWER(Occupation) || 's.'
FROM OCCUPATIONS
GROUP BY Occupation;
```

| 易错点                        | 正确方式                                        |   |     |   |                    |
| -------------------------- | ------------------------------------------- | - | --- | - | ------------------ |
| `AVERAGE()` 错写             | 应该是 `AVG()`                                 |   |     |   |                    |
| `SUBSTITUTE()` 用错          | 应该是 `REPLACE()`                             |   |     |   |                    |
| 拼接 SQL 输出句子                | 用 concat(a,b) 或者 ||                                  |   | col |   | '...'`或`CONCAT()\` |
| `CAST(... AS CHAR)` 有空格    | 加 `TRIM()` 处理                               |   |     |   |                    |
| `COUNT(*)` vs `COUNT(col)` | `COUNT(*)` 会统计 NULL 行，`COUNT(col)` 不统计 NULL |   |     |   |                    |
| SQL 函数参数顺序                 | 比如 `CEIL(错的AVG - 正确AVG)` 必须顺序正确，否则结果符号反了    |   |     |   |                    |


## 📝 题目简述

有两个表：`Students`（姓名 + 分数） 和 `Grades`（分数段对应的等级）。  
要求输出 `Name`, `Grade`, `Marks`，排序规则如下：
- Grade >= 8：显示真实 Name，按 Name 升序
- Grade < 8：Name 显示为 `'NULL'`，按 Marks 升序
- 所有结果先按 Grade 降序

## ✅ 我的实现（Db2 兼容）

```sql
WITH TB AS (
  SELECT s.Name, s.Marks, g.Grade
  FROM Students AS s
  LEFT JOIN Grades AS g
    ON s.Marks BETWEEN g.Min_Mark AND g.Max_Mark
)
SELECT 
  CASE WHEN Grade >= 8 THEN Name ELSE 'NULL' END AS Name,
  Grade,
  Marks
FROM TB
ORDER BY 
  Grade DESC,
  Name ASC,
  CASE WHEN Grade < 8 THEN Marks ELSE NULL END ASC;
```
忘记题目出处了随便看看，看懂就好。
```sql
WITH Ordered AS (
  SELECT LAT_N,
         ROW_NUMBER() OVER (ORDER BY LAT_N) AS rn,
         COUNT(*) OVER () AS cnt
  FROM STATION
)
SELECT ROUND(AVG(LAT_N), 4)
FROM Ordered
WHERE rn IN ((cnt + 1)/2, (cnt + 2)/2);
```
```sql
WITH TB AS (
  SELECT hacker_id, COUNT(*) AS total
  FROM Challenges
  GROUP BY hacker_id
),
Freq AS (
  SELECT total, COUNT(*) AS freq
  FROM TB
  GROUP BY total
)
SELECT *
FROM TB
JOIN Freq ON TB.total = Freq.total
WHERE freq = 1;
```
```sql
SELECT name, dept, salary,
  RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS dept_rank
FROM Employees;
```
下面这个一个table三个层次还蛮好看的，很喜欢那个1=1 的做法，值得记住记住记住
```sql
WITH TB AS (
  SELECT hacker_id, name, COUNT(*) AS total
  FROM Hackers h
  JOIN Challenges c ON h.hacker_id = c.hacker_id
  GROUP BY hacker_id, name
),
Freq AS (
  SELECT total, COUNT(*) AS freq
  FROM TB
  GROUP BY total
),
MaxVal AS (
  SELECT MAX(total) AS max_total FROM TB
)
SELECT t.hacker_id, t.name, t.total
FROM TB t
JOIN Freq f ON t.total = f.total
JOIN MaxVal m ON 1=1
WHERE t.total = m.max_total OR f.freq = 1
ORDER BY t.total DESC, t.hacker_id;
```
体会下下面rank()  row_number()的区别 
| 函数名            | 是否并列  | 是否跳号  | 举例      |
| -------------- | ----- | ----- | ------- |
| `RANK()`       | ✅ 是并列 | ✅ 会跳号 | 1, 1, 3 |
| `DENSE_RANK()` | ✅ 是并列 | ❌ 不跳号 | 1, 1, 2 |
| `ROW_NUMBER()` | ❌ 不并列 | ❌ 不跳号 | 1, 2, 3 |
```sql
RANK() OVER (ORDER BY total_challenges DESC)
DENSE_RANK() OVER (ORDER BY total_challenges DESC)
ROW_NUMBER() OVER (ORDER BY total_challenges DESC)
```
count() 一定是和group by 连用的
-- 错误示范：
```sql
SELECT dept, COUNT(*) 
FROM employee 
WHERE COUNT(*) > 5 -- ❌ 错误！不能在 WHERE 里用聚合

-- 正确写法：
SELECT dept, COUNT(*) 
FROM employee 
GROUP BY dept
HAVING COUNT(*) > 5
```

📌 题目 1：找出每个部门工资第二高的人
描述：
给定员工表 employee(emp_id, name, dept, salary)，找出每个部门工资第二高的员工。

思路：
使用 DENSE_RANK() 或 ROW_NUMBER() + PARTITION BY

SQL 答案（简化）：

```sql
SELECT dept, name, salary
FROM (
  SELECT *, 
         DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS rk
  FROM employee
) t
WHERE rk = 2
````
📌 题目 2：连续出现 3 天及以上的活跃用户
表结构：user_activity(user_id, activity_date)

思路：
用 LAG() 或 DATE_DIFF + 分组技巧，构造连续活动段

核心 SQL 技术：窗口函数 + 自定义组号

框架参考：
```sql
SELECT user_id
FROM (
  SELECT user_id,
         activity_date,
         ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY activity_date) AS rn
  FROM user_activity
) t
GROUP BY user_id, DATE_SUB(activity_date, INTERVAL rn DAY)
HAVING COUNT(*) >= 3
```
📌 题目 3：员工工资高于公司平均的员工
```sql
SELECT name, salary 
FROM employee
WHERE salary > (
    SELECT AVG(salary) 
    FROM employee
)
```
小技巧： 嵌套子查询是常规方式，但也可以用 CTE 或窗口函数优化
输出：There are a total of 3 doctors.（注意格式、s、小写、句号）

```sql

SELECT 'There are a total of ' || COUNT(*) || ' ' || LOWER(Occupation) || 's.'
FROM OCCUPATIONS
GROUP BY Occupation
ORDER BY COUNT(*), Occupation;
```
2. 🧮 平均工资差题（“0 键打不出来”）
目标：求 平均(错误工资) - 平均(正确工资)，再取上整

```sql
SELECT CEIL(
    AVG(CAST(REPLACE(TRIM(CAST(Salary AS CHAR)), '0', '') AS INTEGER)) 
    - AVG(Salary)
)
FROM EMPLOYEES;
```
REPLACE(..., '0', '')：去掉 0

TRIM(...)：去除空格避免转换错误

CEIL(...)：向上取整

易错点：表达式顺序不能写反

3. 🧠 拼接名字和职位缩写
Alice(D)

```sql
SELECT Name || '(' || SUBSTR(Occupation, 1, 1) || ')'
FROM OCCUPATIONS
ORDER BY Name;
```
这里学了新的语句fetch first row only
还有union
'''sql
(
SELECT CITY, LENGTH(CITY)
FROM STATION
ORDER BY LENGTH(CITY) ASC, CITY ASC
FETCH FIRST 1 ROWS ONLY
)
UNION
(
SELECT CITY, LENGTH(CITY)
FROM STATION
ORDER BY LENGTH(CITY) DESC, CITY ASC
FETCH FIRST 1 ROWS ONLY
);
'''



