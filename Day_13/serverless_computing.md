# Module 10: Serverless Computing — Study Notes

**Course:** AWS Cloud Infrastructure & Architecture | **Instructor:** Akhileshwar

---

## Part 1: Core Definitions

| Term | Definition |
|---|---|
| **AWS Lambda** | A serverless compute service that runs your code in response to events and automatically manages the underlying compute resources |
| **Event-Driven Architecture** | A design pattern where program flow is determined by **events** (file uploads, sensor readings, user clicks) rather than a continuous execution loop |
| **API Gateway** | A fully managed service to create, publish, maintain, monitor, and secure APIs at any scale — the interface between users and backend logic |
| **Step Functions** | A visual workflow service to orchestrate multiple AWS services into serverless workflows — manages sequencing, retries, and error handling for complex tasks |
| **Serverless Applications** | Architectural solutions built on managed cloud services, eliminating the need to provision/manage servers — lets developers focus on code and data |

---

## Part 2: Applications of Serverless Services

Serverless architecture is widely used in production for:

| Use Case | Example |
|---|---|
| **Real-time File Processing** | Automatically resizing images, processing logs, or triggering alerts the moment files are uploaded to S3 |
| **Web Application Backends** | Powering API logic, user authentication, and database interaction without managing EC2 instances |
| **Data Transformation** | ETL pipelines that trigger automatically on new data arrival in a data lake (e.g., S3 → Lambda → AWS Glue or Redshift) |
| **Automated Backups & Compliance** | Scheduled or event-driven scripts for key rotation, configuration audits, or database backups |

---

## Part 3: Practical Activity — S3 File Trigger with Email Notification

**Architecture Flow:**
```
Amazon S3 (File Upload) → AWS Lambda (Processes Event) → Amazon SNS (Sends Email) → Your Inbox
```

### Step 1: Create the Notification Channel (Amazon SNS)
> Set up SNS *first* so you have a Topic ARN to reference in your Lambda code.

1. Open the **SNS console**
2. Left menu → **Topics** → **Create topic**
3. Type: **Standard** (not FIFO)
4. Name: `S3-Upload-Alerts` → **Create topic**
5. Copy the **Topic ARN** (format: `arn:aws:sns:region:account-id:S3-Upload-Alerts`) — you'll need this for the Lambda code
6. On the topic page → **Subscriptions** tab → **Create subscription**
7. Protocol: **Email**
8. Endpoint: your personal email address → **Create subscription**
9. **Crucial:** check your email inbox for a message from "AWS Notifications" → click **Confirm subscription**

### Step 2: Create the Storage (Amazon S3)
1. Open the **S3 console** → **Create bucket**
2. Name: `serverless-upload-demo-[your-initials]-[random-numbers]` (must be globally unique)
3. Leave defaults — ensure **"Block all public access"** stays checked
4. **Create bucket**

### Step 3: Configure Permissions (IAM Role)
Lambda needs permission to read the S3 event and publish to SNS.

1. Open the **IAM console** → **Roles** → **Create role**
2. Trusted entity: **AWS service** → Use case: **Lambda** → **Next**
3. Attach these managed policies:
   - `AWSLambdaBasicExecutionRole` (allows logging)
   - `AmazonSNSFullAccess` (allows sending via SNS)
4. **Next** → Role name: `Lambda-S3-SNS-Role` → **Create role**

### Step 4: Create the Compute Logic (AWS Lambda)
1. Open the **Lambda console** → **Create function**
2. Select **Author from scratch**
3. Function name: `SendEmailOnS3Upload`
4. Runtime: **Python 3.x**
5. Expand **Change default execution role** → **Use an existing role** → select `Lambda-S3-SNS-Role`
6. **Create function**
7. In **Code source**, replace the default code with:

```python
import json
import boto3

def lambda_handler(event, context):
    # Initialize the SNS client
    sns_client = boto3.client('sns')

    # PASTE YOUR SNS TOPIC ARN HERE
    sns_topic_arn = 'arn:aws:sns:your-region:your-account-id:S3-Upload-Alerts'

    try:
        # Extract the bucket name and file name from the S3 event
        bucket_name = event['Records'][0]['s3']['bucket']['name']
        file_name = event['Records'][0]['s3']['object']['key']

        # Format the email message
        email_subject = 'AWS Alert: New File Uploaded!'
        email_body = f"Hello,\n\nA new file named '{file_name}' has just been " \
                     f"successfully uploaded to your S3 bucket '{bucket_name}'.\n\n" \
                     f"Regards,\nYour Automated Cloud Infrastructure"

        # Publish the message to SNS
        sns_client.publish(
            TopicArn=sns_topic_arn,
            Message=email_body,
            Subject=email_subject
        )

        print(f"Success! Notification sent for file: {file_name}")
        return {
            'statusCode': 200,
            'body': json.dumps('Email sent successfully!')
        }

    except Exception as e:
        print(f"Error processing the event: {str(e)}")
        raise e
```

> ⚠️ **Important:** Replace `sns_topic_arn` with the actual ARN copied in Step 1.

8. Click **Deploy** to save the code

### Step 5: Link S3 to Lambda (The Trigger)
1. Still on the Lambda function page → **Function overview** → **Add trigger**
2. Source: **S3**
3. Bucket: select the bucket created in Step 2
4. Event types: leave as **All object create events**
5. Check the acknowledgment box → **Add**

### Step 6: Test the Architecture
1. Open a new tab → navigate to your S3 bucket
2. **Upload** → select any file (image, PDF, text file, etc.) → **Upload**
3. Wait 2–5 seconds → check your email inbox
4. You should receive a formatted email confirming exactly which file was uploaded — the serverless pipeline is working end-to-end 🎉

---

### Quick Reference Summary
- **Lambda** = event-driven compute, no server management
- **API Gateway** = managed API layer sitting in front of backend logic
- **Step Functions** = visual orchestration for multi-step/multi-service workflows
- Build order for event-driven pipelines: **create the notification target (SNS) → create the trigger source (S3) → create the IAM role → write the Lambda function → wire up the trigger → test**
- Common Lambda event pattern: extract data from `event['Records'][0]` → process → call another AWS service via `boto3` client (e.g., `sns_client.publish(...)`)
- Always confirm SNS email subscriptions before testing, or notifications won't be delivered


















