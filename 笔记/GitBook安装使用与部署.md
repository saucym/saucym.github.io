# 安装方法
安装命令
```bash
brew cask install gitbook
```

安装遇到问题：`TypeError: cb.apply is not a function`.

原因: `node`版本不对，本地是`14.0.0`需要使用`12.16.3`版本.

解决方法：安装[nvm](https://juejin.im/post/6844903839204638734)管理本地的多`node`版本

# 使用方法
在当前目录用下面命令跑起来，可以预览，而且修改md文件实时会变化。
```bash
gitbook serve
```
编译生成一个`_book`文件夹。
```bash
gitbook build
```
部署到服务器。
```bash
scp -r _book  root@ip:/web/book
```

# 部署遇到的问题
`Linux`的远程传输文件`scp`出现`Permission denied (publickey).lost connection`问题解决方法。

首先去`~/.ssh`目录下看是否已经有密匙。如果没有可以通过下面命令生成一份。`.pub`结尾的是公匙。
```bash
cd ~/.ssh
ssh-keygen -t rsa
```
将当前主机上的`id_rsa.pub`文件拷贝到远程`Linux`主机的`root`用户目录下的`.ssh`目录下，并且改名为`authorized_keys` 。若已经有该文件可以把内容追加在后面。

这样本地密匙跟远程主机密匙就能对上了，后续`scp`命令就不需要带鉴权信息了。

[参考文章](https://www.jianshu.com/p/ea0643ad0497)

