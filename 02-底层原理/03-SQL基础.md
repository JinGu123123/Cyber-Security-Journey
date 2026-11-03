# SQL注入
## 什么是SQL注入

SQL 注入（SQLi）是一种网络安全漏洞，允许攻击者干扰应用程序对其数据库的查询。这可能让攻击者查看他们通常无法检索的数据。这可能包括属于其他用户的数据，或应用程序可以访问的其他任何数据。在许多情况下，攻击者可以删除或修改这些数据，导致应用程序内容或行为持续发生变化
在某些情况下，攻击者可以升级SQL注入攻击，攻破底层服务器或其他后端基础设施。它还能使他们能够实施拒绝服务攻击


## 成功的SQL注入会产生什么影响

成功的 SQL 注入可能导致敏感数据被未经授权访问，例如：
- 密码
- 信用卡
- 个人用户信息
多年来，SQL 注入攻击已被用于许多高调的数据泄露事件。在某些情况下，攻击者可以获得持续的后门进入组织系统，导致长期的入侵，而这些漏洞可能在很长时间内都未被察觉


## 如何检测SQL注入漏洞

可以手动检测 SQL 注入，方法是对应用中的每个入口点进行系统化测试。为此，通常需要提交：
- 使用单引号字符，寻找错误或其他异常（'）
- 一些 SQL 专用语法，可以根据入口点的基础（原始）值和不同的值进行评估，并观察应用响应中的系统差异
- 布尔条件，如 和，并观察应用响应的差异（or 1=1 or 1=2）
- 这些有效载荷设计用于在 SQL 查询中执行时触发时间延迟，并寻找响应时间的差异
- OAST 载荷设计用于在 SQL 查询中执行时触发带外网络交互，并监控由此产生的交互


## 在查询的不同部分注入SQL的应用

大多数 SQL 注入漏洞都发生在查询的从句内。大多数有经验的测试人员都熟悉这种 SQL 注入方式。然而，SQL 注入漏洞可能出现在查询的任何位置，且存在于不同的查询类型中。 SQL 注入出现的其他地点包括：
- 在语句中，在更新后的值或句子中（update where）
- 在语句中，插入的值内（insert）
- 在语句中，在表或列名中（select）
- 在声明中，在条款内（select order by）


## SQL注入示例

存在许多 SQL 注入的漏洞、攻击和技术，这些都会出现在不同情况下。一些常见的 SQL 注入示例包括：
- 检索隐藏数据，可以修改 SQL 查询以返回更多结果
- 颠覆应用数据，可以更改查询以干扰应用逻辑
- UNION 工具，可以从不同的数据库表中获取数据
- 盲 SQL 注入，控制的查询结果不会在应用的响应中返回

### 检索隐藏数据(Retrieving hidden data)

想象一个购物应用，展示不同类别的产品。当用户点击“礼物”（Gifts）类别时，浏览器会请求以下链接：
`https://insecure-website.com/products?category=Gifts`
这会导致应用进行 SQL 查询，从数据库中检索相关产品的详细信息：
`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`
该 SQL 查询要求数据库返回：
- 所有细节
- 来自桌子（products）
- 其中是（category Gifts）
- 且是（released 1）

这一限制被用来隐藏未发布的产品。假设对于未发布的产品（released = 1；released = 0）
该应用没有实现任何针对 SQL 注入攻击的防御措施。这意味着攻击者可以构造以下攻击，例如：
`https://insecure-website.com/products?category=Gifts'--`
这会得到以下 SQL 查询：
`SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1`
关键是，注意 SQL 中这是一个注释指示器。这意味着查询的其余部分被解读为注释，实际上是删除了它。在这个例子中，这意味着查询不再包含。因此，所有产品都会被展示，包括尚未发布的产品（-- AND released = 1）
可以使用类似的攻击，让应用程序显示任何类别的所有产品，包括他们不知道的类别：
`https://insecure-website.com/products?category=Gifts'+OR+1=1--`
这会得到以下 SQL 查询：
`SELECT * FROM products WHERE category = 'Gifts' OR 1=1--‘ AND released = 1`
修改后的查询返回所有元素

### 颠覆应用逻辑(Subverting application logic)
一个应用程序，允许用户用用户名和密码登录。如果用户提交用户名和密码，应用程序通过执行以下 SQL 查询来检查凭证：`wiener`;`bluecheese`
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
如果查询返回了用户的详细信息，则登录成功；否则，就是被拒绝了
在这种情况下，攻击者可以以任何用户身份登录，无需密码。他们可以通过 SQL 注释序列来移除查询从句中的密码检查。例如，提交用户名和空密码会得到以下查询：`administrator'--`
SELECT * FROM users WHERE username = 'administrator' AND password = ''
该查询返回了该用户，并成功将攻击者登录为该用户

### 从其他数据库表检索数据(Retrieving data from other tables)

当应用程序以 SQL 查询结果响应时，攻击者可以利用 SQL 注入漏洞从数据库内的其他表中检索数据。可以使用该关键词执行额外查询，并将结果附加到原始查询上：UNION SELECT
例如，如果应用程序执行包含用户输入的以下查询：Gifts
SELECT name,description FROM products WHERE category = 'Gifts'
攻击者可以提交以下输入：
`' UNION SELECT username,password FROM users--`
这会导致应用程序返回所有用户名和密码，以及产品的名称和描述

### 盲目SQL注入漏洞(Blind SQL injection vulnerabilities)
许多 SQL 注入实例是盲目漏洞。这意味着应用程序不会返回 SQL 查询的结果或任何数据库错误的详细信息。盲目漏洞仍可被利用以访问未授权数据，但涉及的技术通常更复杂且难以实施
根据漏洞的性质和涉及的数据库，以下技术可用于利用盲目 SQL 注入：
- 根据单一条件的真实性，改变查询逻辑，触发应用程序响应的可检测差异。这可能包括向某个布尔逻辑注入新条件，或有条件地触发错误，比如除以零
- 有条件地触发查询处理的时间延迟。这使你能够根据申请回复所需的时间判断病情的真实性
- 利用 OAST 技术触发带外网络交互。这种技巧极其强大，能在其他技巧无效的情况下发挥作用。通常，可以通过带外通道直接窃取数据。例如，可以将数据放入受控制的域名的 DNS 查询中