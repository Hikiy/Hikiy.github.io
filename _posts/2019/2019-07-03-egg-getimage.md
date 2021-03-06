---
layout: post
title:  "egg.js学习记录：获取前端上传的文件"
date:   2019-07-03 15:15:00 +0200
categories: eggjs
excerpt: 上传多个文件有巨大的坑！！！
tagg: nodeframwork
---

# 获取前端上传的文件

如果是单个文件，就看官方文档够了：https://eggjs.org/zh-cn/basics/controller.html#%E8%8E%B7%E5%8F%96%E4%B8%8A%E4%BC%A0%E7%9A%84%E6%96%87%E4%BB%B6 

如果是多个文件，会有坑，继续往下看

PS：egg.js为了安全起见，有一些文件格式需要在配置文件中扩展才能上传，否则会失败,例如证书文件`.p12`

```
config.multipart = {
        fileExtensions: [ '.p12' ] // 增加对 p12 扩展名的文件支持
    };
```

## 需求

当时遇到坑时的需求，在公司加班到晚上十二点多也没解决，第二天才解决。

- 从前端获取**多个文件**和**多个参数**
- 使用参数生成唯一标识
- 以唯一标识为目录将文件上传至阿里云OSS

## 问题

虽然官方文档提供了获取的方法。但是因为需求比较特殊，需要从前端获取多个文件和参数，然后需要一次性取完多个文件和参数，然后代码通过参数生成一些数据后才真正使用文件。

官方文档中多文件流的获取方式是
```
const parts = ctx.multipart();

while ((part = await parts()) != null) {
    ...
}
```
。每次调用则获取一个参数或者一个文件流。使用while语句调用即可调用完。

### 浏览器pending问题

**但是这时候出现了问题**，每次调用parts()方法后获取的part需要被消费掉。不消费掉将会导致浏览器卡死。这里的消费好像是要传到OSS或者之类的操作，然后我先将它放在了一个数组中，这样并不是消费掉。于是需要手动消费`await sendToWormhole(part);`

### 文件销毁问题

**但是经过测试**，经过了`await sendToWormhole(part);`方法后，存在数组中的文件流也被销毁了。所以这个`multipart()`方法不了了之

### POST的参数问题

上传文件时，还需要前端提供一些POST参数，文件需要`form-data`格式，所以POST参数也使用form-data格式。但是node.js框架默认获取的POST是`x-www-form-urlencoded`格式。所以POST参数默认传不上，使用上面的`parts()`也可以获取到参数，但现在着重处理文件问题。

补充：如果使用了parts()方法，在获取参数的时候可以设置成统一获取：

```
        const parts = ctx.multipart({ autoFields: true });
          while ((part = await parts()) != null) {
            if (part.length) {
            } else if (part.filename) {
              const tmpFile = tmp.fileSync({prefix: 'eggjs-upload-'})
              const ws = fs.createWriteStream(null, {fd:tmpFile.fd})
              part.pipe(ws)
              part.path = tmpFile.name
              part.fd = tmpFile.fd
              files.push(part)
            }
          }
          fields = parts.field
```

注意代码中的`autoFields: true`和`fields = parts.field`。

## 解决方案
从网上找到了一个方案，是在中间件中对header进行判断，然后将文件进行临时存储，类似于PHP中上传文件了。

### 新建中间件
```
/*
  *中间件，作用是把表单方式上传的参数和文件预处理后再写入request.body和ctx.request.upload_files,相当于post数据了
 */
const formidable = require("formidable");
module.exports = options => {
  return async function multipart_upload(ctx, next) {

    var params = {} ;
    if (ctx.request.header['content-type'] && ctx.request.header['content-type'].indexOf("multipart") > -1 ){
        var parse = async function(req) {
          const form = new formidable.IncomingForm();
            return new Promise((resolve, reject) => {
            form.parse(req, (err, fields, files) => {
            resolve({ fields, files })
            })
          });
        }
        const extraParams = await parse(ctx.req);
        if (extraParams) {
          if (extraParams.fields) {
            //将参数合并到body中，这样就可以相当于post数据了
            ctx.request.body = Object.assign(ctx.request.body,extraParams.fields);
          };

          if (extraParams.files) {
            //将上传文件信息合并，这样在控制器里就可以通过ctx.request.upload_files获取了
            ctx.request.upload_files = extraParams.files ;
          };
        };

    }

    await next(); //执行下一步
  };
};
```

### 在Controller层获取
```
let name = ctx.request.upload_files.name;
let path = name.path;
```

egg.js还是这么多坑啊！！！！

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.03  
> 更新日期：2019.07.03
