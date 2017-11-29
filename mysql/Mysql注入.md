### 概念
> 利用数据库的一些漏洞来攻击数据库
### 常用实例
- 例一：
> 当sql语句为`select * from users where username = '$username' and password = '$password'`时可以试用万能密码或者万能用户名,如下万能密码：` bb' or 1 = 1`,万能用户名：` 'xx' union select * from users/*`。其中`/*`表示为不执行之后的语句。
- 例二：
> 当sql语句为`select * from users where username = $username and password = $password`时，其中变量所替换的字符串会被当作数字对待，所以应试用其他方式。万能密码：` 33 union select * from users`,或者万能用户名：` 33 union select * from user/* `。
- 例三：
> 当sql语句为`select * from users where username like '%$username%'`时，通过传入参数`__`或者`%`也会造成注入。
### 如何防止MySQL的注入
- php.ini配置
> 将`php.ini`配置文件的`magic_quotes_gpc=on`打开，服务器会将所有`‘`进行转义，从而避免注入。
- addslashes
> 通过PHP代码层面`addslashes`函数达到转义的效果，并且通过`str_replace`来替换`_%`等特殊字符来避免注入。
- PDO
``` php
<?php
$pdo = new PDO("mysql:host=192.168.0.1;dbname=test;charset=utf8","root");
$st = $pdo->prepare("select * from info where id =? and name = ?");
 
$id = 21;
$name = 'zhangsan';
$st->bindParam(1,$id);
$st->bindParam(2,$name);
 
$st->execute();
$st->fetchAll();
?>
```
>通过软件`wireshark`监听`PHP`与`MySQL`的通信发现，`PHP`只是简单的把`SQL`语句发给`Mysql`。

![image](http://dl.iteye.com/upload/attachment/0082/1268/78ca895f-3dcf-39f3-a452-cb347a4807d1.jpg)

> 其实，这与我们平时使用mysql_real_escape_string将字符串进行转义，再拼接成SQL语句没有差别（只是由PDO本地驱动完成转义的），显然这种情况下还是有可能造成SQL注入的，也就是说在php本地调用pdo prepare中的mysql_real_escape_string来操作query，使用的是本地单字节字符集，而我们传递多字节编码的变量时，有可能还是会造成SQL注入漏洞(php 5.3.6以前版本的问题之一，这也就解释了为何在使用PDO时，建议升级到php 5.3.6+，并在DSN字符串中指定charset的原因。

``` php
//针对php 5.3.6以前版本，以下代码仍然可能造成SQL注入问题：
$pdo->query('SET NAMES GBK'); 
$var = chr(0xbf) . chr(0x27) . " OR 1=1 /*"; 
$query = "SELECT * FROM info WHERE name = ?"; 
$stmt = $pdo->prepare($query); 
$stmt->execute(array($var)); 
```

>正确的转义应该是给mysql Server指定字符集，并将变量发送给MySQL Server完成根据字符转义。

>PDO有一项参数，名为PDO::ATTR_EMULATE_PREPARES ，表示是否使用PHP本地模拟prepare，此项参数默认值未知。而且根据我们刚刚抓包分析结果来看，php 5.3.6+默认还是使用本地变量转，拼接成SQL发送给MySQL Server的，我们将这项值设置为false, 试试效果，如以下代码：

```php
<?php
$pdo = new PDO("mysql:host=192.168.0.1;dbname=test;","root");
$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
 
$st = $pdo->prepare("select * from info where id =? and name = ?");
$id = 21;
$name = 'zhangsan';
 
$st->bindParam(1,$id);
$st->bindParam(2,$name);
$st->execute();
$st->fetchAll();
?>
```
>红色行是我们刚加入的内容，运行以下程序，使用wireshark抓包分析，得出的结果如下：
![image](http://dl.iteye.com/upload/attachment/0082/1270/4b73404b-3cc7-37c7-8679-5200e7dc872e.jpg)
![image](http://dl.iteye.com/upload/attachment/0082/1272/1853a6e4-6e77-3297-921f-19132a5df698.jpg)

>这次`PHP`是将`SQL`模板和变量是分两次发送给`MySQL`的，由`MySQL`完成变量的转义处理，既然变量和`SQL`模板是分两次发送的，那么就不存在SQL注入的问题了，但需要在`DSN`中指定`charset`属性，如：
`$pdo = new PDO('mysql:host=localhost;dbname=test;charset=utf8', 'root')`;

## 注意
> 如果在DSN中指定了charset, 是否还需要执行set names <charset>呢？
是的，不能省。set names <charset>其实有两个作用：
A.  告诉mysql server, 客户端（PHP程序）提交给它的编码是什么
B.  告诉mysql server, 客户端需要的结果的编码是什么
也就是说，如果数据表使用gbk字符集，而PHP程序使用UTF-8编码，我们在执行查询前运行set names utf8, 告诉mysql server正确编码即可，无须在程序中编码转换。这样我们以utf-8编码提交查询到mysql server, 得到的结果也会是utf-8编码。省却了程序中的转换编码问题，不要有疑问，这样做不会产生乱码。

>那么在DSN中指定charset的作用是什么? 只是告诉PDO, 本地驱动转义时使用指定的字符集（并不是设定mysql server通信字符集），设置mysql server通信字符集，还得使用set names <charset>指令。