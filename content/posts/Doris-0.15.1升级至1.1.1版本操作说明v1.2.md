---
title: "Doris 0.15.1升级至1.1.1版本操作说明"
date: 2022-11-02T10:34:02+08:00
draft: true
tags: [Doris,中间件,数据库]
categories: [数据库]
featuredImage: "/images/doris.jpg"
featuredImagePreview: "/images/doris.jpg"

---

# Doris 0.15.1升级至1.1.1版本操作说明

## 一、环境信息

| 名称       | ip            | 备注/其他 |
| ---------- | ------------- | --------- |
| Doris-de01 | 172.31.24.184 |           |
| Doris-de02 | 172.31.16.119 |           |
| Doris-de03 | 172.31.30.174 |           |
| Doris-de04 | 172.31.23.38  |           |
| Doris-de05 | 172.31.22.228 |           |
| Doris-de06 | 172.31.45.123 |           |
| Doris-de07 | 172.31.43.42  |           |
| Doris-fe01 | 172.31.21.232 | IsMaster  |
| Doris-fe02 | 172.31.17.196 |           |
| Doris-fe03 | 172.31.17.204 | OBSERVER  |
| Doris-fe04 | 172.31.11.169 |           |

## **二、具体操作步骤及说明**

### 前置工作

1. 关闭集群副本修复和均衡功能

   升级过程中会有节点重启，所以可能会触发不必要的集群均衡和副本修复逻辑。可以先通过以下命令关闭：

   优先查看下集群默认参数：

   ![image-20220825170322568](/Users/zhangdawei/Library/Application Support/typora-user-images/image-20220825170322568.png)

   ```shell
   # 关闭副本均衡逻辑。关闭后，不会再触发普通表副本的均衡操作。
   $ mysql-client > admin set frontend config("disable_balance" = "true");
   
   # 关闭 colocation 表的副本均衡逻辑。关闭后，不会再触发 colocation 表的副本重分布操作。
   $ mysql-client > admin set frontend config("disable_colocate_balance" = "true");
   
   # 关闭副本调度逻辑。关闭后，所有已产生的副本修复和均衡任务不会再被调度。
   $ mysql-client > admin set frontend config("disable_tablet_scheduler" = "true");
   ```

   **==当集群升级完毕后，在通过以上命令将对应配置设为原值即可。==**

   

2. **重要！！在升级之前需要备份元数据（整个目录都需要备份）！！**

   所有fe机器/data/doris-meta数据目录备份。

   ```shell
   tar -zcvf /data/doris-meta.tar.gz /data/doris-meta
   cp -R /data/doris-meta /data/doris-meta-0.15.1-date
   ```

   

3. 官网下载二进制文件，进行配置

   ```shell
   wget https://archive.apache.org/dist/doris/1.1/1.1.1-rc03/apache-doris-1.1.1-bin-x86.tar.gz 
   tar -zxvf  apache-doris-1.1.1-bin-x86.tar.gz
   mv apache-doris-1.1.1-bin-x86 /opt/doris-1.1.1 
   cd /opt/doris-1.1.1
   ```

   

### 测试 BE 升级正确性

1. 任意选择一个 BE 节点，部署最新的 doris_be 二进制文件。

   1.1、doris07节点测试,**==（具体路径已线上正式环境目录路径为准。）==**

   ```shell
   sh /opt/doris-be-0.15.1/bin/stop_be.sh 
   cd /opt/doris-be-0.15.1/ 
   mv lib lib-0.15.1-$date 
   cp -R /opt/doris-1.1.1/be/lib ./ 
   sh /opt/doris-be-0.15.1/bin/start_be.sh --daemon 
   #查看日志启动是否正常,具体路径已线上正式环境目录路径为准。
   ```

2. 重启 BE 节点，通过 BE 日志 be.INFO，查看是否启动成功。

3. 如果启动失败，可以先排查原因。如果错误不可恢复，可以直接通过 DROP BACKEND 删除该 BE、清理数据后，使用上一个版本的 doris_be 重新启动 BE。然后重新 ADD BACKEND。（**该方法会导致丢失一个数据副本，请务必确保3副本完整的情况下，执行这个操作！！！**）

   



### 测试 FE 元数据兼容性

1. **重要！!元数据兼容性异常很可能导致数据无法恢复！！**

2. 单独使用新版本部署一个测试用的 FE 进程（建议在自己本地的开发机，或者BE节点。如果在Follower或者Observer节点上，需要停止启动的进程,但是不建议在Follower或者Observer节点上测试）。

   2.1、还是在be07节点上copy一份fe节点的部署包、及/data/doris-meta 修改相关配置进行兼容性启动测试。

   2.2、be07节点测试fe升级兼容性==**（具体路径已线上正式环境目录路径为准。）**==

   ```shell
   sh /opt/doris-fe-0.15.1/bin/stop_be.sh 
   cd /opt/doris-fe-0.15.1/ 
   mv lib lib-0.15.1-date 
   cp -R /opt/doris-1.1.1/fe/lib ./ 
   vim /opt/doris-fe-0.15.1/conf/fe.conf
   cp /home/quicksuper/doris-meta.tar.gz /data/ && cd /data && tar -zxvf doris-meta.tar.gz
   vim /data/doris-meta/image/VERSION
   #操作完3-7步骤后最后在执行步骤8
   sh /opt/doris-fe-0.15.1/bin/start_be.sh --daemon 
   #查看日志启动是否正常,具体路径已线上正式环境目录路径为准。
   ```

3. 修改测试用的 FE 的配置文件 fe.conf，将所有端口设置为**与线上不同**。

4. 在 fe.conf 添加配置：cluster_id=123456

5. 在 fe.conf 添加配置：metadata_failure_recovery=true

6. 拷贝线上环境 Master FE 的元数据目录 doris-meta 到测试环境

7. 将拷贝到测试环境中的 doris-meta/image/VERSION 文件中的 cluster_id 修改为 123456（即与第3步中相同）

8. 在测试环境中，运行 sh bin/start_fe.sh 启动 FE

9. 通过 FE 日志 fe.log 观察是否启动成功。

10. 如果启动成功，运行 sh bin/stop_fe.sh 停止测试环境的 FE 进程。

11. **以上 2-6 步的目的是防止测试环境的FE启动后，错误连接到线上环境中。**



## 三、滚动升级

1. 确认新版本的文件部署完成后。逐台重启 FE 和 BE 实例即可。

2. 建议逐台重启 BE 后，再逐台重启 FE。因为通常 Doris 保证 FE 到 BE 的向后兼容性，即老版本的 FE 可以访问新版本的 BE。但可能不支持老版本的 BE 访问新版本的 FE。

3. 建议确认前一个实例启动成功后，再重启下一个实例。实例启动成功的标识，请参阅安装部署文档。

   https://doris.incubator.apache.org/zh-CN/docs/install/install-deploy/  常见问题--进程相关

## 关于版本回滚

因为数据库是一个有状态的服务，所以在大多数情况下，Doris 无法支持版本回滚（版本降级）。在某些情况下，可以支持 3 位或 4 位版本的回滚，但不会支持 2 位版本的回滚。 所以建议通过先升级部分节点并观察业务运行情况的方式（灰度升级）来降低升级风险。

**非法的回滚操作可能导致数据丢失和损坏。**



建议回滚时，保留最初备份文件及tar文件放置源数据丢失。

1、先回滚fe，所有机器全部执行完回滚操作在进行重启动作**==(文件名、文件路径根据线上环境进行修改)==**；因回滚了doris-meta，可能会有升级期间的数据丢失。

```shell
# 关闭副本均衡逻辑。关闭后，不会再触发普通表副本的均衡操作。
$ mysql-client > admin set frontend config("disable_balance" = "true");

# 关闭 colocation 表的副本均衡逻辑。关闭后，不会再触发 colocation 表的副本重分布操作。
$ mysql-client > admin set frontend config("disable_colocate_balance" = "true");

# 关闭副本调度逻辑。关闭后，所有已产生的副本修复和均衡任务不会再被调度。
$ mysql-client > admin set frontend config("disable_tablet_scheduler" = "true");

cd /opt/doris-fe-0.15.1
mv lib  lib-date-1.1.1
cp -R  lib-0.15.1-date  lib
cd /data
mv doris-meta doris-meta-1.1.1-date
cp -R doris-meta-0.15.1-date doris-meta
sh /opt/doris-fe-0.15.1/bin/stop_fe.sh
sh /opt/doris-fe-0.15.1/bin/start_fe.sh --daemon
```

2、检查进程启动及日志。

![l2p3OXWDqd](/Users/zhangdawei/Library/Application Support/LarkShell/sdk_storage/242809214bfd58425e3f89823c700afd/resources/images/l2p3OXWDqd.jpg)

3、回滚be节点，与fe节点操作方式一致，所有机器全部执行完回滚操作在进行重启动作**==(文件名、文件路径根据线上环境进行修改)==**

```shell
cd /opt/doris-be-0.15.1/
mv  lib ./lib-0825-1.1.1-date
cp -R lib-0.15.1-date lib
sh /opt/be-0.15.1/bin/stop_be.sh
sh /opt/be-0.15.1/bin/start_be.sh --daemon
```

![VnNCIDWKFE](/Users/zhangdawei/Library/Application Support/LarkShell/sdk_storage/242809214bfd58425e3f89823c700afd/resources/images/VnNCIDWKFE.jpg)

 回滚后doris集群状态正常，但服务连接报错不可用，根据官方建议不建议回滚。
