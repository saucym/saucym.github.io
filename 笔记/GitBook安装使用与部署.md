# 安装方法
安装命令
```bash
brew cask install gitbook
```

安装遇到问题：`TypeError: cb.apply is not a function`.

原因: `node`版本不对，本地是`14.0.0`需要使用`12.16.3`版本.

解决方法：安装[nvm](https://juejin.im/post/6844903839204638734)管理本地的多`node`版本

# 使用方法
在当前目录用下面命令跑起来，可以预览，而且修改md文件后预览会实时变化。
```bash
gitbook serve
```
编译生成一个`_book`文件夹。
```bash
gitbook build
```

# 部署
下面的`远程服务器`都是指腾讯云`Debian`系统，当然其他云也是没问题的。

[安装`docker`](https://www.runoob.com/docker/ubuntu-docker-install.html)，然后部署`nginx`

在`远程服务器`上先随便启动一个容器，主要是用来生成默认的配置文件。
```bash
docker run -d -p 80:80  --name mynginx nginx
```
然后创建一个本地文件夹把默认配置`copy`一份，然后关闭这个容器并删除。
```bash
mkdir /web
cd /web
docker cp mynginx:/etc/nginx/conf.d .
docker stop mynginx
docker rm mynginx
```
可以使用`vim`编辑这个配置文件（这一步可以不做）。
```bash
vim conf.d/default.conf
```
可以看到有一个`/usr/share/nginx/html`，表示资源地址，不过这个资源地址是容器的修改了也没用，我们启动的时候把本地目录映射到这个地址就可以了，所以这里不需要修改，如果深入研究这个配置文件，可以做到很多事。

然后`本地电脑`把要部署的资源上传到`远程服务器`。
```bash
scp -r _book root@ip:/web/book
```
最后在`远程服务器`上启动容器并加上映射。这个`80:80`前面这个80是服务器的端口，后面这个80是容器的端口。映射`-v`可以加很多个，一般把`conf`里面需要用到的加上就可以了，我们这里把配置文件和资源路径映射一下就好了。
```bash
docker run -d -p 80:80 --name web -v /web/book:/usr/share/nginx/html -v /web/conf.d:/etc/nginx/conf.d nginx
```
这个时候你就可以直接在浏览器输入你的ip地址浏览这个网页了。

# `scp`提示权限被拒绝的解决办法
`Linux`的远程传输文件`scp`出现`Permission denied (publickey).lost connection`问题解决方法。

首先去`~/.ssh`目录下看是否已经有密匙。如果没有可以通过下面命令生成一份。`.pub`结尾的是公匙。
```bash
cd ~/.ssh
ssh-keygen -t rsa
```
将当前主机上的`id_rsa.pub`文件拷贝到远程`Linux`主机的`root`用户目录下的`.ssh`目录下，并且改名为`authorized_keys` 。若已经有该文件可以把内容追加在后面，要配对多个机器就在后面追加就行了。

这样本地密匙跟远程主机密匙就能对上了，后续`scp`命令就不需要带鉴权信息了。

# 问题?
1. 把多个`web`服务部署到同一个机器上，使用不同的路径访问。
目前尝试了编辑配置文件`conf.d/default.conf`，但貌似没效果，后续再深入研究一下。目前暂时启动了两个`docker`指定不同端口来解决这个问题，不过这样就只能这样访问`ip:81`，不优雅。
2. 如何把多个web服务部署到同一个服务器并使用不同的子域名访问。

[参考文章](https://www.jianshu.com/p/ea0643ad0497)

