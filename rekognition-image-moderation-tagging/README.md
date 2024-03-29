# Amazon Rekognition to Automatically Label Stored Images.

This pattern explains how to deploy an AWS SAM application with Amazon S3, AWS Lambda, and Amazon DynamoDB to detect (inappropriate images)[https://docs.aws.amazon.com/rekognition/latest/APIReference/API_DetectModerationLabels.html]. When an image file is created in Amazon S3, the event-driven workflow begins, sending an event to AWS Lambda. This Lambda function, written in Python, invokes the Amazon Rekognition `DetectModerationLabels` function using boto3. Once Rekognition's function has returned it's response, Lambda stores the detected moderation labels in Amazon DynamoDB table.

Learn more about this pattern at Serverless Land Patterns: [https://serverlessland.com/patterns/rekognition-video-tagging](https://serverlessland.com/patterns/rekognition-video-tagging) 

Important: this application uses various AWS services and there are costs associated with these services after the Free Tier usage - please see the [AWS Pricing page](https://aws.amazon.com/pricing/) for details. You are responsible for any AWS costs incurred. No warranty is implied in this example.

## Requirements

* [Create an AWS account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html) if you do not already have one and log in. The IAM user that you use must have sufficient permissions to make necessary AWS service calls and manage AWS resources.
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) installed and configured
* [Git Installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [AWS Serverless Application Model](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) (AWS SAM) installed

## Deployment Instructions

1. Create a new directory, navigate to that directory in a terminal and clone the GitHub repository:
    ``` 
    git clone https://github.com/aws-samples/serverless-patterns
    ```
2. Change directory to the pattern directory:
    ```
    cd rekognition-image-moderation-tagging
    ```
3. From the command line, use AWS SAM to deploy the AWS resources for the pattern as specified in the template.yml file:
    ```
    sam deploy --guided
    ```
4. During the prompts:
    * Enter a stack name
    * Enter the desired AWS Region
    * Allow SAM CLI to create IAM roles with the required permissions.

    Once you have run `sam deploy --guided` mode once and saved arguments to a configuration file (samconfig.toml), you can use `sam deploy` in future to use these defaults.

5. Note the outputs from the SAM deployment process. These contain the resource names and/or ARNs which are used for testing.

## How it works

- This pattern is designed to create all services required to run this workflow.
- The workflow begins with an Amazon S3 bucket. 
- When an object is created within the S3 bucket, it sends an event to a AWS Lambda function.
- This Lambda function invokes Amazon Rekognition's [DetectModerationLabels](https://docs.aws.amazon.com/rekognition/latest/APIReference/API_DetectModerationLabels.html) function, which synchronously analyses the newly stored image.
- Once this task is complete, the Lambda function stores the results in an Amazon DynamoDB table.

## Testing
1. Upload a test image (.jpeg) which contains a (label category)[https://docs.aws.amazon.com/rekognition/latest/dg/moderation.html] to the Amazon S3 bucket created during the deployment step. You can either do this via the console, or by running the following command. Replace `<your_image.jpeg>` with your image file name. Replace `<S3BucketName>` with the name of the S3 bucket generated from the AWS SAM deployment outputs.
    ```
    aws s3 cp <your_image.jpeg> s3://<S3BucketName>
    ```
3. Wait for the lambda-start-processing-image-rekognition Lambda to complete, then retrieve the record output from the DynamoDB table. Replace `<DDBTableName>` with the name of the DynamoDB table generated by the AWS SAM deployment output, in this case, it will be `ImageModerationTags`. You can use the following command:
    ```
    aws dynamodb scan --table-name <DDBTableName>
    ```
    For example: aws dynamodb scan --table-name ImageModerationTags
4. There should now be a newly uploaded record within DynamoDB displayed, depending on the image, the key 'ModerationLabels' will either be empty, or contain values indicating what was found in the image.

## Cleanup
 
1. Delete the stack
    ```
    sam delete
    ```

⚠️ **IMPORTANT** - The above command `sam delete` does not delete the **Amazon S3 bucket** if there are still objects stored within it. Please [empty the bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/empty-bucket.html) before running the above command.
----
Copyright 2023 Amazon.com, Inc. or its affiliates. All Rights Reserved.

SPDX-License-Identifier: MIT-0