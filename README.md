# MFA enabled programmatic access in AWS

## Introduction
In case you want to level up your AWS Security profile (or get required to do so by your Security Team) it might an idea to enable Multi Factor Authentication not only for Console (web interface) access to AWS but also for CLI access.

Once you do - you will quickly find out that it's quite a pain to use the CLI with MFA enabled.

*The small script in this repository makes it easy for you to handle MFA on CLI*. 

At the beginning of your day, you run in once, and for the rest of the day you should be fine.

## Example

Let's assume you have a profile named `default` in your `~/.aws/credentials`

```
➜  ~ aws-mfa
Please enter MFA token for account default
123456

Credentials for `default-mfa` set successfully
```

Now you can execute any CLI command using MFA by using the `default-mfa` profile. For example:

```
➜  ~ aws s3 ls --profile default-mfa
```

## Installation

### Get the source code
Clone the repository into a directory and get into the directory:
`git clone https://github.com/giannismelidis/aws-mfa-cli` then `cd aws-mfa-cli`.

### Make it convenient to run

- Move the script to `/usr/local/bin` directory by running 
`cp aws-mfa /usr/local/bin` 

- Make the script executable
`chmod +x /usr/local/bin/aws-mfa`.

## Requirements

### Install Ruby (probably not needed)
The script is written in Ruby. It will run in most current ruby versions. You should not have to install Ruby. In case of emergency check here: https://www.ruby-lang.org/en/documentation/installation/.

### Install required libaries

#### If you have Bundler Installed
You can install the requirements of the script by running `bundle install`.

#### If you don't have bundler installed
Try installing the dependencies by hand using:

`gem install aws-sdk-iam`
`gem install inifile`

## Running the script

Once you have completed all the above, given that you have a set of AWS credentials (let's assume it is named `default`) in `~/.aws/credentials`,you can run the script as follows:

`INPUT_PROFILE=default OUTPUT_PROFILE=default-mfa aws-mfa`

It will then ask for your MFA token, that you can get from your installed authenticator application on your phone or a hardwarde device given to you.

Once you input the MFA token, it will give you a success message. 

As you probably guessed from the command above you can set the `INPUT_PROFILE` and `OUTPUT_PROFILE` to whatever matches your situation.

### If you are curious what just happened

At the bottom of the `~/.aws/credentials` file a new block should have been generated:

```
[default-mfa]
aws_access_key_id = XXX
aws_secret_access_key = XXX
aws_session_token = XXX
```

For most configurations these "temporary credentials" are valid for 12 hours by default.

### Check if the basics work

You can check if it works by running: 

`AWS_PROFILE=default-mfa aws s3 ls`

### Add some more convenience

For convenience you can add a line in your `~/.bash_profile` or `~/.zshrc`

`alias aws-mfa="INPUT_PROFILE=default OUTPUT_PROFILE=default-tmp aws-mfa"`

Then you can just run `aws-mfa` from any spot in your terminal at the beginning of your working day, enter the MFA token and you should be good to go!

```
➜  ~ aws-mfa
Please enter MFA token for account root
```


# ForceMFA Policy

In case you want to force enable MFA for CLI access, you can use the following policy document:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "iam:GetAccountPasswordPolicy",
                "iam:GetAccountSummary",
                "iam:ListVirtualMFADevices"
            ],
            "Resource": "*",
            "Effect": "Allow"
        },
        {
            "Action": [
                "iam:ChangePassword",
                "iam:GetUser"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}",
            "Effect": "Allow"
        },
        {
            "Action": [
                "iam:CreateAccessKey",
                "iam:DeleteAccessKey",
                "iam:ListAccessKeys",
                "iam:UpdateAccessKey"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}",
            "Effect": "Allow"
        },
        {
            "Action": [
                "iam:DeleteSigningCertificate",
                "iam:ListSigningCertificates",
                "iam:UpdateSigningCertificate",
                "iam:UploadSigningCertificate"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}",
            "Effect": "Allow"
        },
        {
            "Action": [
                "iam:DeleteSSHPublicKey",
                "iam:GetSSHPublicKey",
                "iam:ListSSHPublicKeys",
                "iam:UpdateSSHPublicKey",
                "iam:UploadSSHPublicKey"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}",
            "Effect": "Allow"
        },
        {
            "Action": [
                "iam:CreateServiceSpecificCredential",
                "iam:DeleteServiceSpecificCredential",
                "iam:ListServiceSpecificCredentials",
                "iam:ResetServiceSpecificCredential",
                "iam:UpdateServiceSpecificCredential"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}",
            "Effect": "Allow"
        },
        {
            "Action": [
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice"
            ],
            "Resource": "arn:aws:iam::*:mfa/${aws:username}",
            "Effect": "Allow"
        },
        {
            "Action": [
                "iam:DeactivateMFADevice",
                "iam:EnableMFADevice",
                "iam:ListMFADevices",
                "iam:ResyncMFADevice"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}",
            "Effect": "Allow"
        },
        {
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            },
            "Resource": "*",
            "Effect": "Deny",
            "NotAction": [
                "iam:CreateVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:GetUser",
                "iam:ListMFADevices",
                "iam:ListVirtualMFADevices",
                "iam:ResyncMFADevice",
                "sts:GetSessionToken"
            ],
            "Sid": "DenyAllExceptListedIfNoMFA"
        }
    ]
}
```
