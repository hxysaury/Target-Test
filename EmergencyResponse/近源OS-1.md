
```
链接：https://pan.baidu.com/s/1-qemfxTKITQZs5NI1EZ2_w 
提取码：hm84 
--来自百度网盘超级会员V3的分享
```

### 挑战内容
前景需要：小王从某安全大厂被优化掉后，来到了某私立小学当起了计算机老师。某一天上课的时候，发现鼠标在自己动弹，又发现除了某台电脑，其他电脑连不上网络。感觉肯定有学生捣乱，于是开启了应急。

1.攻击者的外网IP地址

2.攻击者的内网跳板IP地址

3.攻击者使用的限速软件的md5大写

4.攻击者的后门md5大写

5.攻击者留下的flag

解题：

运行桌面上"解题工具.exe"即可

注意：该靶机存在许多非预期解，请合理练习应急响应技能(如果你是为了解题而解题，当然很好解)。



相关账户密码

Administrator

zgsf@2024

这个靶机，做的有点仓促，题解程序写错了，有一个让填md5的，其实是填大写md5

![](imgs/3628d24a4efa4273c85c9d957f7bb5e9_MD5.jpeg)

### 开始挑战

先登录环境，然后rdp连接

把桌面上的word文档放进另一台虚拟机，进行沙箱分析，刚一放进去，windows自带的杀毒软件就提示了

![](imgs/07e167aeb43a6f3bba339e4bebc269cc_MD5.jpeg)

![](imgs/234a2c9a0326c99b96b9ae10a9477742_MD5.jpeg)


![](imgs/7bbab670ddfe7e06e92f167d4b9711cd_MD5.jpeg)

发现攻击者外网IP：8.219.200.130



test.bat被隐藏掉了

![](imgs/a3b27b7efbbf9a8485e8332f12676d3a_MD5.jpeg)

通过文件夹选项显示被隐藏的文件

![](imgs/e6272fbc639c46fe03cfdaec708033e8_MD5.jpeg)


![](imgs/168a9a9935e6acc9649482f9249dc389_MD5.jpeg)

发现内网跳板的地址

然后是限速软件，靶场取自真实环境，真实环境中，一个普通用户怎么去劫持整个局域网网速呢？？

答案：ARP劫持

用啥劫持的呢？

翻一翻C盘文件就知道了  

地址

```
  
C:\PerfLogs\666\666\777\666\666\666\666\666\666\666\666\666\666\666
```

![](imgs/2a7f0a56000340a2c1f82031b43da865_MD5.jpeg)

![](imgs/a989e86776d6729aa8d1dfc2cf20a57d_MD5.jpeg)


使用工具计算Hash值

![](imgs/08c8f4a235b2d2e434a44d79a17e67c4_MD5.jpeg)

`2A5D8838BDB4D404EC632318C94ADC96`


然后就是另一个最后一个后门，

发现shift后门

连按5次shift系统会运行粘滞键

位置在

```
C:\Windows\System32\sethc.exe
```

![](imgs/65805c2a541ea8293f7b68ff8e0f4997_MD5.jpeg)

flag:flag{zgsf@shift666}


![](imgs/9989036a2e178fe9c9b6b79bfc73a2d4_MD5.jpeg)