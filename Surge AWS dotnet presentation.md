# Introduction to AWS With C#/.NET

This document provides an overview of Amazon Web Services (AWS) and using AWS with C#/.NET.

## GENERAL

AWS CLI (version 2):
https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html

AWS Visual Studio Toolkit:
https://aws.amazon.com/visualstudio/

AWS documentation:
https://docs.aws.amazon.com/

LocalStack
https://localstack.cloud/




## Amazon IAM

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
### Amazon S3 Console
https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html

### Buckets (bucket name)
https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-buckets-s3.html

### Objects (object key a.k.a. key name)
https://docs.aws.amazon.com/AmazonS3/latest/userguide/uploading-downloading-objects.html

https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-keys.html

#### Object Metadata
https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingMetadata.html

### CLI Commands
https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3-commands.html

List all buckets:

    aws s3 ls


## Lambda
### S3 Event
https://docs.aws.amazon.com/cli/latest/reference/s3api/put-bucket-notification-configuration.html
https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetBucketNotificationConfiguration.html

    aws s3api get-bucket-notification-configuration help

    aws s3api get-bucket-notification-configuration --bucket mybucketname


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



## CloudWatch

### CloudWatchLogs

## Amazon Simple Email Service

## API Gateway

## SAM (Serverless Application Model)

