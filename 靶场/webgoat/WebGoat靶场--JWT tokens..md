## WebGoat靶场--JWT tokens

### 环境启动

环境地址：https://github.com/WebGoat/WebGoat

启动`WebGoat`靶场

```java
java -jar webgoat-server-8.0.0.M17.jar --server.port=8888 --
server.address=192.168.8.8
```

![image-20230913193559388](./imgs/image-20230913193559388.png)

访问`WebGoat`靶场

`127.0.0.1:8888/WebGoat`

![image-20230913193656453](./imgs/image-20230913193656453.png)

注册一个用户

![image-20230913193820738](./imgs/image-20230913193820738.png)

### 第四关

通过目标：以管理员的身份清楚普通用户的投票数

![image-20230913200732662](./imgs/image-20230913200732662.png)

![image-20230913200854847](./imgs/image-20230913200854847.png)

点击删除投票的按钮，	`BurpSuite`拦截数据包，并观察数据包

![image-20230913201113479](./imgs/image-20230913201113479.png)

![image-20230913201257233](./imgs/image-20230913201257233.png)

访问[jwt.io](https://jwt.io/)网站，粘贴进去

![image-20230913201402858](./imgs/image-20230913201402858.png)

可以看出使用的加密算法为`HS512`

Payload：`admin`为`false`，`user`不是`admin`，而是`Tom`

选中`Payload`

![image-20230913201641103](./imgs/image-20230913201641103.png)

来到[base64](https://www.bejson.com/enc/base64/)这个网站,`admin`修改为`true`，

![image-20230913202005582](./imgs/image-20230913202005582.png)

![image-20230913202053889](./imgs/image-20230913202053889.png)

![image-20230913202116650](./imgs/image-20230913202116650.png)

再把JWT头部的算法类型改为`none`，如果改为了`none`，后面的签名就没有意义了

签名的意义就是为了加密算法是否正确，内容是否正确，现在不用加密了，所以最后一个`.`后面的签名部分就不要了

![image-20230913202440286](./imgs/image-20230913202440286.png)

![image-20230913202500971](./imgs/image-20230913202500971.png)

![image-20230913202813204](./imgs/image-20230913202813204.png)

![image-20230913202929882](./imgs/image-20230913202929882.png)

刷新页面

![image-20230913203028672](./imgs/image-20230913203028672.png)

成功重置！！

### 第五关

修改 exp 有效时间 ，修改username为 WebGoat

![image-20230913211332034](./imgs/image-20230913211332034.png)

![image-20230913211400720](./imgs/image-20230913211400720.png)

![image-20230913211632480](./imgs/image-20230913211632480.png)

爆破秘钥

```python
 hashcat -m 16500 jwt.txt -a 3 -w 3 1.txt
 
-m 16500 这里的 16500 对应的就是 jwt 的 token 爆破；
-a 3 代表蛮力破解 
-w 3 可以理解为高速破解，就是会让桌面进程无响应的那种高速
jwt.txt 是我把题目要求破解的 token 保存到的文件 
pass.txt 密码字典
```



![image-20230913211204638](./imgs/image-20230913211204638.png)

得出密钥是`victory`



![image-20230913211718117](./imgs/image-20230913211718117.png)

### 第七关

通过目标：冒充tom用户，让tom帮我们付钱



`BurpSuite`拦截数据包，并观察数据包

![image-20230913203801862](./imgs/image-20230913203801862.png)

![image-20230913203935783](./imgs/image-20230913203935783.png)

并不是说一定要把`JWT`放到`Cookie`字段里

靶场提供了一个JWT ，点击`here`跳转链接

![image-20230913204049743](./imgs/image-20230913204049743.png)

![image-20230913204124501](./imgs/image-20230913204124501.png)

![image-20230913204249762](./imgs/image-20230913204249762.png)



![image-20230913204628915](./imgs/image-20230913204628915.png)

![image-20230913204343107](./imgs/image-20230913204343107.png)

修改Payload的过期时间和头部的算法类型

[unix时间互换](https://www.bejson.com/convert/unix/)

![image-20230913204736618](./imgs/image-20230913204736618.png)

![image-20230913205049946](./imgs/image-20230913205049946.png)

![image-20230913205125999](./imgs/image-20230913205125999.png)





![image-20230913205313986](./imgs/image-20230913205313986.png)

![image-20230913205345949](./imgs/image-20230913205345949.png)