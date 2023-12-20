> [InfoSecWarrior CTF 2020: 01官网地址](https://www.vulnhub.com/entry/infosecwarrior-ctf-2020-01,446/)

### 信息收集

**主机发现**

```bash
arp-scan -l
```

![image-20230820113228000](./imgs/image-20230820113228000.png)

**端口扫描**

```bash
nmap -A -p- 192.168.80.145
```

![image-20230820113609008](./imgs/image-20230820113609008.png)

**目录爆破**

```bash
dirsearch -u 192.168.80.145 -i 200
```

![image-20230820114637781](./imgs/image-20230820114637781.png)

访问80端口，只能看出是Apache站点，没有别的

![image-20230820114508785](./imgs/image-20230820114508785.png)

访问一下`sitemap.xml`

![image-20230820114743945](./imgs/image-20230820114743945.png)

访问`192.168.80.145/index.htnl`，也没啥有用的，只有一张gif格式的动态图

![image-20230820114949758](./imgs/image-20230820114949758.png)

查看页面源代码，发现隐藏的一个`form`表单,可以删除`hidden='True'`，回车！

![image-20230820115136018](./imgs/image-20230820115136018.png)

页面上就多了一个用于提交的输入框

![image-20230820115324329](./imgs/image-20230820115324329.png)

在输入框中输入`ls`，没有得到正常的回显

![image-20230820115548243](./imgs/image-20230820115548243.png)

把`from`表单换成`post`请求

![image-20230820115659241](./imgs/image-20230820115659241.png)

再次在 输入框中输入`ls`，成功有了 回显

![image-20230820115749390](./imgs/image-20230820115749390.png)

查看`cmd.php`，在表单中输入`cat cmd.php`

![image-20230820115943840](./imgs/image-20230820115943840.png)

使用`ssh`远程连接  `ssh isw0@192.168.80.145`    密码：`123456789blabla`

### 提权



连接成功后，`sudo -l`查看有哪些能以`root`身份执行的命令

![image-20230820120918317](./imgs/image-20230820120918317.png)

去这个网站上找提权命令：`https://gtfobins.github.io/`

只能找到`rpm`命令字的提权方式

![image-20230820121132794](./imgs/image-20230820121132794.png)

![image-20230820121301465](./imgs/image-20230820121301465.png)