---
title: 证书格式和HTTPS设置
date: 2017-12-6 19:31:14
---


[一、HTTPS的方式](#一https的方式)

* [SSL](#ssl)
* [OpenSSL](#openssl)
* [X.509 证书标准](#x509-证书标准)
    
[二、证书编码格式](#二证书编码格式)

* [2.1 PEM](#21-pem)
* [2.2 DER](#22-der)

[三、相关的文件扩展名](#三相关的文件扩展名)

* [CRT](#crt)
* [CER](#cer)
* [KEY](#key)
* [CSR](#csr)
* [P12(PFX)](#p12pfx)
* [JKS](#jks)

[四、证书编码的转换](#四证书编码的转换)

* [4.1 PEM转为DER](#41-pem转为der)
* [4.2 DER转为PEM ](#42-der转为pem)

[五、获得证书](#五获得证书)

* [向权威证书颁发机构申请证书](#向权威证书颁发机构申请证书)
* [生成自签名的证书](#生成自签名的证书)

[六、iOS中实现校验HTTPS证书](#六ios中实现校验https证书)

* [AFSecurityPolicy](#afsecuritypolicy)

[七、校验证书之后的App如何通过Charles抓包](#七校验证书之后的app如何通过charles抓包)

## 一、HTTPS的方式

###  SSL
`Secure Sockets Layer`，现在应该叫`TLS`( `Transport Layer Security` ) 安全传输层协议 ，但由于习惯问题，我们还是叫`SSL`比较多。`HTTP`协议默认情况下是不加密内容的，这样就很可能在内容传播的时候被别人监听到，对于安全性要求较高的场合，必须要加密，`HTTPS`就是带加密的`HTTP`协议，而`HTTPS`的加密是基于`SSL`的，它执行的是一个比较下层的加密，也就是说，在加密前，你的服务器程序在干嘛，加密后也一样在干嘛，不用动，这个加密对用户和开发者来说都是透明的。

###  OpenSSL

简单地说，`OpenSSL`是`SSL`的一个实现，`SSL`只是一种规范。理论上来说，`SSL`这种规范是安全的，目前的技术水平很难破解，但`SSL`的实现就可能有些漏洞。`OpenSSL`还提供了一大堆强大的工具软件，90%我们都用不到。

### X.509 证书标准
`X.509` 是由国际电信联盟（`ITU-T`）制定的数字证书标准，是一种广泛使用的数字证书标准，该标准定义了证书中应该包含哪些内容。`SSL` 使用的就是这种证书标准，编码格式同样的`X.509` 证书。

## 二、证书编码格式
>
* PEM ： `BASE64`编码的证书
* DER ：二进制格式的证书

### 2.1 PEM
`Privacy Enhanced Mail` 打开看文本格式

 `-----BEGIN...` 开头

  `-----END...` 结尾
  
  内容是`BASE64`编码。查看`PEM`格式证书的信息:

```shell
openssl x509 -in certificate.pem -text -noout
```

`Apache`和 `*NIX`服务器偏向于使用这种编码格式。

### 2.2 DER
`Distinguished Encoding Rules` 打开看是二进制格式，不可读。`Java`和`Windows`服务器偏向于使用这种编码格式。

### 三、相关的文件扩展名

>
* CRT ：两种编码都有可能，大多数是`PEM`编码
* CER：两种编码都有可能，大多数是`DER`编码
* KEY ：两种编码都有可能，存放公钥或者私钥
* CSR ：证书签名请求（不是证书）
* P12(PFX) ：包含了证书及私钥
* JKS ：包含了证书及私钥（Java的专利 ）
>

`PEM`和`DER`这两种编码格式导出的文件扩展名并不一定叫`PEM`或者`DER`，常见的扩展名还有以下这些，它们除了编码格式可能不同之外，内容也有差别，但大多数都能相互转换编码格式。

### CRT
`CRT` 是 Certificate 的三个字母，证书的意思，常见于 `*NIX`系统，有可能是`PEM`编码，也有可能是`DER`编码，大多数应该是`PEM`编码。

### CER
`CER` 还是 Certificate，还是证书，常见于`Windows`系统，同样的，可能是`PEM`编码，也可能是`DER`编码，大多数应该是`DER`编码。

### KEY
`KEY` 通常用来存放一个公钥或者私钥，并非`X.509`证书，编码同样的，可能是`PEM` 也可能是`DER`。查看`KEY`的办法：

``` shell
openssl rsa -in mykey.key -text -noout
```

如果是`DER`格式的话，同理应该这样了：

``` shell
openssl rsa -in mykey.key -text -noout -inform der
```

### CSR
`Certificate Signing Request` 即证书签名请求，这个并不是证书，而是向权威证书颁发机构获得签名证书的申请，其核心内容是一个公钥（当然还附带了一些别的信息），在生成这个申请的时候，同时也会生成一个私钥，私钥要自己保管好。做过iOS APP的都应该知道是怎么向苹果申请开发者证书的。
查看的办法：

``` shell
openssl req -noout -text -in my.csr
``` 

（如果是DER格式的话照旧加上`-inform der`，这里不写了）

### P12(PFX)
`Predecessor of PKCS#12` 对 `*nix` 服务器来说，一般 `CRT` 和 `KEY` 是分开存放在不同文件中的，但Windows的IIS则将它们存在一个`PFX`文件中，（因此这个文件包含了证书及私钥）这样会不会不安全？应该不会，PFX通常会有一个“提取密码”，你想把里面的东西读取出来的话，它就要求你提供提取密码，`PFX`使用的时`DER`编码，如何把`PFX`转换为`PEM`编码？

``` shell
openssl pkcs12 -in for-ii.pfx -out for-iis.pem -nodes
```

这个时候会提示你输入提取代码。 `for-iis.pem`就是可读的文本。生成`pfx`的命令类似这样:

``` shell
openssl pkcs12 -export -out certificate.pfx -inkey privateKey.key -in certificate.crt -certfile CACert.crt
```

其中`CACert.crt`是`CA`（权威证书颁发机构）的根证书，有的话也通过-certfile参数一起带进去。这么看来，PFX其实是个证书密钥库。

### JKS
`JKS`- 即`Java Key Storage`，这是`Java`的专利，跟`OpenSSL`关系不大，利用`Java`的一个叫`keytool`的工具，可以将`PFX`转为`JKS`，当然了，`keytool`也能直接生成`JKS`。

## 四、证书编码的转换


### 4.1 `PEM`转为`DER` 

``` shell
openssl x509 -in cert.crt -outform der -out cert.der
```

### 4.2 `DER`转为`PEM`

``` shell
openssl x509 -in cert.crt -inform der -outform pem -out cert.pem
```

（提示:要转换`KEY`文件也类似，只不过把`x.509`换成`rsa`，要转`CSR`的话，把`x.509`换成`req`）

## 五、获得证书

### 向权威证书颁发机构申请证书
用这命令生成一个`CSR`：

``` shell
openssl req -newkey rsa:2048 -new -nodes -keyout my.key -out my.csr
```

把`CSR`交给权威证书颁发机构，权威证书颁发机构对此进行签名，完成。保留好`CSR`，当权威证书颁发机构颁发的证书过期的时候，你还可以用同样的`CSR`来申请新的证书，key保持不变。

### 生成自签名的证书

``` shell
openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem
```

在生成证书的过程中会要你填一堆的东西，其实真正要填的只有`Common Name`，通常填写你服务器的域名，如`lvmama.com`，或者你服务器的IP地址，其它都可以留空的。生产环境中还是不要使用自签的证书，否则浏览器会不认，或者如果你是企业应用的话能够强制让用户的浏览器接受你的自签证书也行。向权威机构要证书通常是要钱的，但现在也有免费的，仅仅需要一个简单的域名验证即可。

### 导出驴妈妈的证书

打开火狐浏览器 输入`https://api3g2.lvmama.com/`，点击地址栏前面的小锁图标

![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/Snip20171130_2.png)

![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/Snip20171130_3.png)

![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/Snip20171130_4.png)

![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/Snip20171130_7.png)

![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/Snip20171130_8.png)

![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/Snip20171130_10.png)

最终导出证书到本地。

## 六、iOS中实现校验HTTPS证书

将导出到本地的证书文件`.crt`结尾的重命名为`.cer`格式的证书，然后加入到本地的Bundle包中。这里之所以将证书格式设置为`.cer`结尾，是因为`AFN`只会去读取`.cer`格式的证书文件。

**注意在导入证书的时候一定概要将 `Target Membership` 勾选上才会存在于App中**

`AFNetworking`中实现了校验`SSL`证书的过程只需要将`AFHTTPSessionManager`的`securityPolicy`属性赋值即可。

下面来看`AFSecurityPolicy`这个类的一些属性和内容。

### AFSecurityPolicy
    
* `allowInvalidCertificates` 是否信任无效或者过期的证书（默认是`NO`）  
* `validatesDomainName`是否验证证书的域名（默认是`YES`） 
* `SSLPinningMode` (**readonly**) 初始化`securityPolicy`对象的时候赋值
    * `AFSSLPinningModeNone` 不使用pin到的证书来验证服务器
    * `AFSSLPinningModePublicKey` 通过pin到的证书中的公钥来验证host证书
    * `AFSSLPinningModeCertificate` 通过pin到的证书来验证host证书

方法`[AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeCertificate]`会在生成`AFSecurityPolicy`的实例化对象`securityPolicy`的时候去App的Bundle包中寻找`.cer`结尾的证书文件作为校验项。

#### NSSet \<NSData *> *pinnedCertificates

`pinnedCertificates`根据`SSL`的`pinning mode`来评估服务器是否可信任的证书
 这个属性默认会在编译`AFNetworking`的`target`中去寻找任意的以`.cer`为后缀的证书文件
 
``` objc
+ (NSSet *)defaultPinnedCertificates {
    static NSSet *_defaultPinnedCertificates = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        NSBundle *bundle = [NSBundle bundleForClass:[self class]];
        _defaultPinnedCertificates = [self certificatesInBundle:bundle];
    });

    return _defaultPinnedCertificates;
}
```
 
 > 提示：如果你是以嵌入`framework`的方式来使用`AFNetworking`，那么默认将不会 `pin` 到证书，需要使用 `certificatesInBundle:bundle`  方法来从target中加载证书，然后创建一个policy对象来调用`policyWithPinningMode:withPinnedCertificates`方法 `pin` 到证书。
 
如果pinning是激活的，那么 `evaluateServerTrust:forDomain:` 方法会在匹配到任一证书之后返回true。
 
**Tips：** 在早起的AFN版本中，可能会有证书链的设置`validatesCertificateChai`，最新的AFN里面已经去掉了证书链。

> Full Certificate Chain Validation has been removed from AFSecurityPolicy. As discussed in #2744, there was no documented security advantage to pinning against an entire certificate chain. If you were using full certificate chain, please determine and select the most ideal certificate in your chain to pin against.
>

`2.6.0`版本的AFN中去掉了证书链的验证。在[`#2744`](https://github.com/AFNetworking/AFNetworking/issues/2744)这个github的issue里讨论了 去校验整个证书链并没有增加任何安全性。也没有任何能够证明对证书链的校验会增加安全性的文档。

> I spoke with the Apple Engineering Security team in depth last week in the WWDC labs, and they agreed there was no additional security value in validating the entire chain. Their own implementation pins against only one certificate in the chain. It's important to understand the trade offs of what level to pin against, but once that decision is made, there is no value in pinning against a higher cert once that has been done.
Unless someone can provide a specific vulnerability that pinning against the entire chain would provide more security than pinning against a single cert, I'll be deprecating the entire chain validation in a future release.

## 七、校验证书之后的App如何通过Charles抓包

**如果是自己的模块：**
只需要在自己的模块里面的`AppDelegate`中加入

```objc
[[TMCache sharedCache] setObject:@(YES) forKey:@"validateServer"];
```
这句代码即可。

**如果是主工程：** 不需要加代码，需要在`DebugViewController`里打开Charles抓包开关即可。

<img src="http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/Simulator Screen Shot - iPhone 8 Plus - 2017-11-30 at 14.42.42.png" width="414" height="736">

**如果是在真机上跑，并且系统在iOS11以上，在下载了`Charles`的证书之后，还需要打开信任证书的开关。**

<img src="http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/WechatIMG119.jpeg" width="414" height="736">

**采用HTTPS证书校验带来的好处**

* 使用证书校验认证用户和服务器，确保数据发送到正确的客户机和服务器；
* 防止数据在传输过程中不被窃取、改变，确保数据的完整性。
* 报文无法被黑客抓取和修改，大幅增加了中间人攻击的成本。

参考文档：

https://blog.myssl.com
https://www.sslshopper.com/ssl-converter.html

http://www.360doc.com/content/15/0520/10/21412_471902987.shtml
