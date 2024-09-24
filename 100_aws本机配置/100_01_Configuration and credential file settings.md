
https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html#cli-configure-files-examples

You can save your frequently used configuration settings and credentials in files that are maintained by the AWS CLI.
The files are divided into `profiles`. By default, the AWS CLI uses the settings found in the profile named `default`. To use alternate settings, you can create and reference additional profiles.


# 1 这两个文件的区别

这两个文件中 每个 profile 都是 相互之间对应的， 同个 profile 之间都是互相填补对方的 

1
credeiental 中保存一些比较敏感的信息， 比如 aws_access_key_id， aws_secret_access_key, aws_session_token, aws_security_token
The AWS CLI stores sensitive credential information that you specify with aws configure in a local file named credentials, in a folder named .aws in your home directory.     


2
config 中保存不太敏感的信息 , 比如 region, format, role_arn , provider_id
The less sensitive configuration options that you specify with aws configure are stored in a local file named config, also stored in the .aws folder in your home directory. 

3 
尽量分为两个文件保存 ， credentials 中的资料优先被考虑 
You can keep all of your profile settings in a single file as the AWS CLI can read credentials from the config file. If there are credentials in both files for a profile sharing the same name, the keys in the credentials file take precedence. We suggest keeping credentials in the credentials files. These files are also used by the various language software development kits (SDKs). If you use one of the SDKs in addition to the AWS CLI, confirm if the credentials should be stored in their own file.

4 
使用variables去指定路径 
Where you find your home directory location varies based on the operating system, but is referred to using the environment variables %UserProfile% in Windows and $HOME or ~ (tilde) in Unix-based systems. You can specify a non-default location for the files by setting the AWS_CONFIG_FILE and AWS_SHARED_CREDENTIALS_FILE environment variables to another local path. 


# 2 Format of the configuration and credential files

The config and credentials files are organized into sections. Sections include profiles and services. A section is a named collection of settings, and continues until another section definition line is encountered. Multiple profiles and sections can be stored in the config and credentials files.

These files are plaintext files that use the following format:

- Section names are enclosed in brackets [ ] such as `[default]`, ``[profile `user1`]``, and `[sso-session]`.
- All entries in a section take the general form of `setting_name=value`.
- Lines can be commented out by starting the line with a hash character (`#`).


Each profile can specify different credentials and can also specify different AWS Regions and output formats. When naming the profile in a `config` file, include the prefix word "`profile`", but do not include it in the `credentials` file.

The following examples show a `credentials` and `config` file with two profiles, region, and output specified. The first _[default]_ is used when you run a AWS CLI command with no profile specified. The second is used when you run a AWS CLI command with the `--profile user1` parameter.

## 2.1 Section type: `profile`

The AWS CLI stores

Depending on the file, profile section names use the following format:

- **Config file:** `[default]` ``[profile `user1`]``
- **Credentials file:** `[default]` ``[`user1`]``
    Do **_not_** use the word `profile` when creating an entry in the `credentials` file.
    

Each profile can specify different credentials and can also specify different AWS Regions and output formats. When naming the profile in a `config` file, include the prefix word "`profile`", but do not include it in the `credentials` file.

The following examples show a `credentials` and `config` file with two profiles, region, and output specified. The first _[default]_ is used when you run a AWS CLI command with no profile specified. The second is used when you run a AWS CLI command with the `--profile user1` parameter.

## 2.2 Section type: `services`

The `services` section is a group of settings that configures custom endpoints for AWS service requests. A profile then is linked to a `services` section.

```
[profile dev]
services = my-services
```

The `services` section is separated into subsections by `<SERVICE> =` lines, where `<SERVICE>` is the AWS service identifier key. The AWS service identifier is based on the API model’s `serviceId` by replacing all spaces with underscores and lowercasing all letters. For a list of all service identifier keys to use in the `services` section, see [Use endpoints in the AWS CLI](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-endpoints.html). The service identifier key is followed by nested settings with each on its own line and indented by two spaces.

The following example configures the endpoint to use for requests made to the Amazon DynamoDB service in the `my-services` section that is used in the `dev` profile. Any immediately following lines that are indented are included in that subsection and apply to that service.

```
[profile dev]
services = my-services

[services my-services]
dynamodb = 
  endpoint_url = http://localhost:8000
```

For more information on service-specific endpoints, see [Use endpoints in the AWS CLI](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-endpoints.html).

If your profile has role-based credentials configured through a `source_profile` parameter for IAM assume role functionality, the SDK only uses service configurations for the specified profile. It does not use profiles that are role chained to it. For example, using the following shared `config` file:

```
[profile A]
credential_source = Ec2InstanceMetadata
endpoint_url = https://profile-a-endpoint.aws/

[profile B]
source_profile = A
role_arn = arn:aws:iam::123456789012:role/roleB
services = profileB

[services profileB]
ec2 = 
  endpoint_url = https://profile-b-ec2-endpoint.aws
```

If you use profile `B` and make a call in your code to Amazon EC2, the endpoint resolves as `https://profile-b-ec2-endpoint.aws`. If your code makes a request to any other service, the endpoint resolution will not follow any custom logic. The endpoint does not resolve to the global endpoint defined in profile `A`. For a global endpoint to take effect for profile `B`, you would need to set `endpoint_url` directly within profile `B`.

# 3 使用awsCLI的时候加上`--profile profile-name`

If no profile is explicitly defined, the default profile is used.

To use a named profile, add the --profile profile-name option to your command. The following example lists all of your Amazon EC2 instances using the credentials and settings defined in the user1 profile.

$ aws ec2 describe-instances --profile user1


下面的内容 是 使用了 aws-adfs login --profile ivu-cloud-e2x --no-sspi --region eu-central-1 --adfs-host adfs02.ivu-cloud.com 后 自动 登录进入  c:\Users\yzh\.aws\config 的。 不是自己写的 


# 4 使用`aws configure` Set and view configuration settings 

aws configure --profile userprod
aws configure  ： 填入default 这个 profile 

There are several ways to view and set your configuration settings using commands.

使用 `aws configure` 但是不加上 --profile ， 就会自动 更新的是 default 这个 profile 
## 4.1 aws configure: 创建新的 profile 

Run this command to quickly set and view your credentials, Region, and output format. The following example shows sample values.

```
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

## 4.2 aws configure set

You can set any credentials or configuration settings using `aws configure set`. Specify the profile that you want to view or modify with the `--profile` setting.

For example, the following command sets the `region` in the profile named `integ`.

```
$ aws configure set region us-west-2 --profile integ
```

To remove a setting, use an empty string as the value, or manually delete the setting in your `config` and `credentials` files in a text editor.

```
$ aws configure set cli_pager "" --profile integ
```

## 4.3 aws configure get 

You can retrieve any credentials or configuration settings you've set using `aws configure get`. Specify the profile that you want to view or modify with the `--profile` setting.

For example, the following command retrieves the `region` setting in the profile named `integ`.

```
$ aws configure get region --profile integ
us-west-2
```

If the output is empty, the setting is not explicitly set and uses the default value.

## 4.4 aws configure get 

aws configure list --profile ivu-cloud-e20


To list configuration data, use the `aws configure list` command. This command lists the profile, access key, secret key, and region configuration information used for the specified profile. For each configuration item, it shows the value, where the configuration value was retrieved, and the configuration variable name.

For example, if you provide the AWS Region in an environment variable, this command shows you the name of the region you've configured, that this value came from an environment variable, and the name of the environment variable.

For temporary credential methods such as roles and IAM Identity Center, this command displays the temporarily cached access key and secret access key is displayed.

```
$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************ABCD  shared-credentials-file    
secret_key     ****************ABCD  shared-credentials-file    
    region                us-west-2             env    AWS_DEFAULT_REGION
```

## 4.5 aws configure list-profiles

aws configure list-profiles
```
ivu-cloud-e20
ivu-cloud-e31
ivu-cloud-e2x
ivu-cloud-e2c
eks-training
default
```

