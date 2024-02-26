
https://www.youtube.com/watch?v=bQ2EtLnN6KQ&list=WL&index=25
# 1 setup process (via AWS Console)


1 新建一个OU
![](image/Pasted%20image%2020240221161054.png)

![](image/Pasted%20image%2020240221161330.png)


2 选中一个 OU, 新建新的 aws account 

![](image/Pasted%20image%2020240221161439.png)


![](image/Pasted%20image%2020240221161412.png)


3 move a aws acount into a ou 

![](image/Pasted%20image%2020240221161703.png)


## 1.1 reset password of new aws account

![](image/Pasted%20image%2020240221161902.png)


得到 email 去 reset password
![](image/Pasted%20image%2020240221161958.png)




# 2 Setup (via Terraform)

## 2.1 创造一个 organization

1: 用 AWS console: grant the AWS Organizations Permissions in root account 

这样 root account 就可以去创造一个 origanization 了

At minimum and possibly more you will need the following IAM permissions. Remember this needs to be in your management/root account.

```javascript
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ManageOrganizations",
      "Effect": "Allow",
      "Action": [
        "organizations:CreateOrganization",
        "organizations:EnableAWSServiceAccess",
        "organizations:DisableAWSServiceAccess",
        "organizations:DescribeOrganization",
        "organizations:ListRoots",
        "organizations:ListAccounts",
        "organizations:ListAWSServiceAccessForOrganization"
      ],
      "Resource": "*"
    }
  ]
}
```


2 用 terraform Code: 
authorize the AWS service principal which you want to enable integration with your organization

```javascript
# Provides a resource to create an AWS organization.
resource "aws_organizations_organization" "this" {

  # List of AWS service principal names for which 
  # you want to enable integration with your organization

  aws_service_access_principals = [
    "cloudtrail.amazonaws.com",
    "config.amazonaws.com",
    "sso.amazonaws.com"
  ]

  feature_set = "ALL"
}
```


3 也可以 Import an existing AWS Organizations

If you have already created/enabled AWS Organizations with the AWS CLI or through the console you do have the option to import and manage with Terraform going forward.

```sh
# If you using Terraform only
terraform import aws_organizations_organization.my_org o-1234567

# If you use Terragrunt
terragrunt import aws_organizations_organization.my_org o-1234567
```

## 2.2 Create new organizational unit and new AWS account

The Terraform code below will create a new member aka AWS account in the organization. But in order to achieve well-architected structure you must place your accounts in OUs. This way the Service Control Policies (SCPs) will be applied to the account immediately. Remember to follow a naming convention for your account and OU names.


```javascript
resource "aws_organizations_organizational_unit" "workload" {
  name      = "workload"
  parent_id = aws_organizations_organization.this.roots[0].id
}


resource "aws_organizations_organizational_unit" "dev" {
  name      = "dev"
  parent_id = aws_organizations_organizational_unit.workload.id

  depends_on = [
    aws_organizations_organizational_unit.workload
  ]
}

resource "aws_organizations_account" "dev" {
  # A friendly name for the member account
  name  = "example-dev"
  email = "validemail@domain.com"

  # Enables IAM users to access account billing information 
  # if they have the required permissions
  iam_user_access_to_billing = "ALLOW"

  tags = {
    Name  = "engineer-dev"
    Owner = "Waleed"
    Role  = "development"
  }

  parent_id = aws_organizations_organizational_unit.dev.id
}
```

After applying this Terraform code you will see a new account member in the AWS Organizations. 
Sadly, you will not get a set of credentials automatically. If your organization has enabled AWS SSO then you can use those credentials to switch roles to the new account after you add the account for your users/groups.

### 2.2.1 Getting the new AWS Accounts password

Since this account was created from AWS Organizations you have to navigate to the AWS console to sign in as the root account. Then enter your new accounts email address and then select **Forgot Password**. Then enter the captcha code and you will receive an email on setting a new password for your new AWS account. Ensure to [setup MFA](https://cloudly.engineer/2019/aws-cloud-account-initial-configuration/aws/) after logging in to secure your account login.



# 3 如何添加一个新的 aws account 以及赋给他权利

https://www.eoda.de/en/wissen/blog/automating-aws-organizations-terraform/

https://github.com/eodaGmbH/aws-organizations-with-terraform

We will start with a very basic structure which uses a module. The module is located in the directory _modules/aws-account_.

The directory structure looks like this:

```
.
├── config.tf
├── main.tf
└── modules
    └── aws-account
        └── variables.tf
```


## 3.1 Step-1 Creating an  new AWS sub Account 


create a new account. Attach it with a role. 这个 role 不需要之前就存在. 
这个 new account 不是 root acoount 
这个 new account 不需要属于 任何的 aws organizaition

We start by adding a new account to our organization. Navigate to [My Organizations](https://console.aws.amazon.com/organizations) and create an account with a globally unique mail address. 
In the field “IAM role name” you can enter “owner” – the default name is quite long, but you may enter anything you like.

--- 


Let’s do something useful and create an AWS Organizations account. We will use the aws_organizations_account resource. The account name and email address are defined by the variables we created in the last step.

```sh
resource "aws_organizations_account" "account" {
  # Account name for the "My organizations" overview
  # This is not the account alias which can be used
  # when switching roles
  name = var.account_name
  # Globally unique email address.
  # Each email account can only have one AWS account
  email = var.email_address
  # A role will be automatically created which has
  # full admin rights. Let's call it "owner"
  role_name = "owner"
  # Tags are not required, but might be helpful for others
  # in the future who read the assigned tags.
  tags = {
    "terraform-managed" : true
  }
}
```


## 3.2 Step-2 Manage the sub account and create an account alias

从 root aws account 切换到 这个 new aws account,    然后 在这个 新的 aws account 中 的界面 上, 给这个 新的 aws account 设置一个 account alias 

When you are logged in to the root AWS account, you can click on [Switch Roles](https://signin.aws.amazon.com/switchrole) in the main menu into the new account. You need to enter the Account ID which you can see on the [My Organizations](https://console.aws.amazon.com/organizations) page. 
“Role” ist the name of the role we entered when creating the account (“owner”, if you followed the instructions in step 1).

You may create an [Account Alias](https://docs.aws.amazon.com/de_de/IAM/latest/UserGuide/console_account-alias.html#AboutAccountAlias), so neither you or your colleagues need to remember the ID. Switch to the account and navigate to the [IAM Dashboard](https://console.aws.amazon.com/iam) to do so.

---


Step 1 executed commands in the root AWS account. However, we want to be able to manage both, the root and the sub account. To achieve this, we will add a second Terraform provider inside of our module. 

By using the assume_role parameter we can tell Terraform to switch to the sub account (described by the role_arn ) before executing actions when using this provider. 
The provider can be used by adding the provider parameter to any AWS Terraform resource.

We will use this provider to create an IAM account alias. Now it is possible to switch to the account by using the account name instead of the numeric AWS ID.

modules/aws-account/main.tf
``` javascript
  tags = {
    "terraform-managed" : true
  }
  
}

resource "aws_iam_account_alias" "alias" {
  provider      = aws.sub-account
  account_alias = var.account_name
}
}

```

modules/aws-account/providers.tf
```js
provider "aws" {
  region = "eu-central-1"
  alias  = "sub-account"

  assume_role {
    # Switch into the account we created in this module
    role_arn = "arn:aws:iam::${aws_organizations_account.account.id}:role/${aws_organizations_account.account.role_name}"
  }
}

```

## 3.3 Step 3 – Adding roles and permissions

role "owner" 在 生成 这个 aws acount 的时候就自动分配给他了, 这个 role “owner” has full administrative access. 

其他的新的 role 会放置在 new  aws account 内.  不是 放置在 aws root acount 内 
新的role 中 会被通过 分配给他 policy , 来获取 permission 

因为 我们之后 会经常需要 切换root account 到 这个 new aws account, 并且是以 新的 role 的身份切换到  这个 new aws account 中.  
如何实现:  _assume_role_policy_ is required to allow the root AWS account to use those roles.

--- 


The permission management in the sub accounts is done via roles. The role “owner” was automatically added when we created the account. It has full administrative access. 
New roles can be created at [IAM/Roles](https://console.aws.amazon.com/iam/home?#/roles). When asked about the “Type of trusted entity”, choose “Another AWS account” and enter the ID of the root account. This will allow us to assume the role from the root account. By using policies, we can set the desired permissions for the role.

For example, we create a **reader** role which gets the **ReadOnlyAccess** AWS policy assigned, and a **dev** role, which gets the policies **ReadOnlyAccess**, **AmazonEC2FullAccess** and **AmazonS3FullAccess**.

When everything was set up correctly, you can now use an administator IAM account in the root AWS account to switch to the created roles.

--- 

Using the resources [aws_iam_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role) and [aws_iam_role_policy_attachment](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment), we create roles with different permissions. _assume_role_policy_ is required to allow the root AWS account to use those roles. This was previously configured with the AWS UI, now we are using a JSON document, preprocessed by [templatefile](https://www.terraform.io/docs/language/functions/templatefile.html) so we can insert the ID of the AWS root account.


modules/aws-account/main.tf
```js
data "aws_caller_identity" "root" {}

resource "aws_organizations_account" "account" {
  # Account name for the "My organizations" overview
  # This is not the account alias which can be used
```


modules/aws-account/policy_assume_role.json.tpl

```js
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::${root_account_id}:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {}
    }
  ]
}
```


modules/aws-account/roles.tf
```js

locals {
  // The policy is the same for all roles, so we read it once to stay DRY
  // It allows the root account to switch to these roles
  policy_assume_role = templatefile("${path.module}/policy_assume_role.json.tpl", {
    root_account_id = data.aws_caller_identity.root.account_id
  })
}

resource "aws_iam_role" "reader" {
  provider           = aws.sub-account
  name               = "reader"
  assume_role_policy = local.policy_assume_role
  tags = {
    "terraform-managed" : true
  }
}

resource "aws_iam_role" "dev" {
  provider           = aws.sub-account
  name               = "dev"
  assume_role_policy = local.policy_assume_role
  tags = {
    "terraform-managed" : true
  }
}

// Give roles the desired permissions

resource "aws_iam_role_policy_attachment" "reader" {
  provider   = aws.sub-account
  role       = aws_iam_role.reader.name
  policy_arn = "arn:aws:iam::aws:policy/ReadOnlyAccess"
}

resource "aws_iam_role_policy_attachment" "dev" {
  for_each = toset([
    "arn:aws:iam::aws:policy/ReadOnlyAccess",
    "arn:aws:iam::aws:policy/AmazonS3FullAccess",
    "arn:aws:iam::aws:policy/AmazonEC2FullAccess",
  ])
  provider   = aws.sub-account
  role       = aws_iam_role.dev.name
  policy_arn = each.key
}
```

## 3.4 Step 4 – Switching roles

为了不仅 admin user in root account can switch to the roles in the sub account
想要 其他 user in root account also can switch to the roles in the sub account, 但是需要  define which users can access which role
比如说 user "dev" in root account  被并入到 user group "dev" in root account.  这个 user group 具有 policies "aws_iam_policy.roleswitch_reader.arn" and "aws_iam_policy.roleswitch_dev.arn"

就是 user dev in root acoount is allowed to aussime role "dev" and "reader" in new aws sub account 


## 3.5 concept 


To allow other users to assume roles in the sub accounts, we need to adjust some IAM settings in the root account.
In the **root AWS account*, go to [IAM/Policies](https://console.aws.amazon.com/iam/home?#/policies) and create the following policy:


```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::XXXXXXXXXXXX:role/ROLENAME"
        }
    ]
}
```

Replace _XXXXXXXXXXXX_ by the **account ID** of the sub account, and _ROLENAME_ by the name of the role in the sub account. Create a new [Group](https://console.aws.amazon.com/iamv2/home?#/groups) and assign the policy to the group. Now you will only need to add users to this group which should have the permission to assume that role.


---

https://signin.aws.amazon.com/switchrole

Switching roles enables you to manage resources across Amazon Web Services accounts using a single user. When you switch roles, you temporarily take on the permissions assigned to the new role. When you exit the role, you give up those permissions and get your original permissions back. Learn more. 

<mark> 一个 admin user 可以 assume a role in another aws aocunt  </mark>

Create role
Before you can switch roles, an administrator must create the role in the account you want to switch to, and then grant it permissions to perform the task you want.

Role access
Your administrator provides you with the account ID or alias and the role name to use.

Switch roles
Click your user name in the navigation bar, then select Switch Role. Enter the account and role information provided by your administrator. You immediately begin using the permissions associated with the new role. Exit the role to resume using your previous permissions.


## 3.6 implemantation 

though you as a admin in the root account can switch to the roles in the sub accounts, other users can’t. 
We will change that by adding groups and policies to the root AWS account. 

We assume that the IAM user accounts were already in your AWS account, and we address them by their username. To be able to define which users can access which role, we add new variables to our module. The definition of the required groups, policies, and assignment of users to the groups are put in `roleswitch.tf`.

![](image/Pasted%20image%2020240226130940.png)

main.tf
```js
  source        = "./modules/aws-account"
  account_name  = "my-account-name"
  account_name  = "my-account-name"
  email_address = "my-team@my-company.com"
  email_address = "my-team@my-company.com"
  owner_users   = ["some-iam-username"]
  dev_users     = ["developer-a", "developer-b"]
  reader_users  = ["junior-developer-x", "manager-y"]
}
}
```


modules/aws-account/policy_role_switch.json
```js
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::%s:role/%s"
        }
    ]
}


```

modules/aws-account/roleswitch.tf
```js
locals {
  policy_role_switch_template = file("${path.module}/policy_role_switch.json")
}

// create policies in root account

resource "aws_iam_policy" "roleswitch_reader" {
  name   = "${var.account_name}-reader"
  policy = format(local.policy_role_switch_template, aws_organizations_account.account.id, "reader")
}

resource "aws_iam_policy" "roleswitch_dev" {
  name   = "${var.account_name}-dev"
  policy = format(local.policy_role_switch_template, aws_organizations_account.account.id, "dev")
}

resource "aws_iam_policy" "roleswitch_owner" {
  name   = "${var.account_name}-owner"
  policy = format(local.policy_role_switch_template, aws_organizations_account.account.id, "owner")
}

// create groups in root account

resource "aws_iam_group" "reader" {
  name = "${var.account_name}-reader"
}

resource "aws_iam_group" "dev" {
  name = "${var.account_name}-dev"
}

resource "aws_iam_group" "owner" {
  name = "${var.account_name}-owner"
}

// attach policies to group in root account

resource "aws_iam_group_policy_attachment" "reader" {
  group      = aws_iam_group.reader.name
  policy_arn = aws_iam_policy.roleswitch_reader.arn
}

resource "aws_iam_group_policy_attachment" "dev" {
  // A user in the "dev" group should also be allowed to assume the lower privileged role "reader"
  for_each = {
    reader = aws_iam_policy.roleswitch_reader.arn
    dev    = aws_iam_policy.roleswitch_dev.arn
  }
  group      = aws_iam_group.dev.name
  policy_arn = each.key
}

resource "aws_iam_group_policy_attachment" "owner" {
  // A user in the "owner" group should be allowed to assume all other roles
  for_each = {
    reader = aws_iam_policy.roleswitch_reader.arn
    dev    = aws_iam_policy.roleswitch_dev.arn
    owner  = aws_iam_policy.roleswitch_owner.arn
  }
  group      = aws_iam_group.owner.name
  policy_arn = each.key
}

// add users to groups

resource "aws_iam_group_membership" "reader" {
  group = aws_iam_group.reader.name
  name  = "${var.account_name}-reader-membership"
  users = var.reader_users
}

resource "aws_iam_group_membership" "dev" {
  group = aws_iam_group.dev.name
  name  = "${var.account_name}-dev-membership"
  users = var.dev_users
}

resource "aws_iam_group_membership" "owner" {
  group = aws_iam_group.owner.name
  name  = "${var.account_name}-owner-membership"
  users = var.owner_users
}

```

modules/aws-account/variables.tf
```js
variable "account_name" {}
variable "account_name" {}
variable "email_address" {}
variable "email_address" {}
variable "reader_users" {
  type    = list(string)
  default = []
}
variable "dev_users" {
  type    = list(string)
  default = []
}
variable "owner_users" {
  type    = list(string)
  default = []
}

```





# 4 Terraform AWS Multi-Account Setup

https://cloudly.engineer/2021/terraform-aws-multi-account-setup/aws/

https://github.com/infrablocks/terraform-aws-organization

