
# 1.[环境配置](#jenkins_env)
# 2.[自主打包任务](#createjob)
# 3.[xib勾选Target检查](#checkxib)
# 4.[创建新分支流程](#newtag)

***

#Jeknins环境的安装跟部署 {#jenkins_env}

1.首先安装最新macos，安装最新Xcode

###安装Brew ：
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"


###安装zsh ：
* 自动安装：
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
* 手动安装：
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc


###安装Xcode Command Line Tools
执行命令：xcode-select --instal 见图
![安装图片](jenkins_images/Xcode-Command-Line-Tools.gif)


安装好了，执行 gcc -v 查看测试结果如下：
Configured with: --prefix=/Applications/Xcode.app/Contents/Developer/usr --with-gxx-include-dir=/usr/include/c++/4.2.1
Apple LLVM version 8.1.0 (clang-802.0.42)
Target: x86_64-apple-darwin16.7.0
Thread model: posix
InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin

###安装 brew update
直接执行 brew update


###安装rvm

打开终端: $ curl -L https://get.rvm.io | bash -s stable

期间可能会问你sudo管理员密码，以及自动通过homebrew安装依赖包，等待一段时间后就可以成功安装好 RVM。然后，载入 RVM 环境（新开 Termal 就不用这么做了，会自动重新载入的)
 
$ source ~/.rvm/scripts/rvm

rvm install 2.3.3 (有时网络有问题，多试几次)

检查是否安装成功: $ rvm -v
若输入上面口令后显示: rvm 1.29.2 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io/]
表明安装成功
 
成功后安装ruby:  $ rvm list known
此时会打印很多字符(里面有ruby版本号)
 
可以选择现有的rvm版本来进行安装（假设以rvm 2.3.0版本的安装为例）
$ rvm install 2.0.0（可能需要管理员权限，使用sudo. $ sudo rvm install 2.0.0）
 
备注:
查询已经安装的ruby: $ rvm list
卸载一个已安装版本 : $ rvm remove 2.0.0
RVM 装好以后，若以前已存在旧版本,需要执行下面的命令将指定版本的 Ruby 设置为系统默认版本
$ rvm 2.3.0 --default

![安装图片](jenkins_images/rvm-install-1.gif)
![安装图片](jenkins_images/rvm-install-2.gif)
![安装图片](jenkins_images/rvm-install-3.gif)



###安装fastlane
执行命令 sudo gem install fastlane --verbose
安装完成后执行： fastlane  -version  显示fastlane 2.53.1 则表示安装成功
![安装图片](jenkins_images/install-fastlane.gif)

###安装Java环境
brew cask install java

###安装Jenkins
brew install jenkins 
启动jenkins ：brew services restart jenkins

###配置jenkins
1.配置启动参数

cat ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist 
cat ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist > ~/Library/LaunchAgents/homebrew.mxcl.jenkins2.plist
vi ~/Library/LaunchAgents/homebrew.mxcl.jenkins2.plist
brew services stop jenkins
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.jenkins2.plist
sudo reboot

如果想网内所有机器都可以访问的话，改成0.0.0.0，端口也是自定义，默认8080
![安装图片](jenkins_images/JenkinsIP.png)

2.创建view
![安装图片](jenkins_images/jenkins_View.gif)
按照图中的操作步骤，即可完成View的创建。

####3.创建项目任务 {#createjob}
![安装图片](jenkins_images/jenkins_createView.gif)
按照图中的操作步骤，即可完成相对应的配置，然后选择构建。

4.配置View的的参数
主要的配置参数见如下图：
![安装图片](jenkins_images/Snip20170904_2.png)
![安装图片](jenkins_images/Snip20170904_3.png)
![安装图片](jenkins_images/Snip20170904_5.png)
![安装图片](jenkins_images/Snip20170905_1.png)
![安装图片](jenkins_images/Snip20170905_2.png)


目录文件夹(~/local/bin/)下面需要有 AppResig(重签名)
,LVPlistToHtml(将plist文件转换成习惯阅读的html文件),LVSFTP(将文件上传到下载服务器中的程序),resign_configuration_files(该文件夹是在证书企业重签名会用到，需要从下载服务器中完整拷贝下来该文件夹以及其中包含的所有文件)

3.需要安装的插件
Gitlab Hook Plugin，Locale plugin，Xcode integration，AnsiColor


###导出证书并添加证书
1.将可打包机器中的证书导出来,双击添加到本机的证书中。
2.即可执行构建计划
注意事项：需要将codesign操作证书的权限允许
![安装图片](jenkins_images/jenkins_keychains.gif)



#自主打包任务
如上面配置项目任务,跳转到[项目任务](#createjob)
具体打包时间计划由构建触发器决定。



#xib勾选Target检查 {#checkxib}
LvmmCheckXibSelectTarget.rb校验检查xib是否勾选Targe选项
![安装图片](jenkins_images/Snip20170905_3.png)


#创建新分支的流程 {#newtag}

创建工作目录->下载主工程代码(cloneMainProject)->下载源(cloneSpec)->修改主工程的版本号BuldleIdVersion(modifyMainProjectBundleIdentifier)->执行创建子模块的分支操作(allModuleProjectDo)->验证新版本分支能否编译成功(verifyMainProjectNewBranch)->提交本地新的分支(mainProjectCommitNewBranch)->代码提交,实现源同步(mainProjectSpecSync)->将主工程切换到需要打Tag分支的上,并校验是否正确(mainProjectChangePodfileNewTag)->提交工程代码(mainProjectCommitNewTag)->打上Tag,同时删除旧的分支(mainProjectCreateNewTagAndDeleteOldBranch)->结束流程

```flow
st=>start: 创建工作目录
c1=>condition: 下载主工程代码
                (cloneMainProject)
c2=>condition: 下载源
                (cloneSpec)
c3=>condition: 修改主工程的版本号
                (modifyMainProjectBundleIdentifier)
c4=>condition: 执行创建子模块的分支操作
                (allModuleProjectDo)
c5=>condition: 验证新分支能否编译成功
                (verifyMainProjectNewBranch)
c6=>condition: 提交新的分支
                (mainProjectCommitNewBranch)
c7=>condition: 源同步
                (mainProjectSpecSync)
c8=>condition: 切换到需要打Tag分支
                (mainProjectChangePodfileNewTag)
c9=>condition: 提交工程代码
                (mainProjectCommitNewTag)
c10=>condition: 打上Tag,同时删除旧的分支
                (mainProjectCreateNewTagAndDeleteOldBranch)
e=>end: 结束流程

st->c1(yes)->c2(yes)->c3(yes)->c4(yes)->c5(yes)->c6(yes)->c7(yes)->c8(yes)->c9(yes)->c10(yes)->e
c1(no)->e
c2(no)->e
c3(no)->e
c4(no)->e
c5(no)->e
c6(no)->e
c7(no)->e
c8(no)->e
c9(no)->e

```
<svg height="3145.888671875" version="1.1" width="553.734375" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 553.734375 3145.888671875" preserveAspectRatio="xMidYMid meet" style="overflow: hidden; position: relative;"><desc>Created with Raphaël 2.1.2</desc><defs><path stroke-linecap="round" d="M5,0 0,2.5 5,5z" id="raphael-marker-block-obj361"></path><marker id="raphael-marker-endblock33-obj362" markerHeight="3" markerWidth="3" orient="auto" refX="1.5" refY="1.5"><use ns1:href="#raphael-marker-block-obj361" transform="rotate(180 1.5 1.5) scale(0.6,0.6)" stroke-width="1.6667" fill="black" stroke="none"></use></marker></defs><rect x="0" y="0" width="110" height="36.78125" r="20" rx="20" ry="20" fill="#ffffff" stroke="#000000" stroke-width="2" class="flowchart" id="st" transform="matrix(1,0,0,1,222.8672,122.543)" style=""></rect><text x="10" y="18.390625" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="stt" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,222.8672,122.543)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.203125">创建工作目录</tspan></text><path fill="#ffffff" stroke="#000000" d="M59.70703125,29.853515625L0,59.70703125L119.4140625,119.4140625L238.828125,59.70703125L119.4140625,0L0,59.70703125" stroke-width="2" id="c1" class="flowchart" transform="matrix(1,0,0,1,158.4531,290.5508)" style=""></path><text x="64.70703125" y="59.70703125" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="c1t" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,158.4531,290.5508)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="-3.80078125">下载主工程代码</tspan><tspan dy="18" x="64.70703125">                (cloneMainProject)</tspan></text><path fill="#ffffff" stroke="#000000" d="M42.83203125,21.416015625L0,42.83203125L85.6640625,85.6640625L171.328125,42.83203125L85.6640625,0L0,42.83203125" stroke-width="2" id="c2" class="flowchart" transform="matrix(1,0,0,1,192.2031,558.0664)" style=""></path><text x="47.83203125" y="42.83203125" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="c2t" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,192.2031,558.0664)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="-3.80078125">下载源</tspan><tspan dy="18" x="47.83203125">                (cloneSpec)</tspan></text><path fill="#ffffff" stroke="#000000" d="M101.91796875,50.958984375L0,101.91796875L203.8359375,203.8359375L407.671875,101.91796875L203.8359375,0L0,101.91796875" stroke-width="2" id="c3" class="flowchart" transform="matrix(1,0,0,1,74.0313,732.7461)" style=""></path><text x="106.91796875" y="101.91796875" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="c3t" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,74.0313,732.7461)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="-3.80078125">修改主工程的版本号</tspan><tspan dy="18" x="106.91796875">                (modifyMainProjectBundleIdentifier)</tspan></text><path fill="#ffffff" stroke="#000000" d="M78.75,39.375L0,78.75L157.5,157.5L315,78.75L157.5,0L0,78.75" stroke-width="2" id="c4" class="flowchart" transform="matrix(1,0,0,1,120.3672,1048.7656)" style=""></path><text x="83.75" y="78.75" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="c4t" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,120.3672,1048.7656)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="-3.796875">执行创建子模块的分支操作</tspan><tspan dy="18" x="83.75">                (allModuleProjectDo)</tspan></text><path fill="#ffffff" stroke="#000000" d="M88.775390625,44.3876953125L0,88.775390625L177.55078125,177.55078125L355.1015625,88.775390625L177.55078125,0L0,88.775390625" stroke-width="2" id="c5" class="flowchart" transform="matrix(1,0,0,1,100.3164,1308.4238)" style=""></path><text x="93.775390625" y="88.775390625" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="c5t" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,100.3164,1308.4238)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="-3.802734375">验证新分支能否编译成功</tspan><tspan dy="18" x="93.775390625">                (verifyMainProjectNewBranch)</tspan></text><path fill="#ffffff" stroke="#000000" d="M94.7109375,47.35546875L0,94.7109375L189.421875,189.421875L378.84375,94.7109375L189.421875,0L0,94.7109375" stroke-width="2" id="c6" class="flowchart" transform="matrix(1,0,0,1,88.4453,1582.1973)" style=""></path><text x="99.7109375" y="94.7109375" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="c6t" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,88.4453,1582.1973)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="-3.796875">提交新的分支</tspan><tspan dy="18" x="99.7109375">                (mainProjectCommitNewBranch)</tspan></text><path fill="#ffffff" stroke="#000000" d="M71.58984375,35.794921875L0,71.58984375L143.1796875,143.1796875L286.359375,71.58984375L143.1796875,0L0,71.58984375" stroke-width="2" id="c7" class="flowchart" transform="matrix(1,0,0,1,134.6875,1890.9629)" style=""></path><text x="76.58984375" y="71.58984375" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="c7t" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,134.6875,1890.9629)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="-3.80078125">源同步</tspan><tspan dy="18" x="76.58984375">                (mainProjectSpecSync)</tspan></text><path fill="#ffffff" stroke="#000000" d="M104.109375,52.0546875L0,104.109375L208.21875,208.21875L416.4375,104.109375L208.21875,0L0,104.109375" stroke-width="2" id="c8" class="flowchart" transform="matrix(1,0,0,1,69.6484,2120.9668)" style=""></path><text x="109.109375" y="104.109375" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="c8t" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,69.6484,2120.9668)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="-3.796875">切换到需要打Tag分支</tspan><tspan dy="18" x="109.109375">                (mainProjectChangePodfileNewTag)</tspan></text><path fill="#ffffff" stroke="#000000" d="M86.578125,43.2890625L0,86.578125L173.15625,173.15625L346.3125,86.578125L173.15625,0L0,86.578125" stroke-width="2" id="c9" class="flowchart" transform="matrix(1,0,0,1,104.7109,2433.541)" style=""></path><text x="91.578125" y="86.578125" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="c9t" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,104.7109,2433.541)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="-3.796875">提交工程代码</tspan><tspan dy="18" x="91.578125">                (mainProjectCommitNewTag)</tspan></text><path fill="#ffffff" stroke="#000000" d="M136.93359375,68.466796875L0,136.93359375L273.8671875,273.8671875L547.734375,136.93359375L273.8671875,0L0,136.93359375" stroke-width="2" id="c10" class="flowchart" transform="matrix(1,0,0,1,4,2660.6973)" style=""></path><text x="141.93359375" y="136.93359375" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="c10t" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,4,2660.6973)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="-3.80078125">打上Tag,同时删除旧的分支</tspan><tspan dy="18" x="141.93359375">                (mainProjectCreateNewTagAndDeleteOldBranch)</tspan></text><rect x="0" y="0" width="80" height="36.78125" r="20" rx="20" ry="20" fill="#ffffff" stroke="#000000" stroke-width="2" class="flowchart" id="e" transform="matrix(1,0,0,1,237.8672,3107.1074)" style=""></rect><text x="10" y="18.390625" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" id="et" class="flowchartt" font-size="15px" transform="matrix(1,0,0,1,237.8672,3107.1074)" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.203125">结束流程</tspan><tspan dy="18" x="10"></tspan></text><path fill="none" stroke="#000000" d="M277.8671875,159.32421875C277.8671875,159.32421875,277.8671875,267.97194504598156,277.8671875,287.5476340123962" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><path fill="none" stroke="#000000" d="M277.8671875,409.96484375C277.8671875,409.96484375,277.8671875,534.1021393928677,277.8671875,555.0745535981569" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="282.8671875" y="419.96484375" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.19921875">yes</tspan></text><path fill="none" stroke="#000000" d="M397.28125,350.2578125C397.28125,350.2578125,422.28125,350.2578125,422.28125,350.2578125C422.28125,350.2578125,422.28125,3082.107421875,422.28125,3082.107421875C422.28125,3082.107421875,277.8671875,3082.107421875,277.8671875,3082.107421875C277.8671875,3082.107421875,277.8671875,3097.48086643219,277.8671875,3104.1166696492583" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="402.28125" y="340.2578125" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.203125">no</tspan></text><path fill="none" stroke="#000000" d="M277.8671875,643.73046875C277.8671875,643.73046875,277.8671875,714.2227255785838,277.8671875,729.7461669016752" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="282.8671875" y="653.73046875" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.19921875">yes</tspan></text><path fill="none" stroke="#000000" d="M363.53125,600.8984375C363.53125,600.8984375,388.53125,600.8984375,388.53125,600.8984375C388.53125,600.8984375,388.53125,3082.107421875,388.53125,3082.107421875C388.53125,3082.107421875,277.8671875,3082.107421875,277.8671875,3082.107421875C277.8671875,3082.107421875,277.8671875,3097.48086643219,277.8671875,3104.1166696492583" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="368.53125" y="590.8984375" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.203125">no</tspan></text><path fill="none" stroke="#000000" d="M277.8671875,936.58203125C277.8671875,936.58203125,277.8671875,1027.9246329665184,277.8671875,1045.764984864276" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="282.8671875" y="946.58203125" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.19921875">yes</tspan></text><path fill="none" stroke="#000000" d="M481.703125,834.6640625C481.703125,834.6640625,506.703125,834.6640625,506.703125,834.6640625C506.703125,834.6640625,506.703125,3082.107421875,506.703125,3082.107421875C506.703125,3082.107421875,277.8671875,3082.107421875,277.8671875,3082.107421875C277.8671875,3082.107421875,277.8671875,3097.48086643219,277.8671875,3104.1166696492583" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="486.703125" y="824.6640625" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.203125">no</tspan></text><path fill="none" stroke="#000000" d="M277.8671875,1206.265625C277.8671875,1206.265625,277.8671875,1288.547533215955,277.8671875,1305.421752674305" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="282.8671875" y="1216.265625" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.203125">yes</tspan></text><path fill="none" stroke="#000000" d="M435.3671875,1127.515625C435.3671875,1127.515625,460.3671875,1127.515625,460.3671875,1127.515625C460.3671875,1127.515625,460.3671875,3082.107421875,460.3671875,3082.107421875C460.3671875,3082.107421875,277.8671875,3082.107421875,277.8671875,3082.107421875C277.8671875,3082.107421875,277.8671875,3097.48086643219,277.8671875,3104.1166696492583" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="440.3671875" y="1117.515625" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.203125">no</tspan></text><path fill="none" stroke="#000000" d="M277.8671875,1485.974609375C277.8671875,1485.974609375,277.8671875,1562.928624248365,277.8671875,1579.1986869632901" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="282.8671875" y="1495.974609375" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.201171875">yes</tspan></text><path fill="none" stroke="#000000" d="M455.41796875,1397.19921875C455.41796875,1397.19921875,456.3671875,1397.19921875,456.3671875,1397.19921875C456.3671875,1397.19921875,460.3671875,1389.19921875,464.3671875,1397.19921875C464.3671875,1397.19921875,480.41796875,1397.19921875,480.41796875,1397.19921875C480.41796875,1397.19921875,480.41796875,3082.107421875,480.41796875,3082.107421875C480.41796875,3082.107421875,277.8671875,3082.107421875,277.8671875,3082.107421875C277.8671875,3082.107421875,277.8671875,3097.48086643219,277.8671875,3104.1166696492583" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="460.41796875" y="1387.19921875" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.19921875">no</tspan></text><path fill="none" stroke="#000000" d="M277.8671875,1771.619140625C277.8671875,1771.619140625,277.8671875,1869.423730224371,277.8671875,1887.9531153633143" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="282.8671875" y="1781.619140625" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.197265625">yes</tspan></text><path fill="none" stroke="#000000" d="M467.2890625,1676.908203125C467.2890625,1676.908203125,476.41796875,1676.908203125,476.41796875,1676.908203125C476.41796875,1676.908203125,480.41796875,1668.908203125,484.41796875,1676.908203125C484.41796875,1676.908203125,492.2890625,1676.908203125,492.2890625,1676.908203125C492.2890625,1676.908203125,492.2890625,3082.107421875,492.2890625,3082.107421875C492.2890625,3082.107421875,277.8671875,3082.107421875,277.8671875,3082.107421875C277.8671875,3082.107421875,277.8671875,3097.48086643219,277.8671875,3104.1166696492583" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="472.2890625" y="1666.908203125" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.197265625">no</tspan></text><path fill="none" stroke="#000000" d="M277.8671875,2034.142578125C277.8671875,2034.142578125,277.8671875,2102.673267320497,277.8671875,2117.9655158372652" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="282.8671875" y="2044.142578125" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.197265625">yes</tspan></text><path fill="none" stroke="#000000" d="M421.046875,1962.552734375C421.046875,1962.552734375,418.28125,1962.552734375,418.28125,1962.552734375C418.28125,1962.552734375,422.28125,1954.552734375,426.28125,1962.552734375C426.28125,1962.552734375,446.046875,1962.552734375,446.046875,1962.552734375C446.046875,1962.552734375,446.046875,3082.107421875,446.046875,3082.107421875C446.046875,3082.107421875,277.8671875,3082.107421875,277.8671875,3082.107421875C277.8671875,3082.107421875,277.8671875,3097.48086643219,277.8671875,3104.1166696492583" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="426.046875" y="1952.552734375" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.201171875">no</tspan></text><path fill="none" stroke="#000000" d="M277.8671875,2329.185546875C277.8671875,2329.185546875,277.8671875,2413.4660175207537,277.8671875,2430.544335547115" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="282.8671875" y="2339.185546875" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.201171875">yes</tspan></text><path fill="none" stroke="#000000" d="M486.0859375,2225.076171875C486.0859375,2225.076171875,488.2890625,2225.076171875,488.2890625,2225.076171875C488.2890625,2225.076171875,492.2890625,2217.076171875,496.2890625,2225.076171875C496.2890625,2225.076171875,502.703125,2225.076171875,502.703125,2225.076171875C502.703125,2225.076171875,506.703125,2217.076171875,510.703125,2225.076171875C510.703125,2225.076171875,511.0859375,2225.076171875,511.0859375,2225.076171875C511.0859375,2225.076171875,511.0859375,3082.107421875,511.0859375,3082.107421875C511.0859375,3082.107421875,277.8671875,3082.107421875,277.8671875,3082.107421875C277.8671875,3082.107421875,277.8671875,3097.48086643219,277.8671875,3104.1166696492583" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="491.0859375" y="2215.076171875" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.201171875">no</tspan></text><path fill="none" stroke="#000000" d="M277.8671875,2606.697265625C277.8671875,2606.697265625,277.8671875,2646.3513655662537,277.8671875,2657.6977047096007" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="282.8671875" y="2616.697265625" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.197265625">yes</tspan></text><path fill="none" stroke="#000000" d="M451.0234375,2520.119140625C451.0234375,2520.119140625,456.3671875,2520.119140625,456.3671875,2520.119140625C456.3671875,2520.119140625,460.3671875,2512.119140625,464.3671875,2520.119140625C464.3671875,2520.119140625,476.0234375,2520.119140625,476.0234375,2520.119140625C476.0234375,2520.119140625,476.0234375,3082.107421875,476.0234375,3082.107421875C476.0234375,3082.107421875,277.8671875,3082.107421875,277.8671875,3082.107421875C277.8671875,3082.107421875,277.8671875,3097.48086643219,277.8671875,3104.1166696492583" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="456.0234375" y="2510.119140625" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.197265625">no</tspan></text><path fill="none" stroke="#000000" d="M277.8671875,2934.564453125C277.8671875,2934.564453125,277.8671875,3081.200701713562,277.8671875,3104.112615555525" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj362)" style=""></path><text x="282.8671875" y="2944.564453125" text-anchor="start" font="10px &quot;Arial&quot;" stroke="none" fill="#000000" font-size="15px" stroke-width="1" style="text-anchor: start; font-style: normal; font-variant-caps: normal; font-weight: normal; font-size: 15px; line-height: normal; font-family: Arial;"><tspan dy="5.197265625">yes</tspan></text></svg>



