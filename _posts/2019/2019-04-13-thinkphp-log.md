---
layout: post
title:  "ThinkPHP学习记录：日志"
date:   2019-04-13 11:17:00 +0200
categories: ThinkPHP
excerpt: 
tagg: phpframwork
---

# 日志

### 日志驱动
```
'log'   => [
    // 日志记录方式，支持 file socket
    'type' => 'File',
    //日志保存目录
    'path' => LOG_PATH,
    //单个日志文件的大小限制，超过后会自动记录到第二个文件
    'file_size'     =>2097152,
    //日志的时间格式，默认是` c `
    'time_format'   =>'c'
],
```
### 日志写入
```
Log::record('操作日志写入失败！内容为新增商品类别','mongo');



Log::save()
Log::write()
```

上述例子中，第一个参数为日志内容，第二个为日志级别。这个完全是可以自定义的。  
>系统在请求结束后会自动调用Log::save方法，所以通常，你只需要调用Log::record记录日志信息即可。

- log 常规日志，用于记录日志
- error 错误，一般会导致程序的终止
- notice 警告，程序可以运行但是还不够完美的错误
- info 信息，程序输出信息
- debug 调试，用于调试信息
- sql SQL语句，用于SQL记录，只在数据库的调试模式开启时有效


### 独立日志
为了便于分析，File类型的日志驱动还支持设置某些级别的日志信息单独文件记录，例如：
```
'log'   => [
    'type'          => 'file', 
    // error和sql日志单独记录
    'apart_level'   =>  ['error','mongo'],
],
```
设置后，就会单独生成```error```和```mongo```两个类型的日志文件，主日志文件中将不再包含这两个级别的日志信息。
示例：  
![image 独立日志示例](https://note.youdao.com/yws/public/resource/41f1e1d015a5bbb8475c9e282f944ece/xmlnote/5200FA16FF464F2C9F5E75743FD02CDF/13199)

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.04.13  
> 更新日期：2019.04.13
