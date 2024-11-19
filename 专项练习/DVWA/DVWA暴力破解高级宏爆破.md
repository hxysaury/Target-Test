# DVWA暴力破解高级宏爆破

先将安全等级调至高级，点击`submit`提交

![image-20230812133032883](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812133032883.png)



浏览器开启`bp`代理

![image-20230812133146227](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812133146227.png)

kali开启`bp` 工具，开启`Proxy`

![image-20230812133244225](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812133244225.png)

点击`Brute Force`这个选项卡

![image-20230812133345799](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812133345799.png)

bp拦截到请求的数据包

![image-20230812133401248](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812133401248.png)

### 宏设置

![image-20230812134604864](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812134604864.png)

如果是有的`bp`版本比较旧，在旧版本的上面菜单栏有一个`Project options`点击去选择`Session`，就会出现跟下面一样的宏界面

![image-20230812134624697](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812134624697.png)

![image-20230812134734809](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812134734809.png)

![image-20230812134843572](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812134843572.png)

![image-20230812134857752](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812134857752.png)

![image-20230812135038820](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812135038820.png)

![image-20230812135217542](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812135217542.png)

![image-20230812135252339](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812135252339.png)

![image-20230812135346060](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812135346060.png)

![image-20230812135408900](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812135408900.png)

![image-20230812135643912](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812135643912.png)

![image-20230812135659204](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812135659204.png)

![image-20230812135737624](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812135737624.png)

![image-20230812135818684](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812135818684.png)

![image-20230812135826336](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812135826336.png)

宏设置 完成

测试宏 是否设置正确

![image-20230812140037783](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812140037783.png)

点击登录，`bp`会 拦截到请求的数据包，打开`bp`的`Proxy`，点击`Forward`放包

![image-20230812140146372](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812140146372.png)

发送到`Repeater`，点击`send`，观察`user_token`有没有跟着变化

![image-20230812140342403](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812140342403.png)

### 开始爆破

发送到`Intruder`

![image-20230812140857957](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812140857957.png)

模式选择`Cluster bomb`集束炸弹

![image-20230812140930086](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812140930086.png)

将输入的用户名和密码打上标记

![image-20230812140950661](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812140950661.png)

给第一个标记导入用户字典

![image-20230812141028162](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812141028162.png)

给第二个标记导入密码字典

![image-20230812141042769](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812141042769.png)

选择一个线程

![image-20230812141106428](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812141106428.png)

选择总是重定向

![image-20230812141142690](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812141142690.png)

**开始爆破**

![image-20230812141252843](https://gitee.com/zh_sng/cartographic-bed/raw/master/img/image-20230812141252843.png)

### 总结

```
首先访问有user_token的页面，bp拦截到当前页面链接
使用宏配置，正则表达过滤user_token
输入账号密码拦载，将拦截的内容先放到重发器(Repeater)里面测试一下，user_token是否会变，如果变了，表面之前的宏设置成功
最后使用测试器(Intruder)，进行密码爆破，使用集束炸弹模式
```

