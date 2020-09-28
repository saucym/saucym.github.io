# 页面部署到服务器
`Linux`的远程传输文件`scp`出现`Permission denied (publickey).lost connection`问题解决方法。

首先去`~/.ssh`目录下看是否已经有密匙。如果没有可以通过下面命令生成一份。`.pub`结尾的是公匙。
```bash
cd ~/.ssh
ssh-keygen -t rsa
```
将当前主机上的`id_rsa.pub`文件拷贝到远程`Linux`主机的`root`用户目录下的`.ssh`目录下，并且改名为`authorized_keys` 。若已经有该文件可以把内容追加在后面。

这样本地密匙跟远程主机密匙就能对上了，后续`scp`命令就不需要带鉴权信息了。

[参考文章](https://www.jianshu.com/p/ea0643ad0497)
