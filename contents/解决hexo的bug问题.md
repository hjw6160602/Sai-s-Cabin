---
title: 解决hexo的报错bug
date: 2018-03-09 17:35:25
tags: 错题本
---


每次启动hexo的时候都有这个报错，很是头疼
![](http://p4yixtz2j.bkt.clouddn.com/1520588306.png )
百度了一下发现 

```shell
npm install hexo --no-optional
```

这行命令没啥用，后来去了stackoverflow发现了一个回答

This worked for me after switching to Node 6.1 (and when re-installing node modules didn't work):

Install and save dtrace-provider

```shell
$ npm install dtrace-provider --save
```

Delete 'node_modules' folder

Re-install node modules

```shell
$ npm install
```

I found this thread before combining your attempts with another solution on the Github project issues for restify (https://github.com/restify/node-restify/issues/1093) and simplified best as possible.

还是没啥用，但是得到了灵感，我用这个方法首先是报错：

<img src="http://p4yixtz2j.bkt.clouddn.com/1520588483.png" width="800" height="600">

然后就到~/.npmrc文件中修改npm的源

```javascript
registry=https://registry.npmjs.org
```

将淘宝的源改为官方的，然后重新执行上面的`npm install dtrace-provider --save` 就安装下来的，直接到
`/usr/local/lib/node_modules/hexo-cli/node_modules/`目录下把`dtrace-provider`文件夹直接替换掉，报错终于解决了！

