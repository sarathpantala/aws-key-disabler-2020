

# AWS Lambda - IAM Access Key Disabler

![Image of KeyIcon](/docs/images/GitHubTepapaKeyIcon.jpg)

The AWS Key disabler is a Lambda Function that disables AWS IAM User Access Keys after a set amount of time in order to reduce the risk associated with old access keys.

## AWS Lambda Architecture

![Image of Arch](/docs/images/GitHubTepapaLambda.png)

## SysOps Output for EndUser

![Image of iPhoneEmail](/docs/images/GitHubTepapaOutput.png)

## Developer Toolchain

![Image of Toolchain](/docs/images/GitHubTepapaToolchain.png)

## Features

Can be configued to:
* Email a warning message to any IAM user account whose access keys are about to expire or have been expired, refer to the `email.user` section within the `/grunt/package.json` build configuration file. For this to work the IAM user account must use a valid email for the username.

* Email a report containing the output (json) for a single key scan to a single defined administrator account, refer to the `email.admin` section within the `/grunt/package.json` build configuration file.

Note: IAM Access Keys are only **disabled**, not deleted nor replaced

## Prerequisites

This script requires the following tools to run.
* NPM installed https://nodejs.org/en/ - tested with version `6.14.8`
* Gruntjs installed http://gruntjs.com/ - tested with version `grunt-cli v1.3.2`
* AWSCLI commandline tool installed https://aws.amazon.com/cli/ - tested with version `aws-cli/2.0.58`

It also assumes that you have an existing AWS account with SES (email) enabled. SES usage requires that you have a domain verified and sandbox mode removed.

## Installation Instructions

These instructions are for OSX. Your mileage may vary on Windows and other \*nix.

Before you start make sure that your AWSCLI configuration has been correctly setup with the right credentials and that it can authenitcate into your AWS account. Run the following command to check who it is authenticating as:

```
aws iam get-user
``` 

Make sure that this AWS IAM user has IAM admin *like* priviledges - so that it can create the various AWS resources (Lambda, IAM, CloudWatch etc.) used by this project.

1. Grab yourself a copy of this repo `git clone https://github.com/jeremycook123/aws-key-disabler-2020`
2. Navigate into the projects `grunt` folder: run `cd aws-key-disabler-2020/grunt`
3. Setup the Grunt task runner, e.g. install its dependencies: run `npm install`
4. Update the custom configuration within the `/grunt/package.json` file:

* Replace `key_disabler.aws_account_number` with your own AWS Account Id.
* Update `key_disabler.keystates.first_warning` and `key_disabler.keystates.last_warning` to the age that the key has to be in days to trigger an email warning.
* Update `key_disabler.keystates.expired` to the age in days when the key expires. At this age the key is disabled.
* Set `email.admin.enable` to `True` if you want to send an email report to an administrator email address containing a full report of all IAM users and their Access Key status. Email delivery is performed via AWS SES (make sure that it has been configured correctly). Configure `email.admin.to` to be a valid email address.
* Set `email.user.enable` to `True` if you want to send an individual email to each IAM user - containing the information about their Access Key status and whether a particular key is due to be expired or has been expired. Note: this only works for IAM accounts where the username is a valid email address. Email delivery is performed via AWS SES (make sure that SES has been configured correctly).
* Update the `email.from` to be a valid email address.
* Set the `lambda.deployment_region` to a region that supports Lambda and SES. Also ensure that the region has SES sandbox mode disabled.
* Update the `lambda.schedule.expression` to be a valid cron job expression for when you want the Lambda function automatically triggered.

5. From within the `/grunt` directory - run `grunt bumpup && grunt deployLambda` to bump your release version number and perform a build/deploy of the Lambda function to the selected region

## Invoking the Lambda Function manually from the commandline using the AWSCLI

This is useful for testing and performing adhoc runs

Execute the lambda function by name, `AccessKeyRotation`, logging the output of the scan to a file called `scan.report.log`:

`aws lambda invoke --function-name AccessKeyRotation scan.report.log --region us-east-1`
```javascript
{
    "StatusCode": 200
}
```

Use `jq` to render the contents of the `scan.report.log` to the console:

`jq '.' scan.report.log`
```javascript
{
  "reportdate": "2016-06-26 10:37:24.071091",
  "users": [
    {
      "username": "TestS3User",
      "userid": "1",
      "keys": [
        {
          "age": 72,
          "changed": false,
          "state": "key is already in an INACTIVE state",
          "accesskeyid": "**************Q3GA1"
        },
        {
          "age": 12,
          "changed": false,
          "state": "key is still young",
          "accesskeyid": "**************F3AA2"
        }
      ]
    },
    {
      "username": "BlahUser22",
      "userid": "2",
      "keys": []
    },
    {
      "username": "LambdaFake1",
      "userid": "3",
       "keys": [
        {
          "age": 23,
          "changed": false,
          "state": "key is due to expire in 1 week (7 days)",
          "accesskeyid": "**************DFG12"
        },
        {
          "age": 296,
          "changed": false,
          "state": "key is already in an INACTIVE state",
          "accesskeyid": "**************4ZASD"
        }
      ]
    },
    {
      "username": "apiuser49",
      "userid": "4",
       "keys": [
        {
          "age": 30,
          "changed": true,
          "state": "key is now EXPIRED! Changing key to INACTIVE state",
          "accesskeyid": "**************ER2E2"
        },
        {
          "age": 107,
          "changed": false,
          "state": "key is already in an INACTIVE state",
          "accesskeyid": "**************AWQ4K"
        }
      ]
    },
    {
      "username": "UserEMRKinesis",
      "userid": "5",
       "keys": [
        {
          "age": 30,
          "changed": false,
          "state": "key is now EXPIRED! Changing key to INACTIVE state",
          "accesskeyid": "**************MGB41A"
        }
      ]
    },
    {
      "username": "CDN-Drupal",
      "userid": "6",
       "keys": [
        {
          "age": 10,
          "changed": false,
          "state": "key is still young",
          "accesskeyid": "**************ZDSQ5A"
        },
        {
          "age": 5,
          "changed": false,
          "state": "key is still young",
          "accesskeyid": "**************E3ODA"
        }
      ]
    },
    {
      "username": "ChocDonutUser1",
      "userid": "7",
       "keys": [
        {
          "age": 59,
          "changed": false,
          "state": "key is already in an INACTIVE state",
          "accesskeyid": "**************CSA123"
        }
      ]
    },
    {
      "username": "ChocDonut2",
      "userid": "8",
       "keys": [
        {
          "age": 60,
          "changed": false,
          "state": "key is already in an INACTIVE state",
          "accesskeyid": "**************FDGD2"
        }
      ]
    },
    {
      "username": "admin.skynet@cyberdyne.systems.com",
      "userid": "9",
       "keys": [
        {
          "age": 45,
          "changed": false,
          "state": "key is already in an INACTIVE state",
          "accesskeyid": "**************BLQ5GJ"
        },
        {
          "age": 71,
          "changed": false,
          "state": "key is already in an INACTIVE state",
          "accesskeyid": "**************GJFF53"
        }
      ]
    }
  ]
}
```

## Additional configuration option

* You can choose to set the message used for each warning and the final disabling by changing the values under `key_disabler.keystates.<state>.message`
* You can change the length of masking under `key_disabler.mask_accesskey_length`. The access keys are 20 characters in length.

## Troubleshooting

This script is provided as is. We are happy to answer questions as time allows but can't give any promises.

If things don't work ensure that:
* You can authenticate successfully against AWS using the AWSCLI commandline tool
* SES is not in sandbox mode and the sender domain has been verified
* The selected region provides both Lambda and SES https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/

## Bonus Points

Once the Lambda Function has been successfully deployed - the following commands can be performed:

1. `aws lambda list-functions`
2. `openssl dgst -binary -sha256 ..\Releases\AccessKeyRotationPackage.1.0.18.zip | openssl base64`
3. `aws lambda invoke --function-name AccessKeyRotation report.log --region us-east-1`
4. `jq '.' report.log`
5. `jq '.users[] | select(.username=="johndoe")' report.log`
5. `jq '.' report.log | grep age | cut -d':' -f2 | sort -n`

## Bonus Bonus Points

1. `jq 'def maximal_by(f): (map(f) | max) as $mx | .[] | select(f == $mx); .users | maximal_by(.keys[].age)' report.log`
2. `jq 'def minimal_by(f): (map(f) | min) as $mn | .[] | select(f == $mn); .users | minimal_by(.keys[].age)' report.log`
