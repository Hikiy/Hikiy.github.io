---
layout: post
title:  "解决方案：密码加密方案"
date:   2019-05-08 17:17:00 +0200
categories: 解决方案
excerpt:
---

# 密码加密方案

这里使用php语言，当时是在公司使用。  
**密码加密思路**：  

- 数据库字段不保存密码，而是保存一个hash过的字段  
这个字段通过数据库中的几个字段进行hash（例如数据库多加个密码盐值字段、账号创建时间、密码）：将hash条件都拼接成字符串，然后进行hash,例如:

```
$pass = "password={$password}&pass_salt={$passSalt}&created={$created}"
$hashpass = hash('sha256',$pass);
```
- 建议hash多次，这样几乎无法被破解了：

```
for($i=0;$i < 10; $i++){
    $hashpass = hash('sha256',$pass);
}
```

**加密盐值构造思路**：
- 赋值一个字符串，位数长一点，如32位。
- 将该字符串打乱，然后从这个字符串中选择随机位置开始，获取指定数量的字符并结束，例如

```
$salt = substr(str_shuffle($strs), mt_rand(0, strlen($strs)-30), 30);
```
顺便解释一下函数：

```
substr(string,start,length)//String	必需。规定要返回其中一部分的字符串。
                           //start	必需。规定在字符串的何处开始。
                           //       正数 - 在字符串的指定位置开始
                           //       负数 - 在从字符串结尾开始的指定位置开始
                           //       0 - 在字符串中的第一个字符处开始
                           //length 可选。规定被返回字符串的长度。默认是直到字符串的结尾。
                           //       正数 - 从 start 参数所在的位置返回的长度
                           //       负数 - 从字符串末端返回的长度

str_shuffle(string)//必需。规定要打乱的字符串。
mt_rand(min,max)//返回[min,max]中的随机整数
```

当时还有个疑问,项目中在密码这一块却用了php原本有的hash(),但是在一些id上的hash却使用了如下方法，这个方法是php扩展类：

```
use Hashids\Hashids;

$this->hashO = new Hashids('盐值', 10);
$hashid = $this->hashO->encode($id);
$id = $this->hashO->decode($hashid);
```
**其实是对称加密和非对称加密的区别**（能解码和不能解码）

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.05.08  
> 更新日期：2019.05.08
