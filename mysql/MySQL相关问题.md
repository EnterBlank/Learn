### MySQL相关问题
- 在执行批量插入语句时，`MySQL`报了一个`Packet for query is too large`的错误

> 这个问题是由于`MySQL`的写入数据量过大造成的。可以通过在查询`SHOW VARIABLES LIKE '%max_allowed_packet%'`，得到如下结果：
![image](https://img-blog.csdn.net/20170320100821194?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGxidDAxMTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
由于`max_allowed_packet`字段设置的值过小导致了该问题。可以通过`my.ini`配置文件，将`max_allowed_packet=20M`手动改大，来避免错误的产生。

- 在执行批量预插入语句时，`MySQL`报了一个`Prepared statement contains too many placeholders`的错误。

> 这个错误的大概意思就是预执行语句包含的占位符过多。Mysql支持的占位符最多为65535(2^16-1)个，超过这个数量就会报错。所以解决的办法就是分批次插入。