# Deathnote

> 靶机文档：[Deathnote: 1](https://www.vulnhub.com/entry/deathnote-1,739/)
>
> 下载地址：[**Download (Mirror)**](https://download.vulnhub.com/deathnote/Deathnote.ova)
>
> 难易程度：**Easy**

### 信息收集

#### 主机发现

![image-20230916155129366](./imgs/image-20230916155129366.png)

#### 端口扫描

![image-20230916155210255](./imgs/image-20230916155210255.png)

访问靶机的`80`端口，报错，如下图显示：

地址栏输入地址后，自动跳转到一个域名

![image-20230916155618498](./imgs/image-20230916155618498.png)

说明我们需要做`域名映射`

![image-20230916155811166](./imgs/image-20230916155811166.png)

映射做好后，就可以看到页面正常显示 了，发现路径是`/wordpress/`，那肯定 就是`wordpress`CMS 了

![image-20230916155843612](./imgs/image-20230916155843612.png)



#### 目录扫描

##### dirsearch

```bash
 dirsearch -u 192.168.8.6 -i 200
```

![image-20230916155937956](./imgs/image-20230916155937956.png)

##### gobuster

```bash
gobuster dir -u http://192.168.8.6 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
```

![image-20230916160203913](./imgs/image-20230916160203913.png)

##### dirb扫描

```bash
dirb http://192.168.8.6/wordpress/
```



![image-20230916162446318](./imgs/image-20230916162446318.png)



`robots.txt`

![image-20230916160601110](./imgs/image-20230916160601110.png)

查看这个图片地址的源代码：

![image-20230916162823301](./imgs/image-20230916162823301.png)

`/wordpress/wp-login.php`

![image-20230916160910211](./imgs/image-20230916160910211.png)

`wordpress/wp-content/uploads/`

![image-20230916162622489](./imgs/image-20230916162622489.png)

`wordpress/wp-content/uploads/2021/07/user.txt`

![image-20230916163006633](./imgs/image-20230916163006633.png)

`wordpress/wp-content/uploads/2021/07/notes.txt`

![image-20230916163204339](./imgs/image-20230916163204339.png)

### 漏洞利用

将`user.txt`和`notes.txt`里的内容分别复制到到两个文件里

#### wpscan扫描

枚举用户

```bash
wpscan --url http://192.168.8.6/wordpress -e u
```

![image-20230916161219028](./imgs/image-20230916161219028.png)

使用 `cewl http://192.168.8.6/ -w password.txt` 制作密码字典。

```bash
 cewl http://192.168.8.6/ -w password.txt
```

得到的密码啥也不是

#### Hydra爆破

![image-20230916163917422](./imgs/image-20230916163917422.png)

用户:`l`

密码：`death4me`

---

ssh连接

![image-20230916164045025](./imgs/image-20230916164045025.png)

查看`user.txt`

![image-20230916164115199](./imgs/image-20230916164115199.png)

发现编码为`brins`，也可以叫做`ook`，解码

解码地址：https://ctf.bugku.com/tool/brainfuck

![image-20230916164218429](./imgs/image-20230916164218429.png)

在`/opt/`下发现一个`L `的目录

发现一串十六进制字符，然后转码

![image-20230916164515393](./imgs/image-20230916164515393.png)

[CyberChef](https://gchq.github.io/CyberChef/)

![image-20230916164824191](./imgs/image-20230916164824191.png)

得到一个密码：`kiraisevil`

在前面使用`wpscan`扫描的时候扫出来一个`kira`用户，而且在`/home`目录下，也有这么个用户

所以可以尝试切换用户

切换成功第一步先看有没有能以`root身份运行的命令字`

![image-20230916165124887](./imgs/image-20230916165124887.png)

发现`kira`用户有任意的权限

![image-20230916165240520](./imgs/image-20230916165240520.png)

---

[DC-4](https://blog.csdn.net/ZhaoSong_/article/details/132110397)最后也是靠`sudo su`提的权



### 总结

-  多使用几个目录扫描工具，会有不一样的结果，
- 从扫出来的目录中仔细找
- wpscan就是专门扫这个CMS的工具，
- 九头蛇爆破