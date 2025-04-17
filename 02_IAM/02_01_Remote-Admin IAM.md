
如果你想创建一个 Remote-Admin IAM 角色，用于远程管理 AWS 资源（支持跨账户访问）



# 1 账户A: role with name Remote-Admin 

## 1.1 创建 Remote-Admin role 的Policy

在 AWS 账户 A（资源账户）中 (mit ID 111111111111)，创建 IAM 角色 Remote-Admin，允许 AWS 账户 B（管理员账户 mit id 222222222222）中的用户访问。

📌 创建角色的信任策略（Trust Policy）： 将这个 policy attach to Remote-Admin role in account A 
"AWS": "arn:aws:iam::222222222222:root" → 允许 AWS 账户 B（222222222222） 假设该角色。
"sts:AssumeRole" → 允许目标账户 B 使用 aws sts assume-role 获取临时凭证。
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::222222222222:root" // 这是账户B    root 的作用是 **Allows all IAM users and roles** from account `222222222222` to assume this role. **Does not** restrict it to just the root user of the account.
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

In AWS IAM (Identity and Access Management), the "Principal": { "AWS": "arn:aws:iam::222222222222:root" } notation refers to the entire AWS account (with account ID 222222222222).

**What does `root` mean in AWS ARN?**
- `"arn:aws:iam::222222222222:root"` represents **all IAM users, roles, and federated users** within the AWS account **222222222222**.
- It **does not** refer to the **root user** of the AWS account (the user with full control). Instead, it applies to **all identities** within that AWS account.
- When used in an **IAM Trust Policy**, it means **any IAM user or role from the specified AWS account** can assume the role (if they have the right permissions).


---

How to Restrict Access to Specific Roles or Users?
If you want only a specific IAM role (not everyone in the account) to assume the role, replace root with a specific IAM role ARN:

Now only MyAdminRole in account 222222222222 can assume this role.
```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::222222222222:role/MyAdminRole"
  },
  "Action": "sts:AssumeRole"
}
```

---


保护 Remote-Admin 角色，启用 MFA
- **只有启用了 MFA 的用户** 才能假设此角色。
- 这可以防止攻击者在**凭证泄露**时，未经授权访问 AWS 资源。

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::222222222222:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
```


### 1.1.1 `arn:aws:iam::222222222222:root`






## 1.2 赋予 Remote-Admin 角色管理员权限 的policy 


给 Remote-Admin role in 账户a 附加 AdministratorAccess 权限：

IAM 权限策略（AdministratorAccess）：
- `"Action": "*"` → 允许执行 **所有 AWS 操作**（完全管理员权限）。
- `"Resource": "*"` → 适用于 **所有 AWS 资源**。
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```



如果不想给完全管理员权限，可以自定义策略。例如，只允许访问 EC2 和 S3：
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "s3:*"
      ],
      "Resource": "*"
    }
  ]
}
```



# 2 在 AWS 账户 B（管理员账户）中使用 Remote-Admin role of 账户A

在 AWS 账户 B（222222222222）中，用户可以使用 AWS CLI 假设该角色。


## 2.1 在账户B 中执行, 使用 AWS CLI 假设 Remote-Admin 角色  获取临时凭证

```
aws sts assume-role \
  --role-arn "arn:aws:iam::111111111111:role/Remote-Admin" \    这是账户A 
  --role-session-name "AdminSession"
```

- `--role-arn "arn:aws:iam::111111111111:role/Remote-Admin"` → 这是要假设的角色 ARN。
- `--role-session-name "AdminSession"` → 会话名称，用于区分不同的登录会话。


✅ 如果 MFA 受限，你可以使用 MFA 假设角色：
```
aws sts assume-role \
  --role-arn "arn:aws:iam::111111111111:role/Remote-Admin" \
  --role-session-name "AdminSession" \
  --serial-number "arn:aws:iam::222222222222:mfa/my-user" \
  --token-code 123456
```

--serial-number → MFA 设备的 ARN。
--token-code → MFA 设备生成的 6 位验证码。



返回响应 
```
{
  "Credentials": {
    "AccessKeyId": "ASIAEXAMPLEKEY",
    "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
    "SessionToken": "FwoGZXIvYXdzEJr//////////wEa...",
    "Expiration": "2025-04-01T12:34:56Z"
  }
}
```


## 2.2 设置环境变量，使用假设角色的临时凭证


```
export AWS_ACCESS_KEY_ID="ASIAEXAMPLEKEY"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_SESSION_TOKEN="FwoGZXIvYXdzEJr//////////wEa..."
```

现在，你可以运行 AWS 命令，例如：

aws s3 ls
aws ec2 describe-instances


## 2.3 自动化：配置 AWS CLI Profile


你可以将 Remote-Admin 角色 添加到 AWS CLI 配置文件，以便快速切换角色。


**修改 `~/.aws/config` 文件**

```
[profile remote-admin]
role_arn = arn:aws:iam::111111111111:role/Remote-Admin
source_profile = default
region = eu-central-1
```


然后，你可以使用以下命令切换到 Remote-Admin 角色：
```
aws s3 ls --profile remote-admin
```
