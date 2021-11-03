---
title: "My First Post"
date: 2021-10-20T16:42:12+08:00
draft: true
tags: [呼叫中心,通话,工具]
categories: [通话]

---


# sngrep部署文档

### 安装包为：sngrep-1.4.7

### 操作步骤：

建议将安装位置放置在*数据盘*路径

```javascript
yum install -y ncurses-devel make libpcap-devel pcre-devel openssl-devel git gcc autoconf automake
tar -zxvf  sngrep-1.4.7.tar.gz
cd sngrep-1.4.7
./bootstrap.sh
./configure
make &&make install
一般报错多为缺少依赖，自行百度查找依赖。yum进行安装就可以
sngrep -c  ##进入抓包界面
```

```javascript
你可以将以下标志传递给. /configure 以启用某些功能
配置标志功能
–with-openssl 添加OpenSSL支持来解析捕获的消息( 请求。 libssl )
–with-gnutls 添加GnuTLS支持来解析捕获的消息( 请求。 gnutls )
–with-pcre 在正则表达式字段中添加Perl兼容的正规表达式 支持
–enable-unicode 添加 Ncurses/unicode支持( 要求。 libncursesw5 )
–enable-ipv6 启用IPv6数据包捕获支持。
–enable-eep 启用EEP数据包发送/接收支持。
```

![image-20210112145042181](/images/sngrep01.png)



###  sngrep基本参数
```javascript
-h --help：显示帮助信息
-V --version：显示版本信息
-d --device：指定抓包的网卡
-I --input：从pacp文件中解析sip包
-O --output：输出捕获的包到pacp文件中
-c --calls：仅显示邀请消息
-r --rtp：捕获RTP数据包有效载荷捕获rtp包
-l --limit：限制捕获对话的数量
-i --icase：使大小写不敏感
-v --invert：颠倒（不太明白）
-N --no-interface：不显示sngrep界面，仅捕获
-q --quiet：不要在无界面模式下打印捕获的对话框
-D --dump-config：打印活动的配置设置并退出
-f --config：从文件中读取配置
-R --rotate：达到捕获限制时轮换呼叫。
-H --eep-send：荷马sipcapture网址（udp：XXXX：XXXX）
-L --eep-listen：监听封装的数据包（udp：XXXX：XXXX）
-k --keyfile：RSA私钥文件解密捕获的数据包

抓取INVITE请求的包:
sngrep ^INVITE

抓取REGISTER请求的包:
sngrep ^REGISTER

抓取OPTIONS请求的包:
sngrep ^OPTIONS

捕获端口5060上的所有SIP数据包:
sngrep port 5060

使用sngrep查看来自pcap文件的SIP数据包，并使用过滤器:
sngrep -I file.pcap host 192.168.1.1 and port 5060

或实时捕获，将数据包保存到新文件:
sngrep -d eth0 -O save.pcap port 5060 and udp
```

###  快捷键
```javascript
• 箭头键：在列表中移动，除上下箭头还可以使用j，k来移动光标
• 输入：显示当前或所选对话框的消息流
• 答：自动滚动到新电话，自动滚动到新的电话
• F2或S：将所选/所有对话框保存到PCAP文件，保存对话框到pacp文件
• F3或/或TAB：输入显示过滤器。该过滤器将应用于列表中的文本行，进入搜索
• F4或x：显示当前选择的对话框和其相关的一个。回到第一个SIP消息上
• F5：清除通话清单，清空通话清单
• F6或r：以原始文本显示所选对话框的消息，显示原始的sip消息
• F7或f：显示高级过滤器对话框显示高级过滤弹窗
• F9或l：如果启用，则打开/关闭地址解析
• F10或t：选择显示的列，显示或隐藏侧边sip消息栏
呼叫列表页面还能够显示两个弹窗，按f可以显示高级过滤配置
```
