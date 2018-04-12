### HHVM详解
- 出生
> `facebook`研发的一个项目
- 作用
> 将`PHP`代码转换成一种字节码中间格式，并且会缓存转换得到的字节码，然后通过使用`JIT`编译器转换并优化缓存的字节码，将其变成x86_64机器码。
- 适用范围
> 已经做了基本的优化措施后仍然需要提升应用性能的开发者。
- 配置
> 和`Zend Engine`一样，也使用`php.ini`配置文件。其默认路径是`/etc/hhvm/php.ini`。如果把`HHVM`当作`FastCGI`服务器使用，就要把相关的初始化指令添加到`/etc/hhvm/server.ini`。
- 执行
> 执行`hhvm`二进制文件时，要使用`-c`选项指定配置文件的路径。如果使用`hhvm`执行命令行脚本，只需要`/etc/hhvm/php.ini`配置文件,如下

> `hhvm -c /etc/hhvm/php.ini my-script.php`

> 如果使用`hhvm`二进制文件启动`FastCGI`服务器，`/etc/hhvm/php.ini`和`/etc/hhvm/server.ini`两个文件都需要，如下

> `hhvm -m server -c /etc/hhvm/php.ini -c /etc/hhvm/server.ini`
