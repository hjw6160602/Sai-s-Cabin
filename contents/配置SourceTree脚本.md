---
title: 配置SourceTree脚本
date: 2017-09-12 16:37:46
---

## 首先我们来看下SourceTree的Custom Actions是什么？
Custom Actions是SourceTree提供给用户去执行自己的一些脚本的一些设置。首先打开SourceTree，点击菜单栏上的Preferences（或者使用Command＋，），点击Custom Actions就可以看到了。样式如下图

![SourceTree01](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/source_tree_01.png) 

可以看到上面装载了一些我自定义的脚本。以第一个脚本为例，”lvUpdateTags“为脚本的名字，第二部分为脚本的路径，第三部分为脚本执行所需参数，第四部分为在自定义脚本中不会用到，可以忽略。脚本内设置如下图，可以点击打开打印全部输出等。Parameters部分所传参数可以自定义，也可以传SourceTree定义好的，在图上都给出了解释。

![SourceTree02](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/source_tree_02.png) 

如图所示的方式可以配置自定义的脚本，但是如果想简化这个过程，把你的所有配置打包，一个命令安装，就需要其他方式。

## 一行命令配置所有自定义脚本
首先找到SourceTree存储这些命令的文件，如下图

![SourceTree03](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/source_tree_03.png) 

上面的路径就是SourceTree存储自定义脚本的文件，文件的内容如下图。

![SourceTree04](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/source_tree_04.png) 

这就是actions.plist所存储的内容，可以看到标注红框的地方都是自定义脚本的名字或路径。
这样理解就是只需要把整个plist文件按照自己的配置部署好，那么SourceTree就可以读取到所有的自定义脚本。那么该用什么格式存储信息呢，又该怎么写入plist中呢？
这里我们选用json文件存储，首先json是key－value存储的，很方便理解和添加；其次json文件可以通过程序解析成字典，然后在程序中将读入的json转化为字典，写入plst文件即大功告成了。
下面是json文件源码
```
[
    {
        "name": "lvUpdateTags",
        "target": "~/SourceTreeScript/lvUpdateTags.sh",
        "params": "$REPO"
    },
    {
        "name": "lvUpdatePod",
        "target": "~/SourceTreeScript/lvUpdatePod.sh",
        "params": "$REPO"
    },
    {
        "name": "lvUpdatePod_NoRepoUpdate",
        "target": "~/SourceTreeScript/lvUpdatePod_NoRepoUpdate.sh",
        "params": "$REPO"
    },
    {
        "name": "lvUpdateRemotes",
        "target": "~/SourceTreeScript/lvUpdateRemotes.sh",
        "params": "$REPO"
    },
    {
        "name": "lvOpenWorkspace",
        "target": "~/SourceTreeScript/lvOpenWorkspace.sh",
        "params": "$REPO"
    },
    {
        "name": "lvCreateMergeRequest",
        "target": "~/SourceTreeScript/lvCreateMergeRequest.sh",
        "params": "$REPO"
    }
]
```
   


## 解析配置Custom Actions的命令
为了达到一行命令配置Custom Actions，那就需要写一个命令，利用Xcode完成一个命令的编写。
下面是源码

```
 #import <Foundation/Foundation.h>
     NSMutableDictionary *getActionDic(NSString * name, NSString *params, NSString *target) {
    NSMutableDictionary *action = [NSMutableDictionary dictionaryWithDictionary:@{@"fileAction":@(0), @"logAction":@(0), @"name":name, @"params":params, @"repoAction":@(1), @"separateWindow":@(0), @"shortcutKeyCode":@"-1", @"shortcutKeyDisplay":@"", @"shortcutKeyModifiers":@(0), @"showFullOutput":@(1), @"target":target}];
    return action;
     }
     int main(int argc, const char * argv[]) {
    @autoreleasepool {
        if (argc < 2) {
            printf("请输入一个配置自定义脚本的json文件路径\n");
            return 1;
        }
        NSError *jsonError = nil;
        NSString *jsonFilePath = [NSString stringWithFormat:@"%s", argv[1]];
        id jsonObject = [NSJSONSerialization JSONObjectWithData:[NSData dataWithContentsOfFile:jsonFilePath] options:NSJSONReadingMutableContainers error:&jsonError];
        if ([jsonObject isKindOfClass:[NSArray class]]) {
            NSMutableArray *actions = [NSMutableArray array];
            for (NSDictionary *actionDic in jsonObject) {
                NSString *name = [actionDic objectForKey:@"name"];
                NSString *params = [actionDic objectForKey:@"params"];
                NSString *target = [actionDic objectForKey:@"target"];
                if (!name || !params || !target) {
                    printf("获取不到应该有的参数\n");
                    return 1;
                }else {
                    [actions addObject:getActionDic(name, params, target)];
                }
            }
            NSString *configPath = [@"~/Library/Application Support/SourceTree/actions.plist" stringByStandardizingPath];
            NSMutableData *data = [NSMutableData data];
            NSKeyedArchiver *archiver = [[NSKeyedArchiver alloc] initForWritingWithMutableData:data];
            [archiver setOutputFormat:NSPropertyListXMLFormat_v1_0];
            [archiver encodeObject:actions forKey:@"root"];
            [archiver finishEncoding];
            if(data) {
                BOOL result = [data writeToFile:configPath atomically:YES];
                if (result) {
                    NSArray *readActions = [NSKeyedUnarchiver unarchiveObjectWithFile:configPath];
                    printf("目前plist中已经存在的action\n%s\n", [[readActions description] UTF8String]);
                }
            } else {
                printf("写入错误\n");
            }
        }
    }
    return 0;
}
```
再完成上面两步操作之后就可以在终端中执行configActions命令，再跟一个json文件的路径参数，即可完成Custom Actions的配置。

## 最终版简化命令
需要用到setup.sh文件，内容如下

```
 #!/bin/bash
     echo '*************************'
     echo "setup start"
     if [[ -d "$HOME/SourceTreeScript" ]]; then
       rm -rf "$HOME/SourceTreeScript"
    fi
    mkdir "$HOME/SourceTreeScript"
    cd "`pwd`/SourceTreeScriptTemp"
    for file in `find . -name "*.sh"`
    do
      fileFullName=`basename $file`
      echo "$fileFullName"
      if [[ ! "$fileFullName" = "setup.sh" ]]; then
      	cp $file "$HOME/SourceTreeScript"
      fi
    done
    cp "`pwd`/create_mergerequest" "/usr/local/bin/create_mergerequest"
    "`pwd`/configActions" "`pwd`/config.json"
    echo '*************************'
    cd -
```
    
将源码上传到git，下载代码和部署一条龙完成。

```
rm -rf SourceTreeScriptTemp && git clone http://lvioscode.com/ios_support_tools/SourceTreeScript.git SourceTreeScriptTemp && source "`pwd`/SourceTreeScriptTemp/setup.sh" && rm -rf SourceTreeScriptTemp
```
