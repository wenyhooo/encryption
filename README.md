



## iOS常用的编码和加密

* [iOS常用的编码和加密](#ios常用的编码和加密)
   * [base64](#base64)
   * [UTF-8](#utf-8)
   * [常见加密算法](#常见加密算法)
      * [非对称加密](#非对称加密)
      * [对称加密](#对称加密)
         * [1、密钥](#1密钥)
         * [2、填充](#2填充)
         * [3、AES的五种模式](#3aes的五种模式)
      * [Hash](#hash)
         * [SHA-1 与 MD5区别](#sha-1-与-md5区别)
         * [<a href="https://zh.wikipedia.org/wiki/MD5" rel="nofollow">MD5</a>举例应用](#md5举例应用)
         * [MD5没有解密，那它就绝对安全吗？](#md5没有解密那它就绝对安全吗)
         * [加盐（salt）方案为此诞生](#加盐salt方案为此诞生)
         * [加盐（salt）安全吗？](#加盐salt安全吗)
         * [”HMAC加密方案“为此诞生](#hmac加密方案为此诞生)
         * [HMAC了还有破绽吗？](#hmac了还有破绽吗)
      * [破解MD5](#破解md5)
         * [暴力枚举法：](#暴力枚举法)
         * [字典法：](#字典法)
         * [<a href="https://zh.wikipedia.org/wiki/彩虹表" rel="nofollow">彩虹表法</a>](#彩虹表法)

我们平时所接触的大多数是base64编码、UTF-8和一些加密算法（md5等），下面就聊聊这几种东西到底都是什么？

### base64

base64是一种编码方案，不能用来加密。 base64后的结果由64种字符组成，对编码的结果可以反解码。

对文件base64编码的终端命令：

```shell
base64 文件路径
```

文件越大，输出的字符越长，由64种字符组成。

OC示例：

把数据流进行base64编码

```objective-c
#import <Foundation/NSData.h>
- (NSString *)base64EncodedStringWithOptions:(NSDataBase64EncodingOptions)options;//输出形式文本
- (NSData *)base64EncodedDataWithOptions:(NSDataBase64EncodingOptions)options//输出形式Data
```

把base64编码后的NSData/NSString还原

```
- (nullable instancetype)initWithBase64EncodedString:(NSString *)base64String options:(NSDataBase64DecodingOptions)options；
- (nullable instancetype)initWithBase64EncodedData:(NSData *)base64Data options:(NSDataBase64DecodingOptions)options；
```

使用场景：

一般加密过后的数据(NSData)会用base64来编码传输， 因为一般不直接用二进制来传输。

### UTF-8

UTF-8到底是什么？为什么我们有时候需要用utf-8编码，聊一聊UTF-8的由来就清楚了：

------

那就得从**ASCII**说起，它的全称：**A**merican **S**tandard **C**ode for **I**nformation **I**nterchange，**美国信息交换标准代码**，也主要用于显示现代英语。共收录128个字符，0~127。

------

后来要走向国际化，欧洲各国语言不通，字母也不一样，它就无法用 ASCII 码表示，于是乎**EASCII诞生**（**Extended ASCII**，延伸美国标准信息交换码）是将ASCII码由7位扩充为8位而成。EASCII的内码是由0到255共有256个字符组成。

------

后来要走向全球，不同的国家有不同的字母，因此，哪怕它们都使用256个符号的编码方式，代表的字母却不一样，至于亚洲国家的文字，使用的符号就更多了，汉字就多达10万左右。一个字节只能表示256种符号，要表示中文韩文日文就必须使用多个字节表达一个符号。所以**Unicode诞生**。

------

但是Unicode处理英文和非英文都是占2个字节，对英文很不公平，浪费资源，于是乎<u>**UTF-8诞生**</u>（Unicode TransformationFormat-8bit）是用以解决国际上字符的一种多字节编码，它对英文使用8位（即一个字节），中文使用24为（三个字节）来编码。UTF-8包含全世界所有国家需要用到的字符，是国际编码，通用性强。UTF-8编码的文字可以在各国支持UTF8字符集的浏览器上显示。如果是UTF8编码，则在外国人的英文IE上也能显示中文，他们无需下载IE的中文语言支持包。



### 常见加密算法

#### 非对称加密

常见的非对称算法主要有RSA、DSA等。

RSA 是由三个提出者 [Ron Rivest](http://en.wikipedia.org/wiki/Ron_Rivest)、[Adi Shamir](http://en.wikipedia.org/wiki/Adi_Shamir) 和 [Leonard Adleman](http://en.wikipedia.org/wiki/Len_Adleman) 的姓氏首字母组成的。

两种使用方式：

1. 常见公钥加密，私钥解密 
2. 但私钥也可以加密 ，公钥也可以解密

公钥可以有多个，可以多方持有，但私钥只有一个。

优缺点：

它的秘钥很长，加密的计算量比较大，安全性较高，但是加密速度比较慢。

一般来说用于少量数据的加密。可是哪里这么巧都是少量数据啊？

可以通过对其他加密算法加密出来的较短的加密结果进行二次RSA加密，配合使用

#### 对称加密

主要包含：AES、3DES、DES（高级密码标准）

对称算法是可逆的。

AES加密简称：（Advanced Encryption Standard），是DES算法的替代者，也是当今最流行的对称加密算法之一。

AES算法，**有三个重要概念：秘钥、填充、模式。**

##### 1、密钥

AES算法实现加密和解密的根本是密钥，对称加密算法之所以对称，是因为这类算法对明文的加密和解密需要使用**同一个密钥**。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghpgfhgnzfj30uq0lo42f.jpg" alt="image-20200813202546473" style="zoom: 33%;" />

这么多enum实际上是对不同长度密钥的使用。理论上来讲AES256的安全性最高，而128的速度最快，本质原因是它们的加密的处理轮数不同。

##### 2、填充

**加密方式：**AES算法在对内容加密的时候，并不是把整个明文内容加密成一整段密文，而是把明文拆分成若干独立的明文块，每一个铭文块长度128bit，这样分段加密，最终再拼接到一起，就是最终的AES加密结果。

如果不能被128整除，有余，这个时候就需要**填充**，有三种填充方式：

- NoPadding:

  不做任何填充，但是要求明文必须是16字节的整数倍。

- PKCS7Padding  iOS只支持这个

  <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghph2g4nx7j30yc0asq4i.jpg" alt="image-20200813204747553" style="zoom:30%;" />

  系统默认使用CBC模式，可以明文指定使用ECBMode, 这个在第三条讲解

  ```objective-c
  kCCOptionPKCS7Padding | kCCOptionECBMode
  ```

  在明文块末尾补足相应数量的字符，且每个字节的值等于缺少的字符数。

  ```
  比如明文：{B,B,B,B,B,B,B,B,B,B},缺少6个字符，则补全为{B,B,B,B,B,B,B,B,B,B,6,6,6,6,6,6}
  ```

  缺少6个字符，每个字符字节的值等于缺少的字符数6。所以尾部填充 6,6,6,6,6,6

需要注意的是：加密和解密需要使用同一种填充方式，解密后也需要删除掉填充上的数据

##### 3、AES的五种模式

先介绍iOS系统支持的两种模式：系统默认CBC

1. **CBC模式：**

   > 密码分组链模式 Cipher Block Chaining
   >
   > 先将明文切分成若干小段，然后每一小段与初始块或者上一段的密文段进行异或运算后，再与密钥进行加密。

   **优点：**能掩盖明文结构信息，保证相同密文可得不同明文，所以不容易主动攻击，安全性好于ECB，适合传输长度长的报文，是SSL和IPSec的标准。

   **缺点：**（1）不利于并行计算；（2）传递误差——前一个出错则后续全错；（3）第一个明文块需要与一个初始化向量IV进行抑或，初始化向量IV的选取比较复杂。

2. **ECB模式：**

   > 电码本模式  Electronic Codebook Book
   >
   > 将整个明文分成若干段相同的小段，然后对每一小段进行加密。

   **优点：**操作简单，易于实现；分组独立，易于并行；误差不会被传送。——简单，可并行，不传送误差。

   **缺点：**掩盖不了明文结构信息，难以抵抗统计分析攻击。——可对明文进行主动攻击。

#### Hash

RSA之后延伸出来了很多加密算法，就比如Hash

Hash也叫散列函数，也叫数据的指纹，信息摘要。

其实严格来说，并不属于加密算法，因为它不可以解密。

它不可逆，它的作用主要用于对信息的一致性和完整性的校验

Hash算法也叫散列函数，它分多种：MD5、SHA1/256/512

Hash有几个显著特点：

- 这些算法都是算法公开
- 对相同的数据加密得到的结果是一样的
- 对不同的数据（文本、图片、电影等）加密，得到的结果是定长的，比如MD5就是32个字符
- 只能加密、不可逆，不能解密

这些Hash算法都略有不同，MD5 和SHA-1 是单项散列函数的典型代表，拿SHA-1 与 MD5 来比较一下

##### SHA-1 与 MD5区别

因为二者均由MD4改进得来，主要增加了算法的复杂度和不可逆性，他们很相似。

- 对强行攻击的安全性：最显著和最重要的区别是SHA-1摘要比MD5摘要长32 位，产生具有相同摘要的两个报文的难度MD5是2^128数量级的操作，而对SHA-1则是2^160数量级的操作，因而,SHA-1对强行攻击的强度更大。
- 对密码分析的安全性：由于MD5的设计，易受密码分析的攻击，SHA-1显得不易受这样的攻击。
- 速度：在相同的硬件上，SHA-1 的运行速度比 MD5 慢。

 在项目中Hash用到的非常多， 拿最常用的MD5来详细说明使用场景 

##### MD5举例应用

一种被广泛使用的密码散列函数，可以产生出一个128位（16字节）的散列值（hash value），用于确保信息传输完整一致。

终端测试命令：

```shell
md5 -s “字符串” 
```

OC代码框架

```objective-c
#import <CommonCrypto/CommonCrypto.h>
extern unsigned char *CC_MD5(const void *data, CC_LONG len, unsigned char *md)
```

我所接触到的应用场景：密码加密、数据完整性校验。

一般的登录流程：客户端输入密码->对密码md5->提交md5登录。

一般来说服务器并不存储登录密码而是存储md5，但是也有直接存密码的，就会出现传说中，服务器数据泄露，密码泄露。

以前大多软件还支持找回密码，现在几乎见不到了，都是重置密码。

##### MD5没有解密，那它就绝对安全吗？

随便搞一串数字217923012 假设这就是密码

```
$ md5 -s "217923012"             
MD5 ("217923012") = c95d97ed441401a575e0ff0f70eb1de8
```

把得到的md5打开网址https://www.cmd5.com/ 解密一下，发现可以解密。我又拿了我多年使用的字母加数字组合的密码，竟然也可以，细思极恐。

其他加密算法，也都雷同于此。

##### 加盐（salt）方案为此诞生

加盐什么意思呢？[维基百科](https://zh.wikipedia.org/wiki/盐_(密码学))。简单来说，就是把密码217923012用一些鬼画符&……%￥##……**……%，穿插拼接一下，然后再去做md5，这样会极大降低Hash数据库的海量数据匹配度。 

##### 加盐（salt）安全吗？

肯定不安全。再看下网站的可选选项，发现人家也有加盐的匹配方案

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghn5qons90j30wg0pkkei.jpg" alt="image-20200811204446772" style="zoom:30%;" />

比如选项 md5($pass.$salt);Joomla 。其中的 pass就是密码， salt就是盐， 你只要给他盐，他就能帮你解密！

而且这个盐如果在客户端写死的话，逆向是很容易拿到的，就会变的非常不安全

##### ”HMAC加密方案“为此诞生

HMAC 简单来说就是动态盐，一个用户一个盐

客户端到服务器的交互流程：

客户端登录时没有salt，就先向服务器请求，获得salt后，进行盐加密，登录。

换了设备也是这个流程，不过像QQ、微信、apple登录，并不是你向服务器要salt，它就会给，它会先向其他设备发起征求同意的请求，这也就是设备锁。

##### HMAC了还有破绽吗？

还是有的，如果有人通过wifi路由器等方式拿到你的hash值，直接模拟账号发起请求怎么办？

方向是增加复杂度。比如：也可以用一些有规律的202008021211时间年月日时分）和 密码HMAC后的一起拼接再次Hash,服务器校验时也拼接上时间，然后比对，考虑到请求延迟，服务器收到后也获取当前服务器时间，可以做向前一分钟容错。

如果对方逆向Hook到系统的加密函数，直接获取到你的加密前的参数这不就知道原始数据了，这如何防护？？

这就需要app安全方面的知识，反调试、加固（代码混淆）、反重签、越狱检测等等。

#### 破解MD5

##### 暴力枚举法：

> 粗暴地枚举出所有原文，并计算出它们的Hash值，看看哪个Hash值和给定的信息摘要一致。

这种方法虽然简单，但是时间复杂度极高

```
比如：8位仅包含数字和字母的密码。每一位都有62种可能性（A~Z,a~z,0~9）,那么8位的排列组合就是62的8次方，约等于200万亿吓人吧？！ 聪明人可以通过一些取巧，比如优先：生日、有意义的单词等。但也还是不少哦！
```

##### 字典法：

如果说暴力是时间换空间，那么字典就是空间换时间

> https://www.cmd5.com/ 创建的记录约90万亿条，占用硬盘超过500TB。用一个巨大的字典 ，存储了尽可能多的明文和Hash的对应。聪明人可以优先存储一些常用的密码和摘要。

如果你想到上面的200万亿，每一条记录对应192（128+64）bit,那么需要4.65PB（bit, Byte, KB, MB, GB, TB, PB, EB, ZB, BB）存储空间，它才仅超过500TB。

那么如何在这时间和空间两者之间，取到一个平衡呢？ 往下看**彩虹表法**

##### 彩虹表法

> 维基百科：**[彩虹表](https://zh.wikipedia.org/wiki/彩虹表)**是一个用于加密散列函数逆运算的预先计算好的表，常用于破解加密过的密码散列。 彩虹表常常用于破解长度固定且包含的字符范围固定的密码（如信用卡、数字等）。这是[以空间换时间](https://zh.wikipedia.org/wiki/以空間換時間)的典型实践，比[暴力破解](https://zh.wikipedia.org/wiki/暴力破解)（[Brute force](https://en.wikipedia.org/wiki/Brute-force_attack)）使用的时间更少，空间更多；但与储存密码空间中的每一个密码及其对应的哈希值（Hash）实现的查找表相比，其花费的时间更多，空间更少。使用[加盐](https://zh.wikipedia.org/wiki/盐_(密码学))的[密钥派生函数](https://zh.wikipedia.org/wiki/密钥派生函数)可以使这种攻击难以实现。

[彩虹表法相关文章](https://www.cnblogs.com/shoshana-kong/p/11468330.html)
