# saucym.github.io  这里记录一些日常笔记  沉淀一下



#一些命令
检验包签名是否正常(无返回表示正常): codesign --verify Payload/PhotoTool.app 
实现重签名: codesign -f -s 'iPhone Developer: Thomas Kollbach (7TPNXN7G6K)' Example.app
打包成ipa: zip -qr app-resigned.ipa Payload/
查看framework支持的架构: lipo -info /xxx.framework/xxx

检出代码并且自动更新子模块: git clone --recursive https://github.com/saucym/xxx.git

//下面是终端命令打包 可以删除目录里面的git和svn版本控制文件
find . -name .svn | xargs rm -rf
find . -name .DS_* | xargs rm -rf
find . -name .git | xargs rm -rf

#抓包方法
第一步：使用USB数据线将iOS设备连接到MAC上
第二步：获得iOS设备的UDID，可以使用iTools查看，也可以使用Xcode的Organizer工具查看
第三步：创建RVI接口
$ rvictl -s <UDID>
RVI虚拟接口的命令规则可为rvi0，rvi1，。。。,创建后可以使用以下命令查看是否创建成功
$ ifconfig rvi0
第四步：在mac上用抓包工具wireshark或tcpdump等工具抓包分析
$ sudo tcpdump -i rvi0 -n -vv
第五步：分析结束后，移除创建的RVI接口
$ rvictl -x <UDID>

//抓包命令
sudo tcpdump -s 0 -X -ien1 host  101.227.131.102
