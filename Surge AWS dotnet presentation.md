# Introduction to AWS With C#/.NET

This document provides an overview of Amazon Web Services (AWS) and using AWS with C#/.NET.

Here are just a few of the AWS services:
- IAM : Identity and Access Management
- S3 : storage (files, etc. as "objects")
- Lambda : functions which execute based on certain events
- SQS : Simple Queue Service
- SNS : Simple Notification Service
- CloudWatch : system monitoring, logging, etc.
- SES : Simple Email Service
- API Gateway : create REST API from Swagger
- SAM : Serverless Application Model (use YAML to create your AWS resources)

## GENERAL

GitHub repo containing this document and related files:
https://github.com/mdhatmaker/aws-dotnet-tutorial

AWS CLI (version 2):
https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html

AWS Visual Studio Toolkit:
https://aws.amazon.com/visualstudio/

AWS documentation:
https://docs.aws.amazon.com/

### LocalStack
https://localstack.cloud/

To start LocalStack with *all* available AWS services (from command line):

    localstack start

Alternatively, you can create a docker-compose.yml file and start LocalStack with *only* the services that you want.

To use the AWS CLI with a running LocalStack, simply append "--endpoint http://localhost:4566" to the end of every AWS CLI command line.

For example, to get a list of buckets available via LocalStack:

    aws s3 ls --endpoint http://localhost:4566

### Configuration

Use your configuration profile(s) to set up access_key, secret_key, and region:

    aws configure

    aws configure list

Your AWS configuration files will typically reside in **<user_home>/.aws/config** and **<user_home>/.aws/credentials**.

Sample **.aws/config**:

    [default]
    region = us-west-2
    output = json

Sample **.aws/credentials**:

    [default]
    aws_access_key_id = ABC123ABC123ABC123AB
    aws_secret_access_key = VaZ2ABnL2rBCtACwPmhHBCaBC12sA0uKvABCDEF2

    [myprofile]
    aws_access_key_id = ABC123ABC123ABC123AB
    aws_secret_access_key = VaZ2ABnL2rBCtACwPmhHBCaBC12sA0uKvABCDEF2
    region = us-east-1
    output = json


## Amazon IAM (Identity and Access Management)

AWS Identity and Access Management (IAM) enables you to manage access to AWS services and resources securely. Using IAM, you can create and manage AWS users and groups, and use permissions to allow and deny their access to AWS resources.

https://aws.amazon.com/iam/

### Lambda function
Create the execution role that gives your function permission to access AWS resources. To create an execution role with the AWS CLI, use the create-role command.

    aws iam create-role --role-name lambda-ex --assume-role-policy-document '{"Version": "2012-10-17","Statement": [{ "Effect": "Allow", "Principal": {"Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole"}]}'

You can also define the trust policy for the role using a JSON file. In the following example, trust-policy.json is a file in the current directory. This trust policy allows Lambda to use the role's permissions by giving the service principal lambda.amazonaws.com permission to call the AWS Security Token Service AssumeRole action.

Example **trust-policy.json**:

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "Service": "lambda.amazonaws.com"
                },
                "Action": "sts:AssumeRole"
            }
        ]
    }

Use this trust policy JSON file to create-role:

    aws iam create-role --role-name lambda-ex --assume-role-policy-document file://trust-policy.json




## S3
Amazon S3 Documentation:
https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html

Amazon S3 Console:
https://console.aws.amazon.com/s3/

Buckets (bucket name):
https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-buckets-s3.html

Objects (object key a.k.a. key name):
https://docs.aws.amazon.com/AmazonS3/latest/userguide/uploading-downloading-objects.html

Object Metadata:
https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingMetadata.html

### CLI Commands
https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3-commands.html

List all buckets:

    aws s3 ls

List objects in bucket:

    aws s3 ls s3://mybucket

    aws s3 ls s3://mybucket --recursive

Liet objects in bucket that begin with "documents" prefix:

    aws s3 ls s3://mybucket/documents --recursive

Sync bucket with local folder:

    aws s3 sync C:\temp s3://mybucket1




## Lambda
Some useful links related to lambdas:

https://medium.com/@pekelny/how-to-unit-test-an-aws-lambda-524069d4fe06

https://towardsdatascience.com/how-i-write-meaningful-tests-for-aws-lambda-functions-f009f0a9c587

### S3 Event
https://docs.aws.amazon.com/cli/latest/reference/s3api/put-bucket-notification-configuration.html

https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetBucketNotificationConfiguration.html

    aws s3api get-bucket-notification-configuration help

    aws s3api get-bucket-notification-configuration --bucket mybucketname

    aws s3api put-bucket-notification-configuration --bucket mybucketname --notification-configuration file://notification.json

Sample **notification.json** file:

    {
        "LambdaFunctionConfigurations": [
            {
                "Id": "s3-putobject-event",
                "LambdaFunctionArn": "arn:aws:lambda:us-west-2:869937370881:function:Dev-MDMBulkSwitchCoreAgency",
                "Events": [
                    "s3:ObjectCreated:*"
                ],
                "Filter": {
                    "Key": {
                        "FilterRules": [
                            {
                                "Name": "Prefix",
                                "Value": ""
                            },
                            {
                                "Name": "Suffix",
                                "Value": ".xlsx"
                            }
                        ]
                    }
                }
            }
        ]
    }



### SQS Event

### CLI Commands
https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-awscli.html

    aws lambda list-functions --max-items 10

    aws lambda list-functions --max-items 10 --starting-token eyJNYXJrZXIiOiBudWxsLCAiYm90b190cnVuY2F0ZV9hbW91bnQiOiAxMH0=

Before create-function, write code in **index.js** file (and "zip function.zip index.js" to create a ZIP file).

Example **index.js**:

    exports.handler = async function(event, context) {
        console.log("ENVIRONMENT VARIABLES\n" + JSON.stringify(process.env, null, 2))
        console.log("EVENT\n" + JSON.stringify(event, null, 2))
        return context.logStreamName
    }

<br>

    aws lambda create-function --function-name my-function \
    --zip-file fileb://function.zip --handler index.handler --runtime nodejs12.x \
    --role arn:aws:iam::123456789012:role/lambda-ex

The Lambda CLI get-function command returns Lambda function metadata and a presigned URL that you can use to download the function's deployment package.

    aws lambda get-function --function-name my-function

Invoke our function:

    aws lambda invoke --function-name my-function out --log-type Tail

    aws lambda invoke --function-name my-function out --log-type Tail \
    --query 'LogResult' --output text |  base64 -d

    aws lambda invoke --function-name my-function --cli-binary-format raw-in-base64-out --payload '{"key": "value"}' out

We can look at the logs produced by our function:

    aws logs get-log-events --log-group-name /aws/lambda/my-function --log-stream-name <logStreamID> --limit 5

<br>

    aws lambda delete-function --function-name my-function




## Simple Queue Service (SQS)

## Simple Notification Service (SNS)

## CloudWatch

### CloudWatchLogs

## Amazon Simple Email Service

## API Gateway

## SAM (Serverless Application Model)

