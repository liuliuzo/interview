#数据库字符集utf8和utf8mb4的详细区别
1.MySQL的“utf8mb4”是真正的“UTF-8”。2.MySQL的“utf8”是一种“专属的编码”，它能够编码的Unicode字符并不多。我要在这里澄清一下：所有在使用“utf8”的MySQL和MariaDB用户都应该改用“utf8mb4”，尽量不要再使用“utf8”。

那么什么是编码？什么是UTF-8？我们都知道，计算机使用0和1来存储文本。比如字符“C”被存成“01000011”，那么计算机在显示这个字符时需要经过两个步骤：

1.计算机读取“01000011”，得到数字67，因为67被编码成“01000011”。
2.计算机在Unicode字符集中查找67，找到了“C”。
同样的：
1.我的电脑将“C”映射成Unicode字符集中的67。
2.我的电脑将67编码成“01000011”，并发送给Web服务器。几乎所有的网络应用都使用了Unicode字符集，因为没有理由使用其他字符集。

Unicode字符集包含了上百万个字符。最简单的编码是UTF-32，每个字符使用32位。这样做最简单，因为一直以来，计算机将32位视为数字，而计算机最在行的就是处理数字。但问题是，这样太浪费空间了。

UTF-8可以节省空间，在UTF-8中，字符“C”只需要8位，一些不常用的字符，比如“”需要32位。其他的字符可能使用16位或24位。一篇类似本文这样的文章，如果使用UTF-8编码，占用的空间只有UTF-32的四分之一左右。

MySQL的“utf8”字符集与其他程序不兼容，它所谓的“”，可能真的是一坨……

总结如果你在使用MySQL或MariaDB，不要用“utf8”编码，改用“utf8mb4”。这里提供了一个指南用于将现有数据库的字符编码从“utf8”转成“utf8mb4”

##应用与拓展
MySQL简史为什么MySQL开发者会让“utf8”失效？我们或许可以从提交日志中寻找答案。

MySQL从4.1版本开始支持UTF-8，也就是2003年，而今天使用的UTF-8标准（RFC 3629）是随后才出现的。

旧版的UTF-8标准（RFC 2279）最多支持每个字符6个字节。2002年3月28日，MySQL开发者在第一个MySQL 4.1预览版中使用了RFC 2279。

同年9月，他们对MySQL源代码进行了一次调整：“UTF8现在最多只支持3个字节的序列”。

是谁提交了这些代码？他为什么要这样做？这个问题不得而知。在迁移到Git后（MySQL最开始使用的是BitKeeper），MySQL代码库中的很多提交者的名字都丢失了。2003年9月的邮件列表中也找不到可以解释这一变更的线索。

不过我可以试着猜测一下。

2002年，MySQL做出了一个决定：如果用户可以保证数据表的每一行都使用相同的字节数，那么MySQL就可以在性能方面来一个大提升。为此，用户需要将文本列定义为“CHAR”，每个“CHAR”列总是拥有相同数量的字符。如果插入的字符少于定义的数量，MySQL就会在后面填充空格，如果插入的字符超过了定义的数量，后面超出部分会被截断。

MySQL开发者在最开始尝试UTF-8时使用了每个字符6个字节，CHAR(1)使用6个字节，CHAR(2)使用12个字节，并以此类推。

应该说，他们最初的行为才是正确的，可惜这一版本一直没有发布。但是文档上却这么写了，而且广为流传，所有了解UTF-8的人都认同文档里写的东西。

不过很显然，MySQL开发者或厂商担心会有用户做这两件事：

1.使用CHAR定义列（在现在看来，CHAR已经是老古董了，但在那时，在MySQL中使用CHAR会更快，不过从2005年以后就不是这样子了）。2.将CHAR列的编码设置为“utf8”。我的猜测是MySQL开发者本来想帮助那些希望在空间和速度上双赢的用户，但他们搞砸了“utf8”编码。

所以结果就是没有赢家。那些希望在空间和速度上双赢的用户，当他们在使用“utf8”的CHAR列时，实际上使用的空间比预期的更大，速度也比预期的慢。而想要正确性的用户，当他们使用“utf8”编码时，却无法保存像“”这样的字符。

在这个不合法的字符集发布了之后，MySQL就无法修复它，因为这样需要要求所有用户重新构建他们的数据库。最终，MySQL在2010年重新发布了“utf8mb4”来支持真正的UTF-8。

为什么这件事情会让人如此抓狂因为这个问题，我整整抓狂了一个礼拜。我被“utf8”愚弄了，花了很多时间才找到这个bug。但我一定不是唯一的一个，网络上几乎所有的文章都把“utf8”当成是真正的UTF-8。

“utf8”只能算是个专有的字符集，它给我们带来了新问题，却一直没有得到解决。

***在使用MySQL工具的时候我们经常会遇到乱码问题，其实这是由于编码导致的。下面就介绍下Mysql的编码问题***

#查看数据库编码
```
show variables like 'char%';
```
我们可以看到
|  Variable_name  | Value  |
|  ----  | ----  |
|character_set_client|utf8mb4|
|character_set_connection|utf8mb4|
|character_set_results|utf8mb4|
|character_set_server|utf8mb4|
|character_set_database|utf8|
|character_set_system|utf8|
|character_sets_dir|C:\Program Files\MySQL\MySQL Server 8.0\share\charsets\|
|character_set_filesystem|binary|

-  character_set_client
主要用来设置客户端使用的字符集。**通俗的讲就是mysql把客户端传递过来的数据都当成是utf8mb4**
-  character_set_connection
主要用来设置连接数据库时的字符集，如果程序中没有指明连接数据库使用的字符集类型则按照这个字符集设置。
-  character_set_database
主要用来设置默认创建数据库的编码格式，如果在创建数据库时没有设置编码格式，就按照这个格式设置。
-  character_set_filesystem
文件系统的编码格式，把操作系统上的文件名转化成此字符集，即把 character_set_client转换character_set_filesystem， 默认binary是不做任何转换的。
-  character_set_results
数据库给客户端返回时使用的编码格式，如果没有指明，使用服务器默认的编码格式。**通俗的讲就是mysql发送个客户端的数据是utf8mb4的**
-  character_set_server
服务器安装时指定的默认编码格式，这个变量建议由系统自己管理，不要人为定义。
-  character_set_system
数据库系统使用的编码格式，这个值一直是utf8，不需要设置，它是为存储系统元数据的编码格式。
-  character_sets_dir
这个变量是字符集安装的目录。

#Mysql的字符集内部处理
1.mysql Server收到请求时将请求数据从 character_set_client 转换为 character_set_connection
2.进行内部操作前将请求数据从 character_set_connection 转换为内部操作字符集,步骤如下
　　A. 使用每个数据字段的 CHARACTER SET 设定值；
　　B. 若上述值不存在，则使用对应数据表的字符集设定值
　　C. 若上述值不存在，则使用对应数据库的字符集设定值；
　　D. 若上述值不存在，则使用 character_set_server 设定值。
3.最后将操作结果从内部操作字符集转换为 character_set_results
![image.png](https://upload-images.jianshu.io/upload_images/14079828-9e79bb8527143ae1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

***到这里大家应该知道怎么处理mysql UI工具的各种乱码问题了吧！***

文章参考：
[关于MySQL中的8个 character_set 变量说明](https://blog.csdn.net/sun8112133/article/details/79921734)
[数据库字符集utf8和utf8mb4的详细区别](https://blog.csdn.net/weixin_44269886/article/details/89364733)
