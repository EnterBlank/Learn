- 前言
> `PHP`的底层是通过`C`语言编写的。`PHP`代码先由`Zend`编译器编译成`opcode`，再由`zend`虚拟机执行`opcode`。

- `C`语言怎么实现PHP的变量类型？
> 在zend.h文件里面C定义了一个结构体，每一个PHP变量对应一个结构体，如下：
```
zval_struct{
    zvalue_value value;       //储存PHP变量的值
    zend_uint  refcount__gc;  //指向次数
    zend_uchar type;          //变量的类型
    zend_uchar is_ref__gc;    //是否引用
}
```
> 其中zvalue_value的结构如下：
```
typedef union _zvalue_value{  //(联合)
    long lval;        //长整型
    double dval;      //浮点型
    struct {
        char *val;
        int len;
    } str;
    HashTable *ht;
    Zend_object_value obj;
}zvalue_value;
```
> 其中type的类型有以下几种：`IS_NULL`,`IS_BOOL`,`IS_LONG`,`IS_DOUBLE`,`IS_STRING`,`IS_ARRAY`,`IS_PBJECT`,`IS_RESOURCE`。PHP如果为`NULL`型,则结构体的`type`为`IS_NULL`，不设置`value`的值。如果为`BOOL`型，则结构体的`type`为`IS_BOOL`,再将`value`的值设置为`1/0`。如果为`RESOURCE`型，则结构体的`type`为`IS_RESOURCE`,再将`value`的值设置为服务器打开的接口编号（一般来说，资源型往往是服务器上打开一个接口）。

>假设$a = 'hello',那么他的结构体应该如下：
```
{
    {
        char : 'hello'
        len : 5
    }
    type : IS_STRING
    refcount__gc : 1
    is_ref__gc : 0
    
}
```
>由上可见，`PHP`中的字符串长度是直接体现在结构体中的，所以`strlen`的速度是非常快的，其时间复杂度为`O（1）`

> 再紧接着 $b = $a,那结构体又会发生什么变化？
```
{
    {
        char : 'hello'
        len : 5
    }
    type : IS_STRING
    refcount__gc : 2
    is_ref__gc : 0
    
}
```

> 接着 $b = 5 ,那结构体又会发生什么变化？
```
{
    {
        char : 'hello'
        len : 5
    }
    type : IS_STRING
    refcount__gc : 1
    is_ref__gc : 0
    
    
    zvalue : 5
    type : IS_STRING
    refcount__gc : 1
    is_ref__gc : 0
    
}
```
>由上可见，当`$b = $a`时，两个变量共享一个结构体，当`$b`的值改变时，会分裂出一个新的结构体，不会去改变`$a`的结构体，这个特性叫写时复制（`cow写时复制`）
- 结构体只储存了变量的值，那变量名呢？怎么和变量的值对应上？

> 这时候就要提到符号表（`symbol_table`）了。符号表其实是一个哈希表，存储了变量名和变量的`zval`结构体地址的映射。

- 如上所述，既然是写时复制，那为什么当`$b = &$a;$b='apple'`时，`$a`,`$b`的值全部都变了？
> 当引用时，$a$b是公用一个结构体，所以不会触发写时复制的特性。结构体变更如下：
```
{
    $b = &$a;
    
    {
        char : 'hello'
        len : 5
    }
    type : IS_STRING
    refcount__gc : 2
    is_ref__gc : 1
    //当发生引用时，is_ref__gc会变成1
    
    $b = 'apple';
    zvalue : 5
    type : IS_STRING
    refcount__gc : 2
    is_ref__gc : 1
    //$b的值改变时，会直接改变与$a共用的结构体的值，所以$a的值也随之变化
}
```
- 如果`$a = 3;$b = $a;$c = &$a;$c='apple'`,`$b`的值会被改变吗？
> 当引用时，如果结构体有被多个变量共享，那么就会强制复制出一个结构体，来保证`$b`的值不变。