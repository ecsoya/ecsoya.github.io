---
layout: post
date:   2018-11-21
categories: [scf]
title: "腾讯云API网关实践"
---

> API 网关（API Gateway）是 API 托管服务，提供 API 的完整生命周期管理，包括创建、维护、发布、运行、下线等。您可使用 API Gateway 
封装自身业务，将您的数据、业务逻辑或功能安全可靠的开放出来，用以实现自身系统集成、以及与合作伙伴的业务连接。


### 简单原理

API网关是有一个个服务组成的，而每一个服务里可以创建多个API。

每一个API都是由`前端`和`后端`组成。

* 前端：主要用来配置请求的`类型`，`方法`，`路径`，`参数`等。
* 后端：可以支持`http`，`mock`和`腾讯云无服务器云函数`。

当配置了后端为`腾讯云无服务器云函数`之后，就可以通过前端配置的URL去加载你的账号中的所有的云函数了。

### 注意事项

1. 支持CORS：
   默认情况下，API网关都是支持跨域访问的。
2. 鉴权类型：
    * 密钥对：在访问API网关的时候，需要添加`签名验证`。
    * 免鉴权：无条件的访问API网关。

### 密钥对鉴权

什么是密钥对鉴权？详细信息请参考官方[参考文档](https://cloud.tencent.com/document/product/628/11819)和[示例](https://github.com/apigateway-demo)。

简单来说，就是在每次请求的时候生成一个`Authorization header`。
而这个header的具体内容如下，并且格式要完全保持一致（注意逗号和空格）。

`Authorization: hmac id="secret_id", algorithm="hmac-sha1", headers="date source", signature="Base64(HMAC-SHA1(signing_str, secret_key))"`

1. `hmac`：这是固定内容，用来标识算法。
2. `id`：在API网关的`密钥`中生成的`密钥名`。
3. `algorithm`：加密算法的类型，目前只支持`hmac-sha1`。
4. `headers`：参与签名计算的 header，按实际计算时的顺序排列，headers里面必须要包含`date`或是`x-date`里面的一个，用于表示请求构造的时间，且为GMT格式（如：Fri, 09 Oct 2015 
00:00:00 GMT）。
5. `signature`：计算签名后得到的签名。

#### 实现方法：

一， 得到当前系统时间，并转为GMT格式。

```javascript
    let nowDate = new Date();
    let dateTime = nowDate.toGMTString();
```
二，将headers拼串。

比如我们参与签名的header有2个：`X-Date`和`Source`，则拼串的代码为：

```javascript
   let signStr = "x-date: " + dateTime + "\n" + "source: " + source;
```
拼好的串格式如下，注意换行（用ASCII的换行符`\n`）

```
X-Date:Fri, 09 Oct 2015 00:00:00 GMT
Source:MyApp
```
三，生成签名

将上一步中拼好的串使用`Base64(HMAC-SHA1(signing_str, secret_key))`签名。

```javascript
    let sign = CryptoJS.HmacSHA1(word, SECRET_KEY);
    return CryptoJS.enc.Base64.stringify(sign);
```
四，生成`Authorization header`

将第二步生成的header和第三步生成的签名以及secretId一起拼成`Authorization header`。

五，发送请求

将第二步中用到的headers和生成的签名全部放到request的头信息里

```
X-Date:Fri, 09 Oct 2015 00:00:00 GMT
Source:MyApp
Authorization: hmac id="secret_id", algorithm="hmac-sha1", headers="x-date source", signature="Base64(HMAC-SHA1(signing_str, secret_key))"
```

##### 更多：
* [参考源码](https://github.com/ecsoya/tencent-function-gateway)
* [腾讯云无服务器云函数实践](http://blog.ecsoya.work/%E8%85%BE%E8%AE%AF%E4%BA%91/2018/11/20/scf.html)
* [基于SpringBoot的腾讯云函数调试](http://blog.ecsoya.work/%E8%85%BE%E8%AE%AF%E4%BA%91/2018/11/22/scf-boot.html)
* [基于腾讯云对象存储，构建自动上传无服务函数的Maven插件](http://blog.ecsoya.work/%E8%85%BE%E8%AE%AF%E4%BA%91/2018/11/24/scf-maven.html)
