# AWS Lambda Trigger Alert on File Upload

> Configure AWS Lambda to trigger on S3 uploads and send an email alert via SNS.

---

## 📌 Overview

- **Trigger:** File upload (PUT / ObjectCreated) to an S3 bucket
- **Action:** Lambda publishes a message to an SNS topic
- **Notification:** SNS delivers an email alert to subscribers
- **Runtime:** Python (boto3)

---

## 🧱 Architecture Flow

```text
User Upload
    │
    ▼
[ S3 Bucket ] ──(PUT event)──► [ AWS Lambda ] ──► [ SNS Topic ] ──► 📧 Email Subscriber
                                      │
                                      ▼
                               [ CloudWatch Logs ]
```

---

## 🪜 Step-by-Step Setup

### Step 1 — Create an S3 Bucket
- AWS Console → **S3 → Create Bucket**
- Enter bucket name → keep default settings → **Create bucket**

### Step 2 — Create SNS Topic for Alerts
- AWS Console → **SNS → Topics → Create Topic**
- Type: **Standard**
- Enter topic name → **Create Topic**

### Step 3 — Create Email Subscription
- Inside the SNS topic → **Create Subscription**
- Protocol: **Email**
- Enter your email → **confirm** via link in inbox

### Step 4 — Create IAM Role for Lambda
- IAM → **Roles → Create Role → Lambda**
- Attach policies:
  - `AmazonS3ReadOnlyAccess`
  - `AmazonSNSFullAccess`
- Create role

### Step 5 — Create Lambda Function
- AWS Lambda → **Create Function → Author from Scratch**
- Enter function name
- Runtime: **Python**
- Attach the IAM role from Step 4

### Step 6 — Add Lambda Code
Paste the Python code (below) to publish an SNS alert on each upload.

### Step 7 — Configure S3 Trigger
- Lambda → **Add Trigger → S3**
- Choose bucket
- Event type: **PUT / ObjectCreated**
- Enable trigger

### Step 8 — Test the Workflow
Upload any file into the S3 bucket → Lambda fires → SNS sends the alert email.

---

## 🐍 Sample Lambda Code (Python)

```python
import json
import boto3

sns = boto3.client('sns')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    file_name = event['Records'][0]['s3']['object']['key']

    message = f"New file uploaded: {file_name} in bucket {bucket}"

    sns.publish(
        TopicArn='YOUR_SNS_TOPIC_ARN',
        Message=message,
        Subject='S3 File Upload Alert'
    )

    return {
        'statusCode': 200,
        'body': json.dumps('Alert Sent Successfully')
    }
```

> **Note:** Replace `YOUR_SNS_TOPIC_ARN` with the ARN of your SNS topic.

---

## ✅ Checklist

- [ ] S3 bucket created
- [ ] SNS topic created (Standard)
- [ ] Email subscription confirmed
- [ ] IAM role with S3 read + SNS publish permissions
- [ ] Lambda function created (Python runtime)
- [ ] Lambda code added with correct SNS Topic ARN
- [ ] S3 PUT trigger wired to Lambda
- [ ] Test upload triggers email alert
- [ ] CloudWatch logs confirm successful execution

---

## 💡 Tips

- Use **event filters** (prefix/suffix) on the S3 trigger to alert only for specific folders or file types (e.g., `uploads/`, `.csv`).
- For richer messages, include file **size**, **uploader IP**, or a **presigned download URL** in the SNS message body.
- For Slack/Teams instead of email, subscribe an **HTTPS webhook** to the same SNS topic.
