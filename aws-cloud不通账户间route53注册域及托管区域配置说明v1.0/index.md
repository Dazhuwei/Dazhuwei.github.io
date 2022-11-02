# AWS-cloud不通账户间ROUTE53注册域及托管区域配置说明V1.0


# AWS-cloud不通账户间ROUTE53注册域及托管区域配置说明V1.0

### 前期说明：

​		A、B两个账户均为AWS-Cloud账户，目前域名xxx.com注册配置在A账户Route53，现需转移xxx.com域名至B账户Route53，因AWS-Cloud页面控制台无操作功能，需进行api相关操作进行域名迁移。

**注意**

​	域名无法在注册后的前 14 天内转移。



### 一、前期准备：

#### 1、A、B账户的iam新建子账户授权，下载保存对应AK/SK==（妥善保存）==

​	**==重要==**

您必须使用根账户，或者使用通过以下一种或多种方式被授予 IAM 权限的 IAM 用户登录：

- ​	为用户分配了 AdministratorAccess 托管策略。

- ​	为用户分配了 AmazonRoute53DomainsFullAccess 托管策略。

- ​	为用户分配了 AmazonRoute53FullAccess 托管策略。

- ​	为用户分配了 PowerUserAccess 托管策略。


用户有权执行以下所有操作：TransferDomains、DisableDomainTransferLock 和 RetrieveDomainAuthCode。

如果您没有使用根账户或者具有所需权限的 IAM 用户登录，我们将无法执行转移。此要求可防止未经授权的用户将域转移到其它 AWS 账户。



#### 2、配置一台云主机安装aws-cli客户端（因涉及海外及平台为aws，建议使用aws海外节点EC2）

```shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
./aws/install -i /usr/local/aws-cli -b /usr/local/bin
aws --version
//返回结果aws-cli/2.7.24 Python/3.8.8 Linux/4.14.133-113.105.amzn2.x86_64 botocore/2.4.5
```

已Linux操作系统为例，[其他操作系统安装参考链接](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/getting-started-install.html#getting-started-install-instructions)



#### 3、配置账户AK/SK认证

```shell
#根据操作步骤提前替换对应账户的AK/SK
aws configure
AWS Access Key ID [None]: ****************EX5V                                               
AWS Secret Access Key [None]: ****************MzVU
Default region name [None]: us-east-1
```



### 二、转移域名操作步骤

#### 1、在A账户AK/SK认证下的aws-cli下操作确定转移的主域名名称：


```shell
aws route53domains enable-domain-transfer-lock \
    --region us-east-1 \
    --domain-name xxx.com

//返回操作id，不重要
{
"OperationId": "f500bd1d-1848-45ff-a9a4-dcfb1e1be4a72"
 }
```


#### 2、A账户AK/SK认证下aws-cli下操作，确定转移到B账户下account-id

```shell
aws route53domains transfer-domain-to-another-aws-account \
--domain-name xxx.com \
--account-id 5********1   

{
    "OperationId": "32ae99ee-c3a20-45f3-8920-365647b7abc22",
    "Password": "1i[5**|&fDxtD1"  //注意密码记录，下一步需要确认使用
}
```



#### 3、B账户AK/SK认证下aws-cli下操作，确认域名迁移至B账户。

```shell
aws route53domains accept-domain-transfer-from-another-aws-account \
--domain-name xxx.com --password "1i[5**|&fDtD11" \
--region us-east-1 

{
    "OperationId": "d2xx4816bf6-6048-4de5-9bae-5f6f1fdb640a2"
}
```



#### 4、后续页面在Route53-----托管区域-----创建托管区域。

![image-20221012214918227](/Users/zhangdawei/Documents/Hugo/quickstart/static/images/route53-01.png)

### 后续：

旧账户Route53中的注册域、托管区域要截图保留好进行删除或者迁移解析记录。

新账户Route53中的注册域**名称服务器**，注意进行修改为现有账户托管区域中DNS解析的NS记录值




