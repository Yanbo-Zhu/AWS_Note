
# 1 Foundation

- IAM user 
    - a user assumes a xx role
- IAM role: 
    - A policy is assigned to a role 
    - a reader role which gets the ReadOnlyAccess AWS policy assigned
    - a **dev** role, which gets the policies **ReadOnlyAccess**, **AmazonEC2FullAccess** and **AmazonS3FullAccess**.\
- IAM policy 


# 2 AWS Assume IAM role 的使用

https://yanbin.blog/how-to-assume-aws-iam-role/
https://repost.aws/knowledge-center/iam-assume-role-cli


AWS 要授权给他人访问指定资源有哪几种方式呢？

1. 创建一个 AWS 帐号让别人用，那是 AWS 干的事
2. 在自己帐号下创建一个用户，把 Access Key ID 和 Secret Access Key 告诉别人。可为该用户限定权限，但任何获得那两个 Key 的人都能使用该用户。
3. 创建一个 IAM Role, 并指定谁(帐号或 Role) 能以该 Role 的身份来访问。被 Assume 的 Role 可限定权限和会话有效期。

用 Assume Role 的方式具有更高的安全可控性，还不用维护 Access Key ID 和 Secret Access Key。比如在构建和部署时通常是有一个特定的 Account, 然后 Assume 到别的 IAM Role 去操作资源。

本文将详细介绍在帐号 A 创建一个 IAM Role(标注为 R) 并分配一些权限，然后允许另一个帐号 B 以 IAM Role - R 的身份来访问帐号 A 下的资源。IAM Role 将用 awscli 来创建，Assume Role 的过程用 awscli 和 boto3 Python 代码两种方式来演示。

### 2.1.1 帐号 A 下创建 IAM Role

首选要以帐号 A 登陆 AWS，不管哪种方式只要让 `aws s3 cli` 能正常工作就行。AWS 获得访问资格(Credentials) 的顺序可参照 Boto3 的文档 [Configuring Credentials](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html#configuring-credentials)。

比如说在 `~/.aws/credentials` 中配置了 profile A 和 B，分别对应帐号 A 和 B 的 profile 名称，在帐号 A 下创建 IAM Role 前用 `export AWS_DEFAULT_PROFILE=A` 指明当前为帐号 A

准备 role-trust-policy.json
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789022:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {}
    }
  ]
}
```


这里的 `123456789022` 是帐号 B 的 Account ID, 如果允许更多的帐号可以往上面的 `Principal` 中添加。

创建 test-assumed-role
> $ aws iam create-role --role-name test-assumed-role --assume-role-policy-document file://role-trust-policy.json

再给新建的 test-assumed-role 加上 S3 的只读权限
> $ aws iam attach-role-policy --role-name test-assumed-role --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

### 2.1.2 帐号 B Assume 帐号 A 的 role(awscli)

在帐号 A 下创建好 test-assumed-role 后，现在为 awscli 切换到帐号 B 下，比如 `export AWS_DEFAULT_PROFILE=B`。接着要用 `aws sts assume-role` 命令

> $ aws sts assume-role --role-arn arn:aws:iam::123456789011:role/test-assumed-role --role-session-name awscli-session

其中的 `123456789011` 是 帐号 A 的 Account ID，`sts`  是指 Security Token Service，

执行后输出如下

```sh
{
    "Credentials": {
        "AccessKeyId": "PNKDIESJGWAURFEWDLLT",
        "SecretAccessKey": "YcYdWHbtFZEDj/GetAyqRWdgvExxVpDJgC",
        "SessionToken": "IQoJb3JpZ2luX2VjEDoaCXVzLWVhc3QdDD1......QyAd6mnRruoJfLo1DUA==",
        "Expiration": "2021-11-10T02:36:46+00:00"
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "EXYARBWXQYCCAUONCJHG:awscli-session",
        "Arn": "arn:aws:sts::123456789011:assumed-role/test-assumed-role/awscli-session"
    }
}
```


这时候得到一组新的 AccessKeyId, SecretAccessKey 和 SessionToken，可以在 `~/.aws/credentials` 中配置一个新的 profile C, 然后 `export AWS_DEFAULT_PROFILE=C` 来使用。或都用 `export` 分别导出三个环境变量 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, 和 `AWS_SESSION_TOKEN`, 分别对应前面的三个值。

> $ export AWS_ACCESS_KEY_ID=<Credentials.AccessKeyId>  
> $ export AWS_SECRET_ACCESS_KEY=<Credentials.SecretAccessKey>  
> $ export AWS_SESSION_TOKEN=<Credentials.SessionToken>

替换上面的 <Credentials.*> 为 `aws sts assume-role ...` 返回的实际的相应字符串



一种更便捷一步到位的切换 role 的方式

```sh
export $(printf "AWS_ACCESS_KEY_ID=%s AWS_SECRET_ACCESS_KEY=%s AWS_SESSION_TOKEN=%s" \
            $(aws sts assume-role \
            --role-arn arn:aws:iam::123456789011:role/test-assumed-role \
            --role-session-name awscli-session \
            --query "Credentials.[AccessKeyId,SecretAccessKey,SessionToken]" \
            --output text))
```

---

这时候执行 `aws s3 ls` 获得的就是帐号 A 中的 S3 Bucket 列表

> $ aws s3 ls  
> 2021-09-10 12:09:35 bucket1-of-A  
> 2019-12-03 15:55:35 bucket2-of-A

想访问超出 `test-assumed-role` 之外的权限将被提示 Access Denied

> $ aws s3 cp Jenkinsfile s3://wfe-files  
> upload failed: ./test.txt to s3://bucket-of-A/test.txte An error occurred (AccessDenied) when calling the PutObject operation: Access Denied

任何时候都通过 `aws sts get-caller-identity` 来查看当前所使用的角色

> aws sts get-caller-identity  
> {  
>     "UserId": "EXYARBWXQYCCAUONCJHG:awscli-session",  
>     "Account": "123456789011",  
>     "Arn": "arn:aws:sts::123456789011:assumed-role/test-assumed-role/awscli-session"  
> }

### 2.1.3 帐号 B Assume 帐号 A 的 role(boto3)

awscli Assume 一个 IAM Role 的方式是重新获得一组 AccessKeyId, SecretAccessKey 和 SessionToken, 然后切换到被 Assumed 的 Role。用 Python 的 boto3 包实现是完全一样的。

```python
import boto3
 
aws_credentials_b = {
    'region_name': 'us-east-1',
    'aws_access_key_id':'PNKDIESJGWAURFEWDLLT',
    'aws_secret_access_key':'TdTMlDUSKecRadKeMlNIBEmIkRjmZOSvtnhgQDZc',
    'aws_session_token':'IQoJb3JpZ2luX2VjEDYabEbMG5J2lzlv......IEQisSAwzmnkv7LNf+'
}
 
 
sts=boto3.client('sts', **aws_credentials_b)
 
stsresponse = sts.assume_role(
    RoleArn="arn:aws:iam::123456789011:role/test-assumed-role", # under account A
    RoleSessionName='assumed'
)
 
aws_credentials_assumed_role = {
    'region_name':'us-east-1',
    'aws_access_key_id':stsresponse["Credentials"]["AccessKeyId"],
    'aws_secret_access_key':stsresponse["Credentials"]["SecretAccessKey"],
    'aws_session_token':stsresponse["Credentials"]["SessionToken"]
}
 
 
boto3.setup_default_session(**aws_credentials_assumed_role)
 
s3 = boto3.client('s3')
buckets_of_a = [bucket['Name'] for bucket in s3.list_buckets()['Buckets']]
```

帐号 B 登陆，调用 boto3 的 sts.assume_role() 函数切换到帐号 A 下的 IAM Role test-assumed-role，之后的操作就限定到 test-assumed-role 的约束中了。

当然，使用 Python 的话可以进一步封装，比如默认以帐号 B 登陆，然后执行一个函数 `switch_role(role_arn)` 后，后续的 boto3 client 就全部变成了 assumed role 的角色了

```python
import boto3
 
def switch_role(assume_role_arn):
    sts=boto3.client('sts')
    sts_res = sts.assume_role(RoleArn=assume_role_arn, RoleSessionName='new_session')
 
    new_credentials = {'aws' + re.sub('([A-Z]+)', r'_\1', key).lower(): value
                       for (key, value) in sts_res["Credentials"].items() if key != 'Expiration'}
 
    boto3.setup_default_session(**new_credentials)
    
switch_role('arn:aws:iam::123456789011:role/test-assumed-role')
 
s3 = boto3.client('s3')
buckets_of_a = [bucket['Name'] for bucket in s3.list_buckets()['Buckets']]
```

把 `sts_res['Credentials']` 转换为 session 要求的格式是简化，但是要注意以后 assume_role() 响应格式的变化有可能影响到程序的正常执行。


## 2.2 Lambda

![](image/Pasted%20image%2020240418175008.png)

lambda funtion 的 execution Role also need to be attached with assumen Role Policy so this function can aceess the resource from other aws account 


