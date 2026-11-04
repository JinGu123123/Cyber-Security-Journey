# DVWA-SQL Injection Medium级别

## 防御手段分析（从源码得知）

- 使用了`mysqli_real_escape_string()`对单引号等特殊字符进行转义（变成`\'`），导致字符型注入失效
- 但是，SQL 语句变成了`WHERE user_id = $id`，缺失了原本的单引号包裹
![SQL-Injection-ViewSource(Medium)](images/SQL-Injection-ViewSource(Medium).png)


## 绕过思路

- 因为是数字型（没有单引号），所以不需要闭合单引号
- 构造 Payload：`1 OR 1=1`，直接插入到数字部分，逻辑恒真
- 甚至可以直接`1 UNION SELECT user,password FROM users`进行联合查询
![Burp-SQL-Injection(Medium)1](Burp-SQL-Injection(Medium)1.png)

![Burp-SQL-Injection(Medium)2](Burp-SQL-Injection(Medium)2.png)

![Burp-SQL-Injection(Medium)3](Burp-SQL-Injection(Medium)3.png)

![Burp-SQL-Injection(Medium)4](Burp-SQL-Injection(Medium)4.png)


## 领悟

不严谨的修复不如不修复。如果防御者写了`WHERE user_id = '$id'`，Medium 级别可能真的防住了。缺失了单引号，防了等于没防。不过这毕竟是练习用的，更高等级的防御当然会使用更无解的方式