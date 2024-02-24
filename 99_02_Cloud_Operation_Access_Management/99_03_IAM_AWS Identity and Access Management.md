

AWS account

- IAM user 
    - a user assumes a xx role
- IAM role: 
    - A policy is assigned to a role 
    - a reader role which gets the ReadOnlyAccess AWS policy assigned
    - a **dev** role, which gets the policies **ReadOnlyAccess**, **AmazonEC2FullAccess** and **AmazonS3FullAccess**.\
- IAM policy 
    - 




# 1 Implementation with Terraform


## 1.1 Creating an root AWS account of an AWS Organizations
这个 AWS Organizations 不需要之前就存在, 是随着 这个 root aws account 的创造, 这个 aws organization 一起就被创建了 

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


## 1.2 Manage the sub account and create an account alias

Step 1 executed commands in the root AWS account. However, we want to be able to manage both, the root and the sub account. To achieve this, we will add a second Terraform provider inside of our module. By using the assume_role parameter we can tell Terraform to switch to the sub account (described by the role_arn) before executing actions when using this provider. The provider can be used by adding the provider parameter to any AWS Terraform resource.

We will use this provider to create an IAM account alias. Now it is possible to switch to the account by using the account name instead of the numeric AWS ID.
