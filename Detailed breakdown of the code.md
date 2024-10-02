# AWS-S3-Event-Notification-with-Lambda-SQS-and-SNS

This AWS Lambda function processes S3 event notifications, sending metadata to an Amazon SQS queue and sending notifications to an Amazon SNS topic whenever a file is uploaded to an S3 bucket. Here’s a detailed breakdown of the code:

### 1. **Imports**
```python
import json
import boto3
```
- **json:** This is the standard Python module to work with JSON (JavaScript Object Notation) data. It will be used for serializing and deserializing data, such as sending metadata to SQS.
- **boto3:** This is the AWS SDK for Python that allows the function to interact with AWS services like S3, SNS, and SQS.

### 2. **Boto3 Clients**
```python
s3_client = boto3.client('s3')
sns_client = boto3.client('sns')
sqs_client = boto3.client('sqs')
```
These three lines create **clients** for the following AWS services:
- **S3 (Simple Storage Service):** For interacting with S3, although it's not directly used in the code (could be for future functionality).
- **SNS (Simple Notification Service):** For sending notifications to a specific topic when a new file is uploaded.
- **SQS (Simple Queue Service):** For sending messages (metadata) about the S3 event to an SQS queue.

### 3. **Lambda Handler Function**
```python
def lambda_handler(event, context):
    sns_topic_arn = 'arn:aws:sns:us-west-2:411854276167:My-notification-topic'
    sqs_queue_url = 'https://sqs.us-west-2.amazonaws.com/411854276167/My-notification-queue'
```
- **`lambda_handler`:** This is the main function that will be executed whenever the Lambda function is triggered.
- **`event`:** Contains the data that triggered the Lambda function, typically an S3 event notification.
- **`context`:** Provides runtime information to the Lambda function (e.g., request ID, timeout information).

Inside this function:
- **`sns_topic_arn`:** ARN (Amazon Resource Name) of the SNS topic where notifications will be sent.
- **`sqs_queue_url`:** URL of the SQS queue to which metadata about the uploaded file will be sent.

### 4. **Processing S3 Event Records**
```python
for record in event['Records']:
    print(event)
```
- The **event** contains records of S3-related events (e.g., file uploads), and this line iterates over each event in the `Records` list.
- **`print(event)`**: Outputs the event information to CloudWatch logs for debugging or monitoring purposes.

### 5. **Extracting S3 Bucket and Object Information**
```python
s3_bucket = record['s3']['bucket']['name']
s3_key = record['s3']['object']['key']
```
- **`s3_bucket`:** The name of the S3 bucket where the event occurred (in this case, where a file was uploaded).
- **`s3_key`:** The key (file path/name) of the object that was uploaded to the S3 bucket.

### 6. **Sending Metadata to SQS**
```python
metadata = {
    'bucket': s3_bucket,
    'key': s3_key,
    'timestamp': record['eventTime']
}
```
- **`metadata`:** A dictionary containing the metadata for the uploaded file:
    - **`bucket`:** Name of the S3 bucket.
    - **`key`:** File name/key of the uploaded object.
    - **`timestamp`:** Timestamp of the S3 event (when the file was uploaded).

```python
sqs_response = sqs_client.send_message(
    QueueUrl=sqs_queue_url,
    MessageBody=json.dumps(metadata)
)
```
- **`sqs_client.send_message`:** Sends the metadata as a message to the SQS queue.
    - **`QueueUrl`:** Specifies the URL of the SQS queue.
    - **`MessageBody`:** The message itself, which is the metadata converted to JSON using `json.dumps()`.

### 7. **Sending a Notification to SNS**
```python
notification_message = f"New file uploaded to S3 bucket '{s3_bucket}' with key '{s3_key}'"
```
- **`notification_message`:** A string containing information about the uploaded file. This message will be sent to the SNS topic.

```python
sns_response = sns_client.publish(
    TopicArn=sns_topic_arn,
    Message=notification_message,
    Subject="File Upload Notification"
)
```
- **`sns_client.publish`:** Publishes the notification message to the specified SNS topic.
    - **`TopicArn`:** ARN of the SNS topic to which the message will be sent.
    - **`Message`:** The notification message, informing subscribers that a new file has been uploaded.
    - **`Subject`:** A subject line for the message, set as "File Upload Notification".

### 8. **Return Success Response**
```python
return {
    'statusCode': 200,
    'body': json.dumps('Processing complete')
}
```
- This returns a response indicating the Lambda function has successfully processed the event.
    - **`statusCode: 200`:** HTTP status code indicating success.
    - **`body`:** The message "Processing complete" serialized to JSON.

### **Summary**
- **Trigger:** The function is triggered by an S3 event (e.g., file upload).
- **Processing:** For each event, it extracts the S3 bucket and file key, then:
  - Sends metadata (bucket, key, timestamp) to an SQS queue.
  - Publishes a notification to an SNS topic to inform subscribers about the file upload.
- **Response:** Returns an HTTP status code (200) upon successful processing. 

This architecture is commonly used in serverless setups where you want to notify other systems (via SQS or SNS) about S3 bucket events (like file uploads) without having to manage servers.
