Here’s a step-by-step guide on how to implement the AWS Lambda function for processing S3 events and sending notifications to Amazon SNS and SQS using the AWS Management Console:

### **Step 1: Set Up the S3 Bucket**
1. **Go to the AWS S3 Console**:
   - Sign in to the [S3 console](https://console.aws.amazon.com/s3/).
   
2. **Create a New S3 Bucket** (if you don't have one already):
   - Click **Create bucket**.
   - Enter a **bucket name** and select the desired AWS region.
   - Click **Create bucket** to create it.

3. **Enable Event Notifications**:
   - Go to the **Properties** tab of the S3 bucket.
   - Scroll down to **Event notifications** and click **Create event notification**.
   - In **Event name**, specify a name.
   - Choose **All object create events**.
   - Under **Destination**, choose **Lambda Function** (you will associate it with your Lambda function later).

### **Step 2: Create an SQS Queue**
1. **Go to the AWS SQS Console**:
   - Sign in to the [SQS console](https://console.aws.amazon.com/sqs/).

2. **Create a New Queue**:
   - Click **Create queue**.
   - Select **Standard queue**.
   - Provide a **name** for the queue and click **Create queue**.

3. **Copy the Queue URL**:
   - After the queue is created, click on the queue name and copy the **Queue URL** (you will use this URL in the Lambda function).

### **Step 3: Create an SNS Topic**
1. **Go to the AWS SNS Console**:
   - Sign in to the [SNS console](https://console.aws.amazon.com/sns/).

2. **Create a New Topic**:
   - Click **Create topic**.
   - Choose **Standard** as the type.
   - Provide a **name** for the topic (e.g., `My-notification-topic`) and click **Create topic**.

3. **Add a Subscription (Optional)**:
   - Once the topic is created, click on the topic name.
   - Click **Create subscription**.
   - Choose the protocol (e.g., **Email**, **SMS**) and enter the endpoint (email address or phone number) to receive notifications.
   - Click **Create subscription**.
   
4. **Copy the Topic ARN**:
   - After creating the topic, copy the **Topic ARN** (you will need it in your Lambda function).

### **Step 4: Create the Lambda Function**
1. **Go to the AWS Lambda Console**:
   - Sign in to the [Lambda console](https://console.aws.amazon.com/lambda/).

2. **Create a New Lambda Function**:
   - Click **Create function**.
   - Choose **Author from scratch**.
   - Provide a **name** (e.g., `S3EventProcessor`) for the function.
   - Select **Python 3.x** as the runtime.
   - Choose or create an appropriate **execution role** (the role should allow access to S3, SQS, and SNS).

3. **Add the Lambda Code**:
   - In the function code section, delete the default code and paste the Lambda code from the previous explanation:
     ```python
     import json
     import boto3

     s3_client = boto3.client('s3')
     sns_client = boto3.client('sns')
     sqs_client = boto3.client('sqs')

     def lambda_handler(event, context):
         sns_topic_arn = 'arn:aws:sns:us-west-2:411854276167:My-notification-topic'
         sqs_queue_url = 'https://sqs.us-west-2.amazonaws.com/411854276167/My-notification-queue'

         for record in event['Records']:
             print(event)
             
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
4. **Replace ARNs and URLs**:
   - Replace the **SNS Topic ARN** and **SQS Queue URL** in the code with the values you copied earlier.

5. **Configure Function Permissions**:
   - In the **Execution role** settings, ensure that your Lambda function has a role with the following policies:
     - **AmazonS3ReadOnlyAccess** (or equivalent policy for reading S3 event data).
     - **AmazonSNSFullAccess** or similar to publish messages to SNS.
     - **AmazonSQSFullAccess** or similar to send messages to SQS.

### **Step 5: Configure the Lambda Trigger**
1. **Add S3 as a Trigger**:
   - In the Lambda function console, scroll to the **Function Overview** section.
   - Click **Add Trigger**.
   - Select **S3** as the trigger source.
   - Choose your S3 bucket and select **All object create events**.
   - Click **Add** to create the trigger.

2. **Save the Lambda Function**:
   - Click **Deploy** to save your changes.

### **Step 6: Test the Function**
1. **Upload a File to the S3 Bucket**:
   - Go to your S3 bucket and upload a file.
   
2. **Verify the Output**:
   - The Lambda function should automatically trigger when a file is uploaded to the bucket.
   - The function will send metadata to the SQS queue and publish a notification to the SNS topic.
   
3. **Check the SQS Queue**:
   - Go to the SQS console, select your queue, and verify that a new message has been added with the file metadata.

4. **Check the SNS Subscription**:
   - If you have subscribed an email or SMS to the SNS topic, check your inbox for a notification about the file upload.

### **Step 7: (Optional) Monitor and Debug the Lambda Function**
1. **CloudWatch Logs**:
   - Go to the **CloudWatch** console to view the logs generated by your Lambda function.
   - In the logs, you can see details of the S3 event and any errors encountered during execution.

By following these steps, you will have successfully set up an AWS Lambda function that processes S3 events, sends metadata to SQS, and sends notifications via SNS.
