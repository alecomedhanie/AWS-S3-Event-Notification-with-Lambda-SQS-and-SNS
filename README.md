# AWS Lambda Function for S3 Event Processing, SQS, and SNS

## Overview
This project demonstrates how to use an AWS Lambda function to process S3 event notifications when a new file is uploaded to an S3 bucket. The Lambda function extracts metadata from the S3 event, sends the metadata to an SQS queue, and publishes a notification to an SNS topic.

## Architecture Components
1. **Amazon S3**: Object storage where the files are uploaded. This triggers the Lambda function.
2. **AWS Lambda**: A serverless function that processes the S3 events.
3. **Amazon SQS**: A message queue that receives metadata about the S3 file upload.
4. **Amazon SNS**: A notification service that sends a message about the file upload.

## Prerequisites
Before starting, ensure you have the following:
- An **AWS account**.
- AWS **IAM role** with sufficient permissions for Lambda, S3, SNS, and SQS.
- Basic knowledge of AWS services and Python programming.

## Setup Guide

### Step 1: Create an S3 Bucket
1. Go to the AWS [S3 Console](https://console.aws.amazon.com/s3/).
2. Click **Create bucket**, provide a name, and choose a region.
3. In the **Properties** tab, enable **Event notifications** to trigger the Lambda function on file uploads.

### Step 2: Create an SQS Queue
1. Go to the AWS [SQS Console](https://console.aws.amazon.com/sqs/).
2. Click **Create Queue**, choose **Standard Queue**, provide a name, and create the queue.
3. Copy the **Queue URL** for later use.

### Step 3: Create an SNS Topic
1. Go to the AWS [SNS Console](https://console.aws.amazon.com/sns/).
2. Click **Create Topic**, choose **Standard**, and provide a name for the topic.
3. Optionally, add a subscription (Email/SMS) to receive notifications.
4. Copy the **Topic ARN** for later use.

### Step 4: Create a Lambda Function
1. Go to the AWS [Lambda Console](https://console.aws.amazon.com/lambda/).
2. Click **Create Function** and choose **Author from Scratch**.
3. Name the function, select **Python 3.x** as the runtime, and choose an IAM role with necessary permissions (S3, SNS, and SQS).
4. Replace the default code with the following:

    ```python
    import json
    import boto3

    s3_client = boto3.client('s3')
    sns_client = boto3.client('sns')
    sqs_client = boto3.client('sqs')

    def lambda_handler(event, context):
        sns_topic_arn = 'arn:aws:sns:REGION:ACCOUNT_ID:TOPIC_NAME'
        sqs_queue_url = 'https://sqs.REGION.amazonaws.com/ACCOUNT_ID/QUEUE_NAME'

        for record in event['Records']:
            s3_bucket = record['s3']['bucket']['name']
            s3_key = record['s3']['object']['key']
            
            metadata = {
                'bucket': s3_bucket,
                'key': s3_key,
                'timestamp': record['eventTime']
            }

            # Send metadata to SQS
            sqs_response = sqs_client.send_message(
                QueueUrl=sqs_queue_url,
                MessageBody=json.dumps(metadata)
            )

            # Send notification to SNS
            notification_message = f"New file uploaded to S3 bucket '{s3_bucket}' with key '{s3_key}'"
            
            sns_response = sns_client.publish(
                TopicArn=sns_topic_arn,
                Message=notification_message,
                Subject="File Upload Notification"
            )
        
        return {
            'statusCode': 200,
            'body': json.dumps('Processing complete')
        }
    ```

5. Replace the **SNS Topic ARN** and **SQS Queue URL** with your values.
6. Click **Deploy** to save the function.

### Step 5: Add S3 as a Trigger to Lambda
1. Go to the **Function Overview** section of the Lambda function.
2. Click **Add Trigger** and select **S3** as the event source.
3. Choose your S3 bucket and configure it to trigger on **Object Created** events.
4. Click **Add** to finalize the trigger.

### Step 6: Test the Setup
1. Upload a file to the S3 bucket.
2. Check the **SQS Queue** for new messages containing metadata about the uploaded file.
3. If you've subscribed to the SNS topic, check your email/SMS for a notification.

## How It Works
- **S3 Event Trigger**: When a new file is uploaded to the S3 bucket, an event notification is sent to trigger the Lambda function.
- **Lambda Function**: The function retrieves information about the uploaded file (such as the bucket name, file key, and timestamp).
    - The function sends this information as metadata to the **SQS queue**.
    - It also sends a notification message to the **SNS topic**, which can send the message to subscribers via email, SMS, or other protocols.
- **SQS**: The SQS queue receives the metadata about the uploaded file, allowing other systems to consume it asynchronously.
- **SNS**: The SNS topic sends the notification message to all subscribers.

## Permissions Required
Ensure the Lambda function has the following AWS IAM policies:
- **AmazonS3ReadOnlyAccess**: To read S3 event data.
- **AmazonSNSFullAccess**: To publish notifications to SNS.
- **AmazonSQSFullAccess**: To send messages to SQS.

## Monitoring
- You can monitor the Lambda function logs using AWS **CloudWatch Logs**.
- Go to the [CloudWatch Console](https://console.aws.amazon.com/cloudwatch/) and review the logs for successful or failed executions.

## Conclusion
This setup provides a serverless architecture for monitoring file uploads to S3 and notifying other systems asynchronously via SQS and SNS. You can expand this setup by adding more logic to the Lambda function, such as processing the uploaded files, sending alerts, or invoking additional services.
