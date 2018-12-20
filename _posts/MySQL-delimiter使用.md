# mysql:

## delimiter用法

一般在执行MySQL语句时，遇见分号MySQL就会去执行分号之前的语句，认为;分号是一条执行语句的终结，而在写大量sql时（存储过程），可能会出现用到分号的情况，但是又不想MySQL去执行这段sql，如此以来就需要使用到 delimiter关键字，来自定义语句结束符。

## 变量定义
**declare关键字：**用户定义局部变量的，也就是在存储过程中，begin-end之间使用的变量，可以类比与Java方法中的变量。

定义：`declare insert_count int default 0`

使用：

  - 函数赋值方式:`select row_count() into insert_count`
  - 直接赋值：`set insert_count = 1`

**@变量名：**这样定义为会话变量。就是在一个sql session中都可以使用的变量。可以类比与Java中类的全局变量。

定义：

- `set @car_num = 4` 。 
- `select @car_num:=5` 在select中只能使用`:=`来赋值，如果使用`=`会出现null赋值不上的情况。