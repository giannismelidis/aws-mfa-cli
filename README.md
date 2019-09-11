# MFA enabled programmatic access in AWS

## Installation

Clone the repository into a directory:
`git clone https://github.com/giannismelidis/aws-mfa-cli` then `cd aws-mfa-cli`.

Move the script to `/usr/local/bin` directory by running `cp aws-mfa /usr/local/bin` and make it an executable with `chmod +x /usr/local/bin/aws-mfa`.

## Requirements

The script is written in Ruby. For Ruby installation instructions, go to https://www.ruby-lang.org/en/documentation/installation/.

You can install the requirements of the script by running `bundle install`.
If you don't have bundler installed, run `gem install bundler` first.

## Running the script

Once you have completed all the above, given that you have a set of AWS credentials (let's assume it is named `default`) in `~/.aws/credentials`,
you can run `INPUT_PROFILE=default OUTPUT_PROFILE=default-mfa aws-mfa`. It will then ask for your MFA token, that you can get from your installed authenticator application on your phone.
Once you input the MFA token, it will give you a success message. At the bottom of the `~/.aws/credentials` file a new block should have been generated:

```
[default-mfa]
aws_access_key_id = XXX
aws_secret_access_key = XXX
aws_session_token = XXX
```

You can check if it works by running: `AWS_PROFILE=default-mfa aws s3 ls` (given that you have access to list s3 buckets in this account)

For convenience you can add this line in your `~/.aws/bash_profile` file:
`alias awsmfa="INPUT_PROFILE=default OUTPUT_PROFILE=default-tmp aws-mfa"`

Then you can just run `awsmfa` at the beginning of your working day, enter the MFA token and you should be good to go!
