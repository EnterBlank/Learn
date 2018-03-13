### 前言
> 一般来说，通过`http`或者`https`的请求，在接收参数方面我们一般采用了`POST`或者`GET`的方式，那如果我们需要在`shell`命令下把PHP当做脚本来执行，我们又该如何获取到需要的参数呢

### 方法
- 使用`$argv or $argc`参数接收
```
<?php
/**
 * 使用$argv or $argc参数接收
 */
 
echo "接收到{$argc}个参数";
print_r($argv);
```
> 执行

```
[root@abc cx]# /usr/local/php/bin/php test.php
接收到1个参数Array
(
    [0] => test.php
)
[root@abc cx]# /usr/local/php/bin/php test.php a b c d
接收到5个参数Array
(
    [0] => test.php
    [1] => a
    [2] => b
    [3] => c
    [4] => d
)
```
- 使用getopt函数
```
<?php
/**
 * 使用 getopt函数
 */
 
$param_arr = getopt('a:b:');
print_r($param_arr);
```
> 执行

```
[root@abc cx]# /usr/local/php/bin/php test.php -a 345
Array
(
    [a] => 345
)
[root@abc cx]# /usr/local/php/bin/php test.php -a 345 -b 12q3
Array
(
    [a] => 345
    [b] => 12q3
)
[root@abc cx]# /usr/local/php/bin/php test.php -a 345 -b 12q3 -e 3322ff
Array
(
    [a] => 345
    [b] => 12q3
)
```
- 提示用户输入
```
<?php
/**
 * 提示用户输入，类似Python
 */
fwrite(STDOUT,'请输入您的博客名：');
echo '您输入的信息是：'.fgets(STDIN);
```
> 执行

```
[root@abc cx]# /usr/local/php/bin/php test.php
请输入您的博客名：www.baidu.com
您输入的信息是：www.baidu.com
```
