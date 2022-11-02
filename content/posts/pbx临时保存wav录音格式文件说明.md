---
title: "Pbx临时保存wav录音格式文件说明"
date: 2021-11-03T14:34:02+08:00
draft: true
tags: [呼叫中心,通话]
categories: [通话]
featuredImage: "/images/pbx-wav-logo.jpg"
featuredImagePreview: "/images/pbx-wav-logo.jpg"

---

# pbx临时保存wav录音格式文件

## 用ass合成

1.在server.conf中[server]段内添加需要wav文件的账号：

```bash
thisAccountNeedWav=N00000000535,N00000001583
```

多个账号用，(英文逗号)隔开。

2.在server.conf中[server]段内添加wav保存的天数：

```bash
deleteWavs=3 
```

  wav文件比较大，不建议保存太久。

3.reload 一下ass配置文件：

(echo "Login -u 用户名 -p 密码";sleep 1;echo "reload" ;sleep 3)|telnet localhost 7877  

4.新加nginx配置文件：

```bash
		location /inOutTempOfWav {
            root   /var/spool/asterisk/;
            index  index.html index.htm;
            ##以下limit配置必须定义limit参数，否则不予以配置##
            limit_rate_after 10M;
            limit_rate  800k;
            limit_conn monitor_down 10;
            limit_req zone=monitor_down_1 burst=20 nodelay;
        }
```

5.reload nginx配置文件：

```bash
/opt/nginx/sbin/nginx -s reload
```

![image-20201125114849804](/images/pbx-wav-01.png)



配置完成后让前端运维或客户测试。



