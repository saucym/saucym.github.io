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
用下面命令可以生成一个`_book`文件夹，可以把它部署到服务器。
```bash
gitbook build
```

